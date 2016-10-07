Schedoscope's test framework integrates tightly with view development. Facilitating frequent test runs, speeding up development-testing roundtrip time. 

From a conceptual point of view, the framework provides "unit testing" for Schedoscope views and transformations. The focus is neither on integration testing with a target / staging environment nor on load testing against large datasets. Rather, the framework supports correctness tests for views and transformations with simple specification of test input data, fast test execution, and painless checking of resulting view data with expected results.

The main design principles of Schedoscope's test framework are:

* _Integration with Scalatest_: Test specification and result checking is based on [Scalatest](http://scalatest.org), with its expressive library of assertions.
* _Local transformation execution_: In order to decouple testing from the availability of a Hadoop cluster environment and to achieve fast test execution times, the framework embeds various cluster components required for executing the different transformation types and configures most of them to run in _local_ mode.
* _Typesafe test data specification_: Because the specification of input and output data of tests "lives" within Scala, we can completely eliminate errors stemming from wrong types, column names, etc. by compile time checks. Another beneficial side-effect of this approach is that one can use auto-completion in IDEs like Eclipse while writing tests.
* _Default data generation_: The framework encourages you to write separate tests for different aspects of a view transformation by generating reasonable default values for non-specified columns of input data. It is thus possible to focus on some specific colums, reducing the effort of specifying test data to a minimal amount. This is in stark contrast to other approaches where the test data definition overhead encourages you to create one huge input data set covering all test cases.
  
Briefly summarized, writing a test of a given view `V` with the Schedoscope test framework consists of the following steps:

1. defining the input data for all views that `V` depends on
2. adapting the configuration of `V` to accommodate local test execution (if necessary)
3. executing the transformation of `V` (in local mode)
4. checking the outcomes against expected results

To express these steps you can choose between different testing styles, depending on your concrete test case.

The following section shows complete test examples based on the [tutorial](Open Street Map Tutorial); the subsequent sections detail on the individual testing aspects.

## Complete Basic Example

The following example is taken from the tutorial. It tests the view `Restaurants` that is populated
by a Hive transformation from a single other view `Nodes`. Comments can be found within the source code; details on specifying test data input can be found in the subsequent section _Test Data Definition_; the section _Test Running & Result Checking_ then explains test execution and result inspection. Schedoscope ships with its own test spec for ScalaTest: _SchedoscopeSpec_.


    case class RestaurantsTest() extends SchedoscopeSpec {
      
      //
      // specify input data (OSM nodes). By extending the actual view by
      // "with rows", we can specify input data. Each call of "set"
      // adds a line of input data; "v" sets the value for a particular
      // column.
      //

      val nodes = new Nodes(p("2014"), p("09")) with rows {
        set(v(id, "267622930"),
          v(geohash, "t1y06x1xfcq0"),
          v(tags, Map("name" -> "Cuore Mio",
            "cuisine" -> "italian",
            "amenity" -> "restaurant")))
        set(v(id, "288858596"),
          v(geohash, "t1y1716cfcq0"),
          v(tags, Map("name" -> "Jam Jam",
            "cuisine" -> "japanese",
            "amenity" -> "restaurant")))
        set(v(id, "302281521"),
          v(geohash, "t1y17m91fcq0"),
          v(tags, Map("name" -> "Walddörfer Croque Café",
            "cuisine" -> "burger",
            "amenity" -> "restaurant")))
        set(v(id, "30228"),
          v(geohash, "t1y77d8jfcq0"),
          v(tags, Map("name" -> "Giovanni",
            "cuisine" -> "italian")))
      }
    
      //
      // run tests & check results. By extending the actual view by "with test",
      // we enable test running and result checking. With "basedOn", the input
      // views ares specified; "then" actually runs the transformation,
      // "numRows" yields the number of result rows, and "row" fetches a single
      // line of test output. Within a row, individual column values can 
      // be checked by "v", using Scalatest assertions like "shouldBe".
      // 
      // Please note that for sake of brevity, we only inspect the details
      // of the first output row.
      //

      "datahub.Restaurants" should "load correctly from processed.nodes" in {
        new Restaurants() with test {
          basedOn(nodes)
          then()
          numRows shouldBe 3
          row(v(id) shouldBe "267622930",
            v(restaurant_name) shouldBe "Cuore Mio",
            v(restaurant_type) shouldBe "italian",
            v(area) shouldBe "t1y06x1")
        }
      }
    }


## Test Input Data Definition

As stated above, testing using the Schedoscope test framework is based on small, individually defined datasets.
For this reason, the framework allows for the specification of datasets in a row-by-row manner. Because this process tends to be tedious, the framework helps users by 

1. supporting typechecking / autocompletion of input data fields test data specification and 
2. default value generation for non-specified input fields. 

#### Manual Specification

To stick with the above example, the view _Restaurants_ depends on the view _Nodes_. Its
(simplified) definition looks like

    case class Nodes(
      year: Parameter[String],
      month: Parameter[String]) extends View
      [...] {
    
      val version = fieldOf[Int]
      val user_id = fieldOf[Int]
      val longitude = fieldOf[Double]
      val latitude = fieldOf[Double]
      val geohash = fieldOf[String]
      val tags = fieldOf[Map[String, String]]
    [...]
    )

_Nodes_ maps to a Hive table with columns `version`, `user_id`, etc., and the partitions `year` and `month`. In order to generate input rows for this view, creates an anonymous subclass of _Nodes_ with the trait `rows`. Then, one can call the `set` method to add rows and `v` to set column values. 

For example:

      val nodes = new Nodes(p("2014"), p("09")) with rows {
        set(v(version, 1),
          v(user_id, 1234),
          v(longitude, 11111.11),
          v(latitude, 22222.22),
          v(geohash, "t1y1716cfcq0"),
          v(tags, Map("tag1" -> "val1",
            "tag2" -> "val2")))
        set(v(version, 2),
          v(user_id, 5678),
          v(longitude, 33333.33),
          v(latitude, 33333.33),
          v(geohash, "t1y1716cfcqx"),
          v(tags, Map("tag3" -> "val3",
            "tag3" -> "val3")))          
      }

This results in the following two rows in the input table:

| version | user_id | longitude | latitude | geohash | tags |
| ----    | ----    | ----    | ----    | ----    | ----    |
| 1       | 1234    | 11111.11 | 22222.22 | t1y1716cfcq0 | {'tag1':'val1', 'tag2':'val2'} |
| 2       | 5678    | 33333.33 | 44444.44 | t1y1716cfcqx | {'tag3':'val3', 'tag4':'val4'} |

For specifying lists and maps, the standard Scala types `List` and `Map` can be used.
In order to specify input for views which contain structures (i.e., Hive STRUCT fields),
refer to the section 'Defining Structs as Input'. Please note that:

* when defining data, auto-completion can be used - i.e. all fields of the 
  current view are in scope;
* when defining values, these are type-checked against the column type at 
  compile-time.

#### Default Value generation

Often, when testing a specific aspect of a view, not all fields are relevant. For this
reason, the testing framework allows one to skip fields when defining input. It  
fills those irrelevant fields with default values. Using `ROW_ID` as a placeholder 
for the current row number the schema for generating default values for a field named `FIELDNAME` is (by type):

* _String_: `FIELDNAME-ROW_ID` (`ROW_ID` is left-padded with zeroes to length 4)
* _Numeric types (Int, Double, ...)_: `ROW_ID`
* _Map_: empty `Map()`
* _Array_: empty `Array()`
* _Struct_: default value generation for structs is currently not supported

To stick with the `Nodes` example, let's say only _longitude_ and
_latitude_ would be of interest for our current test. Then we could write:


      val nodes = new Nodes(p("2014"), p("09")) with rows {
        set(v(longitude, 11111.11),
          v(latitude, 22222.22))
        set(v(longitude, 33333.33),
          v(latitude, 33333.33))          
      }

The resulting table looks like this:

| version | user_id | longitude | latitude | geohash | tags |
| ----    | ----    | ----    | ----    | ----    | ----    |
| 1       | 1    | 11111.11 | 22222.22 | geohash-0001 | {} |
| 2       | 2    | 33333.33 | 44444.44 | geohash-0002 | {} |


#### Defining Structs as Input

When the test input data contains structured fields (i.e. fields of the Hive STRUCT type), each struct input 
must be defined manually. Let's say we have a struct that looks like this:

    case class ProductInList() extends Structure {
      val productName = fieldOf[String]
      val productNumber = fieldOf[String]
      val productPosition = fieldOf[String]
    }

Then, values for it can be assigned by subclassing with trait `values`, together with the `set` and `v` 
functions you already know for setting values:

    val product0815 = new ProductInList with values {
      set(
        v(productName, "The Gadget"),
        v(productNumber, "0815"),
        v(productPosition, "rec-2"))
    }

## Test Running & Result Checking

The execution of tests is based on Scalatest and can look like this:

    case class RestaurantsTest() extends SchedoscopeSpec {

      val nodesInput = new Nodes(p("2014"), p("09")) with rows {
          // input data definition from above
      }
          
      // test 
      "datahub.Restaurants" should "load correctly from processed.nodes" in {
        new Restaurants() with test {
          basedOn(nodes)
          then()
          numRows() shouldBe 3
          row(v(id) shouldBe "267622930",
            v(restaurant_name) shouldBe "Cuore Mio",
            v(restaurant_type) shouldBe "italian",
            v(area) shouldBe "t1y06x1")
        }
      }                
    }

As you can see, the view which is to be tested is actually being 
instantiated by `new Restaurants()` (this is an example of a 
non-parameterized view). By extending it using `with test`, a set of 
testing functions is added to the view itself. Following the above example,
these are:

* `basedOn(View*)` : Using this function, we can define the actual
  input data for our view to be tested. Usually, one defines 
  one dataset for each view dependency.
* `then(sortedBy?, disableDependencyCheck?, disableTransformationValidation?)` : When calling this function, the materialization process of the view is triggered - in other words, the transformation which 
  populates the view is being executed in local mode. Plain local mode is
  currently available for Hive, Pig, and Mapreduce transformations; 
  Oozie transformations can be tested against a local minicluster (see
  advanced section below). 

  One can optionally pass a view field to `then` as a sorting criteria to
  make sure result rows are returned and checkable in a predictable order. 

  The test framework checks by default, whether each input view you pass to the test via `basedOn()` is among the tested view's dependencies and whether you pass at least one input view for each type of dependency as sanity checks for dependency declarations. This check can be disabled by setting `disableDependencyCheck` to `true`. 

  Finally, the test framework performs some core validations on transformations, which can be disabled by setting `disableTransformationValidation` to `true`. Currently, this only checks whether each `JOIN` clause in your Hive transformations is accompanied by an `ON` predicate.
* `numRows()` : after the transformation has been executed, this function
  yields the number of rows in the resulting view.
* `row()` : By invoking this function, test results can be inspected
  row by row. For each call, an internal row pointer is being incremented
  (starting from the first row with the first call of this function); then,
  within the current row, each field can be individually inspected.
* `v(Field)` : As a counterpart to the `v` function used for 
  specifying input data, this function returns the value for the given field  
  in the current result row. The value is returned in a typesafe manner,
  and can be compared using the full [Scalatest matcher](http://scalatest.org/user_guide/using_matchers)   
  functionality (e.g. `shouldBe` , `shouldHaveLength`, ...). 

## Alternative Testing Styles

Schedoscope offers some slight variations from the standart test style to enhance the readability and runtime for certain test scenarios.

#### Pre-transforming the View Under Test

The _SchedoscopeSpec_ trait offers the possibility to declare a view under test to the test suite. This has two advantages:

1. The view  has be only transformed once during a test.
2. You can define several independent tests on the result of one transformation.

To do this you simply have to tell the test suite which rows you want to transform before testing. This is done by passing a `View` with a `test` trait into `putViewUnderTest()` 

```java
case class RestaurantsTest() extends SchedoscopeSpec {

      // specify input data. This step is the same as before.

      val nodes = new Nodes(p("2014"), p("09")) with rows {
        set(v(id, "267622930"),
          v(geohash, "t1y06x1xfcq0"),
          v(tags, Map("name" -> "Cuore Mio",
            "cuisine" -> "italian",
            "amenity" -> "restaurant")))
        set(v(id, "288858596"),
          v(geohash, "t1y1716cfcq0"),
          v(tags, Map("name" -> "Jam Jam",
            "cuisine" -> "japanese",
            "amenity" -> "restaurant")))
      }
    
      // 
      //The view under test is registered at the test suite.
      //The suite will transform this view at the beginning of the test run
      //before any tests are evaluated. 
      //You can add multiple views under tests.
      //
      val restaurant = putViewUnderTest{
        new Restaurants() with test {
          basedOn(nodes)
          sortRowsBy(restaurant_name)
        }
      }

      //
      //Import the view under test in order to have access to its fields.
      //If you have multiple views under test you have to import the specific view 
      //you're testing into the test case.
      //
      import restaurant._

      //Now you are able to define multiple tests on the same transformed view.
      "datahub.Restaurants" should "have the right amount of rows" in {
        numRows shouldBe 2 
      }

      //A second test on the first row
      it should "have a restaurant" in {
        row(v(id) shouldBe "267622930",
          v(restaurant_name) shouldBe "Cuore Mio",
          v(restaurant_type) shouldBe "italian",
          v(area) shouldBe "t1y06x1")
      }
    
      //
      //If you are testing subsequent rows you have to forward the row index before  
      //defining assertions. This is done by calling startWithRow(index)
      //
      it should "have a second restaurant in {
        startWithRow(1)
        row(v(id) shouldBe "288858596",
          v(restaurant_name) shouldBe "Jam Jam",
          v(restaurant_type) shouldBe "japanese")
      }
    }
```

The above example shows the `then()` method is not invoked by the developer anymore, the transformation is triggered by the test suite in the background. You can still change the settings for the test transformation by using dedicated methods:
```java
val restaurant = putViewUnderTest{
  new Restaurants() with test {
    basedOn(nodes)
    sortRowsBy(restaurant_name)
    disableDependencyCheck()
    disableTransformationValidation()
  }
}
```

#### Reusing HiveSchemas Inside a Test Suite 

Both of the shown test styles rely upon definition of all dependent views and the input outside of the test cases. There are numerous reasons you want to test the same transformation with different input. In the previous examples, this would mean to define the same view schema multiple times but with different input or once with all the input. Instead, the `ReusableHiveSchema` should be used. It circumvents the previously discussed problems and provides the means for better-structured tests. The trait allows you to reuse predefined schemas for views in multiple tests. The test suite will fill these schemas during the tests. After each test case the schemas are emptied. The following example shows a refactoring of the previous full test case.

```java
case class RestaurantsTest() extends SchedoscopeSpec {

      // Specify an input schema:
      val nodes = new Nodes(p("2014"), p("09")) with InputSchema 
     
      //
      // Specify an output schema:
      // The basedOn() method defines which `InputSchema` the `OutputSchema` will 
      // use for input the tests.
      //
      val restaurant = new Restaurants() with OutputSchema {
        basedOn(nodes)
      }
      
      //
      //Import the input schema in order to have access to its fields.
      //If you have multiple output schemas to test you have to import the specific schema 
      //you're testing into the test case.
      //
      import restaurant._
      
      //
      //Each test case now follows the pattern of filling the input schema with data.
      //Triggering a transformation and verifying the results.
      //
      it should "have a restaurant" in {
        //Fill the input schema
        { 
          //define which schema you're acessing. Make sure to enclose it in it's own scope.
          import nodes._
          set(v(id, "267622930"),
            v(geohash, "t1y06x1xfcq0"),
            v(tags, Map("name" -> "Cuore Mio",
              "cuisine" -> "italian",
              "amenity" -> "restaurant")))
        }
        //trigger the test transformation
        then(restaurants)
        //validate input
        numRows() shouldBe 1
        row(v(id) shouldBe "267622930",
          v(restaurant_name) shouldBe "Cuore Mio",
          v(restaurant_type) shouldBe "italian",
          v(area) shouldBe "t1y06x1")
      }
    
      //
      //Use the schemas for another test.
      //
      it should "have a second restaurant in {
        {
          import nodes._
          set(v(id, "288858596"),
            v(geohash, "t1y1716cfcq0"),
            v(tags, Map("name" -> "Jam Jam",
              "cuisine" -> "japanese",
              "amenity" -> "restaurant")))
        }
        then(restaurants)
        numRows() shouldBe 1
        row(v(id) shouldBe "288858596",
          v(restaurant_name) shouldBe "Jam Jam",
          v(restaurant_type) shouldBe "japanese")
      }
    }
```

The `ReusableHiveSchema` trait tries to reuse some resources between test cases, so it should have some minor improvements in runtime in regards to the default test style.
  
## Advanced Features

Apart from the standard testing functionality explained above, the Schedoscope testing framework also offers more advanced features.

#### View Modification

Within Schedoscope, all transformations support a generic configuration
mechanism by using key-value pairs. Typically, the required configuration
is specified directly within the transformation definition of the view
like this (simplified Pig transformation, taken from the tutorial view
`Trainstations`):

    transformVia(() =>
      PigTransformation(
        scriptFromResource("pig_scripts/datahub/insert_trainstations.pig"))
        .configureWith(
          Map("storage_format" -> "parquet.pig.ParquetStorer()")))

As you can see, view configuration is done by the `configureWith` method.
In some cases, it can be necessary to override these specifications for testing.
In the current example, the Pig script produces Parquet output; 
however, for local testing, a plain text output (i.e. PigStorage)
is desired. This can be accomplished by the following mechanism 
in the test case:

    "datahub.Trainstations" should "load correctly from processed.nodes" in {
      new Trainstations() with test {
        basedOn(nodesInput)
        withConfiguration(
          ("storage_format" -> "PigStorage()"))
       ...    
    }

This allows to override any predefined configuration property in order
to adapt the views to the test environment.

#### Test Resource Files

Another common usecase is that transformations (e.g. Hive Queries or
Mapreduce Jobs) require resource files during their execution. The location
of these resource files is usually configured directly in the view
definition. Let's say we have a little example view called `FilteredUsers`, 
which is populated by a Hive transformation which includes a UDF called
`filter_users`, which takes a username as an argument, together with 
a path to a user whitelist file:

    case class FilteredUsers {
      [...]

      transformVia(() =>
        HiveTransformation(
          "SELECT filter_users(user_name, ${user_whitelist}) FROM users"
          withFunctions(
            this,
            Map("filter_users" -> classOf[GenericUDFFilterUsers]))
        .configureWith(
          Map("user_whitelist" -> "/hdfs/path/to/user.whitelist")))
      
      [...]    
    }

Now we have the situation that this transformation will run perfectly
fine in the target hadoop cluster, because the user whitelist file 
has been (hopefully ;) ) deployed there; however, for local testing,
this file is not available. For this purpose, the Schedoscope testing
framework allows to specify a _local_ resource for the given configuration
property as follows: 

    case class FilteredUsersTest() ... {
      
      [...]    
      
      "filtered users" should "load correctly from users" in {
        new FilteredUsers() with test {
          basedOn(...)
          withResource(("user_whitelist", "src/test/resources/user.whitelist"))
          then()
          [...]
        }
      }                
    }

Using this approach, test resource files can be managed comfortably 
within the standard Maven project location, and are made available during
the local mode testing.

#### Testing against Minicluster

In order to speed up tests, transformations are run completely in local mode if possible; this is currently supported for Hive, MapReduce and Pig Transformations. However, running e.g.
an Oozie transformation requires a more infrastructure; for this
purpose, we are using a predefined _Hadoop minicluster_, which is
basically a small virtual Hadoop cluster running directly in the JVM.
It ships with all necessary things to run Oozie transformations.

Setting up the minicluster is easy - just replace the `with test` clause in the test case specification by `with clustertest`. Then, behind the scenes the minicluster will be launched
prior to test execution. Properties of the minicluster (e.g. the namenode URI) can be accesed by the `cluster()` method: 

    "my cool view" should s"load correctly " in {
      new CoolView() with clustertest {
        basedOn(dependency1)
        withConfiguration(
          ("jobTracker" -> cluster().getJobTrackerUri),
          ("nameNode" -> cluster().getNameNodeUri),
          ...
        )
      }
    }      

A requirement for this to work is that the minicluster can use some specific ports; these have to be defined in a file called `minicluster.properties` on your project classpath; we recommend to place it at `src/test/resources/minicluster.properties` with the following content:

    hadoop.namenode.port=9000
    hive.server.port=10000
    hive.metastore.port=30000