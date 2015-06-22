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
by a single other view called "Nodes". Comments on each phase can 
be found within the source code; details on specifying test data input
can be found in the subsequent section _Test Data Definition_; the section
_Test Runing & Result Checking_ then details on test execution and
result inspection.


    case class RestaurantsTest() extends FlatSpec
      with Matchers {
      
      #
      # specify input data (OSM nodes). By extending the acutal view
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
      # views ares specified; "then" acutually runs the transformation,
      # "numRows" yields the number of result rows, and "row" fetches a single
      # line of test output. With in a row, individual column values can 
      # be checked by "v", together with the Scalatest comparision operators
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


# Test Data Defintion

# Test Running & Result Checking

# Advanced Features

## Test Resource Specification

## View Modification

## Testing against Minicluster