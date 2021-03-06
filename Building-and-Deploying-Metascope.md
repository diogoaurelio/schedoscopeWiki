This section describes how to build and deploy Metascope for production use.

When using Maven as your build tool, the basic bundling options are:
- creating and distributing a fat jar using the Maven assembly plugin;
- copying all jars into a separate deployment folder with the Maven dependency plugin, creating a shell script that constructs the classpath and launches the application, and distribute that folder.

This sections describe the latter approach:

## 1. Build Metascope

Build Metascope via Maven with `mvn install -DXX:MaxPermSize=512m`. Check the target directory, which will now contain our deployment folder named `metascope-deployment`. 

The build results in the following directory structure:

    ${baseDir}
    |
    +-- metascope-deployment
        |
        +-- lib
        |   |
        |   +-- dependency1.jar
        |   |
        |   +-- ...
        |
        +-- solr
        |   |
        |   +-- solr.xml
        |   |
        |   +-- ...
        |
        +-- start.sh
        |
        +-- metascope.jar

The `lib` folder contains all dependencies, `metascope.jar` contains the metascope application, and the `start-metascope.sh` script is used to start Metascope from commandline. The `solr` folder contains the schema and config for the Solr index if the embedded mode is used. Check [Configuring Metascope](Configuring Metascope) for further information.

## 2. Create Launch Script

A launch script has to do two things: construct the classpath and launch the Metascope main program. There are [several options for launching Metascope](Starting Metascope); here we restrict ourselves to launching Metascope with the command shell open.

    #!/bin/bash
    CP=""
    for D in lib/*.jar; do CP=${CP}:${D}; done
    CP=${CP}:metascope.jar

    if [ -z "$1" ]; then
      echo "No argument supplied. Please set the path to schedoscope.conf, e.g. './start-metascope /apps/schedoscope/schedoscope.conf'"
    else
      java -Xmx1024m -XX:MaxPermSize=512M -cp ${CP} -Dconfig.file=${1} org.schedoscope.metascope.Metascope
    fi

## 3. Bundle and Deploy

As the last step, one needs to package `${baseDir}/metascope-deployment` and distribute it to the node where Metascope is supposed to run. This step depends on your environment so we will not make any suggestions on how to do this. Take into consideration that Metascope needs to be able to connect to Schedoscope, the Hive Metastore, the Hive Server, and HDFS to be fully functional.