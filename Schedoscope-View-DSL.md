# Introduction

The core of Schedoscope consists of an internal Scala DSL for modeling data sets, their structure, dependencies, and transformation rules as so-called _views_.

A view comprises
- a name and module;
- data fields and their types;
- partitioning parameters;
- comments;
- its dependencies, i.e. the data sets a view is based on;
- the transformation rule by which it is computed from its dependencies;
- additional storage hints about how to store a view's data when materializing it.

In a sense, a view can be thought of a partitioned database table. In fact, Schedoscope maps views directly to Hive table partitions. 

In the following, we present the main building blocks of the view DSL.

# Declaring a View (Names and Modules)

A view represents a data set. At a minimum, it has a name and resides in a module. 

Technically, a view is created by defining a Scala case class, subclassing the base class `View`. The name of this case class forms the name of the view; the package of the view defines the module of the view. 
    
Example: 

    package test.module
    import org.schedoscope.dsl.View
 
    case class Brand extends View


The package and view name are used to create the Hive table name `tableName` by concatenating the database name `dbName` with the storage name `n`. The latter are constructed as follows:

* `dbName`: prepend the configured `schedoscope.app.environment` to the package, lowercase all characters, and replace `.` by `_`. In the example, the `dbName` in the default environment `dev` is `dev_test_module`. This rule can be changed by replacing the builder `var dbNameBuilder: String => String` with an anonymous function receiving the environment and returning the desired database name.

* `n`: lowercase all characters, adding `_` at camel-case boundaries. In the example, that would be `brand`.

As a consequence, the default full table name for the example view in the default environment `dev` is `dev_test_module.brand`. 

# View Fields
Data sets represented by views are structured. They have fields to capture the different attributes of a data set. Each field has a type. 

The following basic types are supported at the moment:
* String
* Byte, Int, Long, Float, Double
* Boolean
* Date

Fields can be declared using `fieldOf`:

    package test.module
    import org.schedoscope.dsl.View
     
    case class Brand extends View {
      val id = fieldOf[String]
      val name = fieldOf[String]
      val createdAt = fieldOf[Date]
      val createdBy = fieldOf[String]
    }
     
    case class Product extends View {
      val id = fieldOf[String]
      val name = fieldOf[String]
      val price = fieldOf[Double]
      val brandName = fieldOf[String]
      val createdAt = fieldOf[Date]
      val createdBy = fieldOf[String]
    }

## Organizing Fields Into Traits

As a view is just a Scala class, common fields and field naming conventions can be factored into reusable traits, for example:

    package test.module
    import org.schedoscope.dsl.View
    import org.schedoscope.dsl.views.Id
    import org.schedoscope.dsl.views.JobMetadata
     
    case class Brand extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]
    }
     
    case class Product extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]
      val price = fieldOf[Double]
      val brandName = fieldOf[String]
    }

The `Id` and `JobMetadata` traits implement the `ViewDsl` interface and look like this:

    trait Id extends ViewDsl {
      val id = fieldOf[String](Int.MaxValue)
    }
     
    trait JobMetadata extends ViewDsl {
      val createdAt = fieldOf[Date](1)
      val createdBy = fieldOf[String](0)
    }

In order to control field ordering across traits, one can pass field weights as additional parameters to field declarations. In the example above, the maximum possible weight is attached to the id field. As a consequence, id will always be considered the first field of a view using the `Id` trait. `createdAt` and `createdBy` are assigned very low weights. They will usually be the last fields of views using the `JobMetadata` trait.

Schedoscope comes with a set of useful [traits](View Traits): `Id`, `JobMetadata`, `DailyParameterization`, `MonthlyParameterization`, `IntervalOccurrence`, and `PointOccurrence`. However, one is free to organize view fields into traits as desired for an application. Schedoscope does not impose any scheduling semantics over those prepackaged traits.

## Complex Fields

Fields can be of complex type: they can be structures, lists, maps, and arbitrary nestings of those. 

Similar to views, structures are declared by subclassing the class `Structure`; lists and maps are formed using the standard Scala types `List` and `Map`.

For example:

    case class NestedStructure extends Structure {
      val aField = fieldOf[Boolean]
    }

    case class ComplexStructure extends Structure {
      val aField = fieldOf[Int]
      val aComplexField = fieldOf[List[NestedStructure]]
    }

    case class ComplexView extends View {
      val aMap = fieldOf[Map[String, Int]]
      val anArray = fieldOf[List[Int]]
      val aComplicatedMap = fieldOf[Map[List[String], List[Map[String, Int]]]]
      val aStructure = fieldOf[ComplexStructure]
    }

