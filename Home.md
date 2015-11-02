# Schedoscope

## Introduction

Schedoscope is a scheduling framework for agile development, testing, (re)loading, and monitoring of your Hadoop data warehouse.

Scheduling with Schedoscope is based on three principles:

1. _Goal orientation_: with Schedoscope, you specify the views you want and the scheduler takes care that the corresponding data are loaded.

2. _Self-sufficiency_: Schedoscope has all information about views available: structure, dependencies, transformation logic. The scheduler thus can start out from an empty metastore and create all tables and partitions as data are loaded.

3. _Reloading is loading_: Schedoscope implements measures to automatically detect changes to view structure and computation logic; as it is self-sufficient, it can then automatically recompute potentially outdated views.

## Getting Started

Get a glance of What Schedoscope does for:

- [Schedoscope at a Glance](Schedoscope at a Glance)

Follow the Open Street Map tutorial to install, compile, and run Schedoscope in a standard Hadoop distribution image:

- [Open Street Map Tutorial](Open Street Map Tutorial)

Read the View DSL Primer for more information about the capabilities of the Schedoscope DSL:

- [Schedoscope View DSL Primer](Schedoscope View DSL Primer)

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
 - [Morphline](Morphline Transformations)
 - [Shell](Shell Transformations)
- [View Traits](View Traits)
- [Test Framework](Test Framework)

## Operating Schedoscope
- [Bundling and Deploying](Bundling and Deploying)
- [Configuring](Configuring Schedoscope)
- [Starting](Starting Schedoscope)
- [Scheduling](Scheduling)
- [REST API](Schedoscope REST API)
- [Command Reference](Command Reference)
- [View Pattern Reference](View Pattern Reference)

## Extending Schedoscope
- [Architecture](Architecture)

## News

###### 9/02/2015 - Release 0.2.1

We have released Version 0.2.1 as a Maven artifact to our Bintray repository (see [Setting Up A Schedoscope Project](https://github.com/ottogroup/schedoscope/wiki/Setting-up-a-Schedoscope-Project) for an example pom).

We have added a configurable DriverRunCompletionHandler mechanism. These handlers are being called after a driver run has finished. This can be later exploited for monitoring. See [reference.conf](https://github.com/ottogroup/schedoscope/wiki/Configuring-Schedoscope) 

###### 8/20/2015 - Schedoscope @ Strata

We are happy to present Schedoscope as a system demo at Strata NYC on Wednesday, 30th September 2015.
 
([more](News))
