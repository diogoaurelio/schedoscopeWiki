This section describes how to build and deploy Metascope for productional use.

When using Maven as your build tool, the basic bundling options are:
- creating and distributing a fat jar using the Maven assembly plugin;
- copying all jars into a separate deployment folder with the Maven dependency plugin, creating a shell script that constructs the classpath and launches the application, and distribute that folder.

In this section, we explore the latter approach. 

## 1. Build Metascope

Build Metascope via Maven with `mvn install -DXX:MaxPermSize=512m`. Check the target directory, which will now contain our deployment folder named metascope. 

The build results in the following directory structure:

    ${baseDir}
    |
    +-- target
        |
        +-- metascope
            |
            +-- lib
            |   |
            |   +-- dependency1.jar
            |   |
            |   +-- ...
            |
            +-- start.sh
            |
            +-- metascope.jar

The `lib` folder contains all dependencies, `metascope.jar` contains the metascope application and the `start-metascope.sh` script is used to start Metascope from commandline.