# View Parameters

Data sets represented by views rarely come into existence as monolithic blocks that never change. Usually, data sets are created at specific levels of granularity, for example, in daily or hourly intervals. Schedoscope allows for such data production granularities to be expressed in form of view parameters. Parameters are declared by passing Parameter objects to the constructor of the view. As a further consequence, view parameters set up the Hive partitioning scheme.

Coming back to our example views, brand and product data could be produced and partitioned on a daily basis for different shops:

    case class Brand(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]
    }
     
    case class Product(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]
      val price = fieldOf[Double]
      val brandName = fieldOf[String]
    }

Even though it is not possible to simply move case class constructor parameter declarations into traits, traits can still be used to make sure that common data production patterns are obeyed by requiring that parameters for a view do exist. In the following example, the Scala compiler will complain if a view does not define parameters required by a trait.

    trait DailyParameterization {
      val year: Parameter[String]
      val month: Parameter[String]
      val day: Parameter[String]
    }
     
    trait ShopParameterization {
      val shopCode: Parameter[String]
    }
     
    trait DailyShopParameterization extends ShopParameterization with DailyParameterization
     
    case class Brand(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with DailyShopParameterization
      with Id
      with JobMetadata {
      val name = fieldOf[String]
    }
     
    case class Product(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with DailyShopParameterization
      with Id
      with JobMetadata {
      val name = fieldOf[String]
      val price = fieldOf[Double]
      val brandName = fieldOf[String]
    }

Please note that for the common cases of daily and monthly partitioning and data production schemes, Schedoscope comes with [basic implementations of traits `DailyParameterization` and `MonthlyParameterization`](View Traits). 

## Instantiation

As views are Scala case classes, they can be straightforwardly instantiated as objects. This is required to build up view dependencies as described further below.

When instantiating a parameterized view, one needs to pass it parameter values. In order to build parameter values from standard Scala values, the method `Parameter.p` should be used. `Parameter.p` should also be used to wrap parameters handed over from one view to another. This ensures consistent ordering of partitioning columns.

    import import com.ottogroup.bi.soda.dsl.Parameter._
 
    val brandsTodayForShop101 = Brand(p("101"), p("2014"), p("12"), p("04"))
    val brandsForDifferentShopAtDifferentDate = Brand(p("0601"), p("2014"), p("10"), p("24"))
    
# Storage Formats

The storage format Hive is supposed to use to materialize views can be specified using storedAs() and passing it a storage format.

The following storage formats are currently supported:

* `TextFile()`: This represents the TEXTFILE format. fieldTerminators etc. can be adjusted from the defaults if need be;
* `Parquet()`: The parquet file format.
* `Avro():` Avro. Requires an HDFS path to where the schema file is located. Note that the schema file is not parsed by Schedoscope. All Avro fields have to be specified in Schedoscope again.

Note that most storage formats offer configuration options and that storage directory location and naming can be adapted on a per-view basis. For more information see [Storage Formats](Storage Formats).

The following defines Parquet as the storage format for our views:

    case class Brand(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]
     
      storedAs(Parquet())
    }
     
    case class Product(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]
      val price = fieldOf[Double]
      val brandName = fieldOf[String]
     
      storedAs(Parquet())
    }

# Comments

A view can be given a comment.

    case class Brand(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]

      comment("Each shop delivers a list of all active brands on a daily basis") 
      storedAs(Parquet())
    }
     
    case class Product(
      shopCode: Parameter[String],
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]
    ) extends View 
      with Id
      with JobMetadata {
      val name = fieldOf[String]
      val price = fieldOf[Double]
      val brandName = fieldOf[String]

      comment("Each shop delivers a list of all active products on a daily basis")  
      storedAs(Parquet())
    }

# Dependencies

Schedoscope views may be computed from other views. This has implications on scheduling as a view can only be computed if all prerequisite views have been computed already.

Dependencies are declared via the method `dependsOn()`. `dependsOn` lazily builds up a dependency graph between view instances. There are two variants of the method:

1. The first variant of `dependsOn()` expects a function that returns a single view instance. `dependsOn` returns this function as its result so it can be assigned to a property for later traversal to the dependency.

1. The other variant of `dependsOn()` expects a function that returns a sequence of view instances. This variant returns `Unit`

Generally, `dependsOn()` can be called multiple times in any variant to incrementally declare dependencies. 

Examples: