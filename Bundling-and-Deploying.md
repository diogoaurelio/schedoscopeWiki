# Introduction

Schedoscope is an internal Scala DSL for specifying views (Hive table partitions), their structure and dependencies, as well as the transformation logic required compute views from other views. 

Hence, 

Your basic options in this regard are:
- create and distribute a fat jar using the Maven assembly plugin;
- copy all jars into a separate deployment directory with the Maven dependency plugin, create a shell scripts that constructs the classpath and launches the application, and distribute that folder.