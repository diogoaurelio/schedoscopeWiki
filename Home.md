## Introduction

Schedoscope is a scheduling framework for agile development, testing, (re)loading, and monitoring of your Hadoop data warehouse.

Schedoscope makes the headache go away you are certainly going to get when frequently having to rollout and retroactively apply changes to computation logic and data structures in your data warehouse with traditional ETL job schedulers such as Oozie.

Scheduling with Schedoscope is based on three principles:

1. _Goal orientation_: with Schedoscope, you specify the views you want and the scheduler takes care that the corresponding data are loaded.

2. _Self-sufficiency_: Schedoscope has all information about views available: structure, dependencies, transformation logic. The scheduler thus can start out from an empty metastore and create all tables and partitions as data are loaded. Also, metadata management and lineage tracing is trivially as data structure and dependencies are explicitly specified.

3. _Reloading is loading_: Schedoscope implements measures to automatically detect changes to view structure and computation logic; as it is self-sufficient, it can then automatically recompute potentially outdated views.

## Getting Started

Get a glance of what Schedoscope does for you:

- [Schedoscope at a Glance](Schedoscope at a Glance)

Build it:

     [~]$ git clone https://github.com/ottogroup/schedoscope.git
     [~]$ cd schedoscope
     [~/schedoscope]$  MAVEN_OPTS='-XX:MaxPermSize=512m' mvn clean install

Follow the Open Street Map tutorial to install and run Schedoscope in a standard Hadoop distribution image:

- [Open Street Map Tutorial](Open Street Map Tutorial)

Read the View DSL Primer for more information about the capabilities of the Schedoscope DSL:

- [Schedoscope View DSL Primer](Schedoscope View DSL Primer)

Read more about how Schedoscope actually performs its scheduling work:

- [Schedoscope Scheduling](Scheduling)

Check out Metascope! It's an add-on to Schedoscope for collaborative metadata management, data discovery and exploration, and data lineage tracing:

<p align="center">
<a href="https://raw.githubusercontent.com/wiki/ottogroup/schedoscope/images/lineage.png">
<img src="https://raw.githubusercontent.com/wiki/ottogroup/schedoscope/images/lineage.png" width="100%">
</a>
</p>

- [Metascope Features and Screenshots](Metascope Primer)

## News

###### 11/30/2016 - Release 0.7.1

We have released Version 0.7.1 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

This release includes a fix removing bad default values for the driver setting `location` for some transformation types. Moreover, it now includes the config setting `schedoscope.hadoop.viewDataHdfsRoot` which allows one to set a root folder different from `/hdp` for view table data without having to register a new `dbPathBuilder` builder function for each view.

###### 11/01/2016 - Release 0.7.0

[Spark transformations, finally](https://github.com/ottogroup/schedoscope/wiki/Spark-Transformations)! Build views based on Scala and Python Spark 1.6.0 jobs or run your Hive transformations on Spark. Test them using the Schedoscope test framework like any other transformation type. `HiveContext` is supported.

We have also upgraded Schedoscope's dependencies to CDH-5.8.3. There is catch, though: we had to backport Schedoscope 0.7.0 to Scala 2.10 for compatibility with Cloudera's Spark 1.6.0 dependencies.

We have released Version 0.7.0 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

###### 10/20/2016 - Release 0.6.7

Minor improvements to test framework.

###### 10/07/2016 - Release 0.6.6
We have released Version 0.6.6 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

The test framework has received some love. There are [two new testing styles](https://github.com/ottogroup/schedoscope/wiki/Test%20Framework#alternative-testing-styles) that can make your tests look prettier and run faster:
* compute a view once and execute multiple tests on its data;
* create the Hive structures for input views and views under test once and load these with different data within each test case saving Hive environment setup overhead and keeping input data and assertions next to each other within each test.

###### 08/19/2016 - Release 0.6.5
We have released Version 0.6.5 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

We have factored out Oozie, Pig, and shell transformations and their drivers into separate modules and removed knowledge about which transformation types exist from `schedoscope-core`. Thus, one can now extend Schedoscope with new tranformation types without touching the core.

We have fixed a bug in the test framework where sorting results with null values yielded a null pointer exception.

###### 08/12/2016 - Release 0.6.4
We have released Version 0.6.4 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

We have added: 
* simple parallel [(S)FTP exporting of views](https://github.com/ottogroup/schedoscope/wiki/(S)FTP%20Export)
* the ability to manually assign versions to transformations with `defineVersion` in order to avoid unnecessary recomputations in complex cases where the automatic transformation logic change detection generates too many false positives.

###### 07/01/2016 - Release 0.6.3
We have released Version 0.6.3 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

We have fixed a security issue with Metascope that allowed non-admin users to edit taxonomies.

###### 06/30/2016 - Release 0.6.2
We have released Version 0.6.2 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

Hadoop dependencies have been updated to CDH-5.7.1. A critical bug that could result in no more views transforming while depending views still waiting has been fixed. Reliability of Metascope has been improved.

###### 06/23/2016 - Release 0.6.1
We have released Version 0.6.1 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom). 

Hive transformations are no longer submitted via Hive Server 2 to the cluster but directly via the `hive-exec` library. The reason for this change are stability and resource leakage issues commonly encountered when operating Hive Server 2. Please note that Hive transformations are now issued with `hive.auto.convert.join` set to false by default to limit heap consumption in Schedoscope due to involuntary local map join operations. Refer to [Hive Transformation](https://github.com/ottogroup/schedoscope/wiki/Hive%20Transformations) for more information on how to reenable map joins for queries that need them.

Also: quite a few bug fixes, better error messages when using the CLI client, improved parallelization of JDBC exports.  

###### 05/27/2016 - Release 0.6.0
We have released Version 0.6.0 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom). 

We have updated the checksumming algorithm for Hive transformations such that changes to comments, settings, and formatting no longer affect the checksum. This should significantly reduce operations worries. However, the checksums of all your Hive queries compared to Release 0.5.0 will change. **Take care that you issue a materialization request with [mode `RESET_TRANSFORMATION_CHECKSUMS`](Scheduling Command Reference) when switching to this version to avoid unwanted view recomputations!** Hence the switch of the minor release number.

The test framework now automatically checks whether there is an `ON` condition for each `JOIN` clause in your Hive queries. Also, it checks whether each input view you provide in `basedOn` is also declared as a dependency.
 
([more](News))
