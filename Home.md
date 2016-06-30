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

Check out Metascope! It's an add-on to Schedoscope for collaborative metadata management, data discovery and exploration, and data lineage tracing:

<p align="center">
<a href="https://raw.githubusercontent.com/wiki/ottogroup/schedoscope/images/lineage.png">
<img src="https://raw.githubusercontent.com/wiki/ottogroup/schedoscope/images/lineage.png" width="100%">
</a>
</p>

- [Metascope Features and Screenshots](Metascope Primer)

## News

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

###### 05/21/2016 - Release 0.5.0
We have released Version 0.5.0 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom). 

This is a biggie. We have added Metascope to our distribution. Metascope is a collaborative metadata management, documentation, exploration, and data lineage tracing tool that exploits the integrated specification of data structure, dependencies, and computation logic in Schedoscope views. See [the tutorial](https://github.com/ottogroup/schedoscope/wiki/Open%20Street%20Map%20Tutorial) and the [Metascope primer](https://github.com/ottogroup/schedoscope/wiki/Metascope%20Primer) for more information.

###### 04/26/2016 - Release 0.4.3

We have released Version 0.4.3 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom). 

This release makes `exportTo` support the `isPrivacySensitive` clause of the View DSL. Fields and partition parameters marked with `isPrivacySensitive` are hashed during export.


###### 04/25/2016 - Release 0.4.2

We have released Version 0.4.2 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom). 

This is a bugfix release solving an issue with an overly pedantic view pattern checker in the HTTP-API sabotaging the `views` command.

###### 04/22/2016 - Release 0.4.0

We have released Version 0.4.0 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom). 

This is a big release including:

* a complete overhaul of the scheduling state machine with significant improvement of test coverage

* `exportTo` clause for simple, seamless, and parallel export of views to relational databases, Redis key-value stores, and Kafka topics (see [View DSL Primer](https://github.com/ottogroup/schedoscope/wiki/Schedoscope-View-DSL-Primer))

* new materialization modes `SET_ONLY` and `TRANSFORMATION_ONLY` for more flexible ops (see [Scheduling Command Reference](https://github.com/ottogroup/schedoscope/wiki/Scheduling-Command-Reference))
 
([more](News))
