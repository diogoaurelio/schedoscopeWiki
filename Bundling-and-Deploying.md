# Introduction

Schedoscope provides an internal Scala DSL for what is essentially programming data sets, their dependencies, and calculation logic - albeit on a very high and declarative level. Hence, a Schedoscope views declarations form a [Scala software development project](Setting up a Schedoscope Project), with jar files and their dependencies as resulting artifacts. Bundling and deploying Schedoscope therefore means bundling and deploying a jar files.

Your basic options in this regard are:
- create and distribute a fat jar using the Maven assembly plugin;
- copy all jars into a separate deployment directory with the Maven dependency plugin, create a shell scripts that constructs the classpath and launches the application, and distribute that folder.

