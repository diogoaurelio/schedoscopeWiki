## News

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