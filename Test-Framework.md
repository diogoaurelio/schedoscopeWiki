# Introduction

The core idea of Schedoscope's test framework is to integrate tightly 
with the development of views and transformations, and to speed up the
testing roundtrip time in order to facilitate frequent test runs. 
From a conceptual point of view, the framework can be seen as an 
implementation of "unit testing" on the level of Schedoscope views
and transformations. The focus hereby is neither integration testing with
a target / staging environment, nor load testing against large datasets - but
rather checking the correctness of views and transformations by manual
specification of test data input, and comparing the outcomes with expected
results.

The main design principles to alleviate the testing process are:

* **Integration with Scalatest** : Running Tests and especially checking 
  results is based upon the well-established mechanisms of Scalatest, which
  offers rich and powerful means for comparing against expected outcomes.
* **Local Transformation Running** : In order to decouple the testing
  process from the availability of (potentially) remote clusters or 
  environments, the framework ships with a _local_ implementation for
  all transformation types. This enables a comfortable local working 
  mode, together with a much faster test execution time. 
* **Typesafe Test Data Management** : Because the specification of input
  and output data of tests "lives" within Scala, we can completely eliminate
  errors stemming from wrong types, column, names... by compile time checks.
  Another beneficial side-effect of this approach is the possibility to use
  auto-completion in IDEs like Eclipse for writing tests.
* **Default data generation** : In many cases, a test is not focused on the
  complete set of input data, but rather on some specific columns or 
  values. For this reason, the test framework generates reasonable 
  default values for non-specified columns, which reduces the effort of 
  specifying test data to the minimal critical amount.
  
  
Briefly summarized, a test of a given view V with the Schedoscope test 
framework consists of the following steps:

1. Definition of input data for all views that V depends on
2. Adaption of the configuration of V (if necessary)
3. Run the transformations of V (in local mode)
4. Check the outcomes against expected results
   
The following section shows a complete test example based on the tutorial;
the subsequent sections detail on the individual testing aspects.


# Complete Example

The following example is taken from the tutorial. It tests a view called
"Restaurants", which holds information about restaurants and is populated
by a Hive transformation from a single other view called "Nodes". 
Comments on each phase can be found within the source code; details on 
specifying test data input can be found in the subsequent section 
_Test Data Definition_; the section _Test Running & Result Checking_ 
then details on test execution and result inspection.


    case class RestaurantsTest() extends FlatSpec
      with Matchers {
      
      #
      # specify input data (OSM nodes). By extending the actual view
      # by "with rows", we can specify input data; each call of "set"
      # adds a line of input data, and "v" sets the value for a particular
      # column.
      #
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
    
      #
      # run tests & check results. By extending the actual view by "with test",
      # we enable test running and result checking. With "basedOn", the input
      # views ares specified; "then" actually runs the transformation,
      # "numRows" yields the number of result rows, and "row" fetches a single
      # line of test output. With in a row, individual column values can 
      # be checked by "v", together with the Scalatest comparison operators
      # like "shouldBe"
      #
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


# Test Input Data Definition

As stated above, testing within the Schedoscope test framework is
based on small, manually defined datasets. For this reason, the
framework offers the possibility to precisely specify a dataset
in a row-by-row manner. Because this process can be of course somehow
tedious, the framework supports the user by (i) typechecking / 
autocompletion of input data fields during specification and (ii)
default value generation for non-specified input fields. 
In addition, existing test input resources (e.g. sample data files)
can be loaded from classpath.

## Manual Specification

To stick with the above example, let's say we want to test a view
called _Restaurants_ (taken from the tutorial), which holds 
information about restaurants in Hamburg extracted from Open Street
Map nodes. This view nees another view called _Nodes_ as input; its
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
**with rows**; then, calling the **set* function adds a row, and calling
the **v** function sets a value for a specific column. By example:

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

For specifying lists and maps, the standard Scala types List, Map can be used.
In order to specify input for views which contain structures (i.e., Hive STRUCT fields),
refer to the section 'Defining structs as input'. Please note that:

* When defining data, auto-completion can be used - i.e. all fields of the 
  current view are in scope.
* When defining values, these are type-checked against the column type at compile-time.

## Default Value generation

Often, for testing a specific aspect of a view, not all fields are relevant. For this
reason, the testing framework allows to leave out fields when defining input, and 
fills those irrelevant fields with default values. Using **ROW_ID** as a placeholder 
for the current row id (i.e. the first row has ROW_ID 1, the second one ROW_ID2, the
schema for generating default values for a field named **FIELDNAME** is (by type):

* **String** : FIELDNAME-ROW_ID (ROW_ID is left-padded with zeroes to length 4)
* **Numeric types (Int, Double, ...)** : ROW_ID
* **Map** : empty Map()
* **Array** : empty Array()
* **Struct** : default values generation for structs is currently not supported


To stick with the "Nodes" example from above, let's say only __longitude__ and
__latitude__ would be of interest for our current focus. Then we would write:


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

Then, values for it can be assigned by using the extension **with values**,
together with the **set** and **v** function known for setting values:

    val product0815 = new ProductInList with values {
      set(
        v(productName, "The Gadget"),
        v(productNumber, "0815"),
        v(productPosition, "rec-2"))
    }

# Test Running & Result Checking

# Advanced Features

## Test Resource Files

## View Modification

## Testing against Minicluster