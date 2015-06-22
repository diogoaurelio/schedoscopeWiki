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
* **Default data generation** : In many cases, a test is not focussed on the
  complete set of input data, but rather on some specific columns or 
  values. For this reason, the test framework generates reasonable 
  default values for non-specified columns, which reduces the effort of 
  specifying test data to the minimal critical amount.
  
  
  
   



# Complete Example

# Test Data Defintion

# Test Running & Result Checking

# Advanced Features

## Test Resource Specification

## View Modification

## Testing against Minicluster