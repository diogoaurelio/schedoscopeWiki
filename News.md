## News

###### 08/19/2016 - Release 0.6.5
We have released Version 0.6.5 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

We have factored out Oozie, Pig, and shell transformations and their drivers into separate modules and removed knowledge about which transformation types exist from `schedoscope-core`. Thus, one can now extend Schedoscope with new tranformation types without touching the core.

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

###### 01/22/2016 - Release 0.3.5

We have released Version 0.3.5 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

This release migrates Schedoscope's Hadoop dependencies to CDH-5.5.1. Furthermore, the test framework has been ported to Hive 1.1.0. Finally, Schedoscope's resilience against Metastore failures has been improved. It is able to reconnect and resume work when the Metastore has become unavailable in more error cases.

###### 11/21/2015 - Release 0.3.4

We have released Version 0.3.4 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

This release fixes a bug in Schedoscope which led to not correctly instantiating ViewActors for newly appearing dependencies such as date changes. Moreover, checksum versioning code has been cleaned up. Note that checksumming is not backwards compatible; you might want to execute your next materializations with the -m RESET_TRANSFORMATION_CHECKSUMS option.


###### 11/13/2015 - Release 0.3.3

We have released Version 0.3.3 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

This release gets some order into the logging framework mess inherited from the various libraries used. It does so by routing Java util logging and Apache commons logging through SLF4J and SLF4J to logback. By muting log4j and setting an appropriate logback-test.xml test outputs are now a lot less chatty.

###### 11/10/2015 - Release 0.3.2

We have released Version 0.3.2 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

This fixes a nasty resource leak in the Touch FileSystemTransformation

###### 11/09/2015 - Release 0.3.1

We have released Version 0.3.1 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

Fields can now be given comments as well: `val id = fieldOf[String]("An ID.")` 

###### 11/06/2015 - Release 0.3.0

We have released Version 0.3.0 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

This is a _big_ release, with the following major changes:

* Migration to Scala 2.11 and Akka 2.3.14
* Support of Hive 1.1.0 in test framework
* Significant code cleanup 
* Significant round of Scaladoc documentation
* Significant performance improvements when dealing with many views / partitions

Please note that the cleanup incurred some breaking of the API. In particular, the storage format classes have been moved to a separate package `org.schedoscope.dsl.storageformats`. Moreover, the various path builders for views have been renamed in a more systematic way. See [Storage Paths](https://github.com/ottogroup/schedoscope/wiki/Storage-Formats#storage-paths).


###### 10/08/2015 - Release 0.2.2

We have released Version 0.2.2 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

Notable changes:

* `materializeOnce` clause for views (see [Schedoscope View DSL Primer / Materialize Once](https://github.com/ottogroup/schedoscope/wiki/Schedoscope-View-DSL-Primer))
* `RESET_TRANSFORMATION_CHECKSUMS_AND_TIMESTAMPS` materialization mode (see [Command Reference / Materialize](https://github.com/ottogroup/schedoscope/wiki/Command-Reference))

###### 9/02/2015 - Release 0.2.1

We have released Version 0.2.1 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

We have added a configurable DriverRunCompletionHandler mechanism. These handlers are being called after a driver run has finished. This can be later exploited for monitoring. See [reference.conf](https://github.com/ottogroup/schedoscope/wiki/Configuring-Schedoscope) 

###### 8/27/2015 - Release 0.2.0

We have released Version 0.2.0 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

The two major changes involve:

* producing no-data state for views with filesystem transformations that actually do not result in any data instead of materialized state;

* the `MonthlyParameterization` trait now includes an additional automatically generated partition value `month_id` of the form `YYYYMM` to offer additional partition selection options in queries. This is similar to `date_id` of `DailyParameterization`. 

Note that the change to `MonthlyParameterization` results in recomputation of views that implement it. 


###### 8/20/2015 - Schedoscope @ Strata

We are happy to present Schedoscope as a system demo at Strata NYC on Wednesday, 30th September 2015.

###### 8/7/2015 - Release 0.1.1

We have released Version 0.1.1 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom). 

This is a minor release that comprises some code cleanup and performance optimizations with regard to view initialization and Morphline transformations. We also ímplemented a [shell transformation](https://github.com/ottogroup/schedoscope/wiki/Shell-Transformations) (more documentation to come). 