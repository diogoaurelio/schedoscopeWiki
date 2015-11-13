## News

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