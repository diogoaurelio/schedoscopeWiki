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

## Declaring a View (Names and Modules)

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

## View Fields
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

### Organizing Fields Into Traits

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


