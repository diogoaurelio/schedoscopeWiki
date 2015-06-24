Schedoscope's test framework integrates tightly with view development. Faciliating frequent test runs, it speeds up development-testing roundtrip time. 

From a conceptual point of view, the framework provides "unit testing" for Schedoscope views and transformations. The focus is neither on integration testing with a target / staging environment nor on load testing against large datasets. Rather, the framework supports correctness tests for views and transformations with simple specification of test input data, fast test execution, and painless checking of resulting view data with expected results.

The main design principles of Schedoscope's test framework are:

* _Integration with Scalatest_: Test specification and result checking is based on [Scalatest](http://scalatest.org), with its expressive library of assertions.
* _Local transformation execution_: In order to decouple testing from the availability of a Hadoop cluster environment and to achieve fast test execution times, the framework embeds various cluster components required for executing the different transformation types and configures them to run in _local_ mode.
* _Typesafe test data specification_: Because the specification of input and output data of tests "lives" within Scala, we can completely eliminate errors stemming from wrong types, column names, etc. by compile time checks. Another beneficial side-effect of this approach is that one can use auto-completion in IDEs like Eclipse while writing tests.
* _Default data generation_: The framework encourages you to write separate tests for different aspects of a view transformation by generating reasonable default values for non-specified columns of input data. It is thus possible to focus on some specific colums, reducing the effort of specifying test data to a minimal amount. This is in stark contrast to other approaches where the test data definition overhead encourages you to create one huge input data set coveriing all test cases.
  
Briefly summarized, writing a test of a given view `V` with the Schedoscope test framework consists of the following steps:

1. defining the input data for all views that `V` depends on
2. adapting the configuration of `V` to accommodate local test execution (if necessary)
3. executing the transformation of `V` (in local mode)
4. checking the outcomes against expected results
   
The following section shows a complete test example based on the [tutorial](Open Street Map Tutorial); the subsequent sections detail on the individual testing aspects.

## Complete Example

The following example is taken from the tutorial. It tests the view `Restaurants` that is populated
by a Hive transformation from a single other view `Nodes`. Comments can be found within the source code; details on specifying test data input can be found in the subsequent section _Test Data Definition_; the section _Test Running & Result Checking_ then explaines test execution and result inspection.


    case class RestaurantsTest() extends FlatSpec
      with Matchers {
      
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

### Manual Specification

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

This results in a hive table with columns 'version', 'user_id', etc.
In order to generate input rows for this view, one uses the extension
`with rows`; then, calling the `set` function adds a row, and calling
the `v` function sets a value for a specific column. By example:

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

For specifying lists and maps, the standard Scala types List and Map can be used.
In order to specify input for views which contain structures (i.e., Hive STRUCT fields),
refer to the section 'Defining structs as input'. Please note that:

* When defining data, auto-completion can be used - i.e. all fields of the 
  current view are in scope.
* When defining values, these are type-checked against the column type at 
  compile-time; hence errors can be detected early in the test creation
  process.

## Default Value generation

Often, for testing a specific aspect of a view, not all fields are relevant. For this
reason, the testing framework allows to leave out fields when defining input, and 
fills those irrelevant fields with default values. Using **ROW_ID** as a placeholder 
for the current row id (i.e. the first row has ROW_ID 1, the second one ROW_ID2, the
schema for generating default values for a field named **FIELDNAME** is (by type):

* _String_ : FIELDNAME-ROW_ID (ROW_ID is left-padded with zeroes to length 4)
* _Numeric types (Int, Double, ...)_ : ROW_ID
* _Map_ : empty Map()
* _Array_ : empty Array()
* _Struct_ : default values generation for structs is currently not supported


To stick with the "Nodes" example from above, let's say only _longitude_ and
_latitude_ would be of interest for our current focus. Then we would write:


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


## Defining structs as input

When the test input data contains structured fiels (i.e. fields of the
Hive STRUCT type), then each struct input must be defined manually. Let's say
we have a struct that looks like this:

    case class ProductInList() extends Structure {
      val productName = fieldOf[String]
      val productNumber = fieldOf[String]
      val productPosition = fieldOf[String]
    }

Then, values for it can be assigned by using the extension `with values`,
together with the `set` and `v` function known for setting values:

    val product0815 = new ProductInList with values {
      set(
        v(productName, "The Gadget"),
        v(productNumber, "0815"),
        v(productPosition, "rec-2"))
    }

# Test Running & Result Checking

The execution of tests is based on Scalatest, which allows to define
test cases and offers rich facilities to compare test results against expected
outcomes. In order to specify one or more test cases for a given view,
the recommended pattern is to write a test case class as follows:

    case class TrainstationsTest() extends FlatSpec with Matchers {

      val nodesInput = new Nodes(p("2014"), p("09")) with rows {
          // input data definition, see above
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
  input data for our view to be tested. Usually, one defines manually
  a dataset for each view dependency (using the methods described in
  the section "Test Data Input Definition" above.
* `then()` : When calling this function, the materialization process of 
  the view is triggered - in other words, the transformation which 
  populates the view is being executed in local mode. Plain local mode is
  currently available for Hive, Pig and Mapreduce transformations; 
  Oozie transformations can be tested against a local minicluster (see
  advanced section)
* `numRows()` : after the transformation has been executed, this function
  yields the number of rows in the resulting view.
* `row()` : By invoking this function, test results can be inspected
  row by row. For each call, an internal row pointer is being incremented
  (starting from the first row with the first call of this function); then,
  within the current row, each field can be individually inspected.
* `v(Field)` : As a counterpart to the `v` function used for 
  specifying input data, this function returns the value for the given field  
  in the current result row. The value is returned in a typesafe manner,
  and can be compared using the full Scalatest matcher functionality
  (e.g. `shouldBe` , `shouldHaveLength`, ...). See
  http://www.scalatest.org/user_guide/using_matchers for details. 
  
Please note that these functions represent a subset of the available
testing functionality - more advanced features (e.g. modifying view
configurations during tests) can be found in the next section. However,
the presented subset represents the core testing functionality, 
which suffices for many standard testing usecases.

# Advanced Features

Apart from the standard testing functionality, the Schedoscope testing
framework also offers more advanced features.

## View Modification

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

## Test Resource Files

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

## Testing against Minicluster

As said above, in order to speed up tests, transformations are run
completely in local mode if possible; this is currently supported
for Hive, Mapreduce and Pig Transformations. However, running e.g.
an Oozie transformation required a bit more infrastructure; for this
purpose, we are using a predefined _Hadoop minicluster_, which is
basically a small virtual Hadoop cluster running directly in the VM.
It ships with all necessary things to run Oozie transformations.

Setting up the minicluster is easy - just replace the well-known
`with test` in the test case specification by `with clustertest`.
Then, behind the scenes the minicluster will be launched
prior to test execution. Properties of the minicluster
(e.g. the namenode URI) can be accesed by the `cluster()`
method: 

    "my cool view" should s"load correctly " in {
      new CustomerCustomerNumber() with clustertest {
        basedOn(vidCustomerNr)
        withConfiguration(
          ("jobTracker" -> cluster().getJobTrackerUri),
          ("nameNode" -> cluster().getNameNodeUri),
          ...
        )
      }
    }      

A requirement for this to work is that the minicluster uses some
specific ports; these have to be defined in a file called `minicluster.properties`
on your project classpath; we recommend to place it at `src/test/resources/minicluster.properties`
with the following content:

    hadoop.namenode.port=9000
    hive.server.port=10000
    hive.metastore.port=30000
    
This configuration is picked up by the minicluster.