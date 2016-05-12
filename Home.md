## Introduction

Schedoscope is a scheduling framework for agile development, testing, (re)loading, and monitoring of your Hadoop data warehouse.

Schedoscope makes the headache go away you are certainly going to get when frequently having to rollout and retroactively apply changes to computation logic and data structures in your data warehouse with traditional ETL job schedulers such as Oozie.

Scheduling with Schedoscope is based on three principles:

1. _Goal orientation_: with Schedoscope, you specify the views you want and the scheduler takes care that the corresponding data are loaded.

2. _Self-sufficiency_: Schedoscope has all information about views available: structure, dependencies, transformation logic. The scheduler thus can start out from an empty metastore and create all tables and partitions as data are loaded.

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

Check out [Metascope](Metascope Primer)! It's a meta data management and data discovery tool which serves as an add-on to Schedoscope.

![Metascope](https://raw.githubusercontent.com/wiki/ottogroup/schedoscope/images/lineage.png)

## Implementing Views
- [Setting up a Schedoscope Project](Setting up a Schedoscope Project)
- [Schedoscope View DSL Primer](Schedoscope View DSL Primer)
- [Storage formats](Storage Formats)
- Transformations
 - [NoOp](NoOp Transformations)
 - [File System](File System Transformations)
 - [Hive](Hive Transformations)
 - [Pig](Pig-Transformations)
 - [MapReduce](MapReduce Transformations)
 - [Oozie](Oozie Transformations)
 - [Shell](Shell Transformations)
- Exporting to
  - [JDBC](JDBC Export)
  - [Redis](Redis Export)
  - [Kafka](Kafka Export)
- [View Traits](View Traits)
- [Test Framework](Test Framework)

## Operating Schedoscope
- [Bundling and Deploying](Bundling and Deploying)
- [Configuring](Configuring Schedoscope)
- [Starting](Starting Schedoscope)
- [Scheduling](Scheduling)
- [HTTP API](Schedoscope HTTP API)
- [Command Reference](Scheduling Command Reference)
- [View Pattern Reference](View Pattern Reference)

## Extending Schedoscope
- [Architecture](Architecture)

## News

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
