# Introduction

Schedoscope provides an internal Scala DSL for what is essentially programming data sets, their dependencies, and calculation logic - albeit on a very high and declarative level. 

Hence, a Schedoscope views declarations form a [Scala software development project](Setting up a Schedoscope Project), with jar files and their dependencies as resulting artifacts. Bundling and deploying Schedoscope therefore means bundling and deploying a jar files.

When using Maven as your build tool, the basic bundling options are:
- create and distribute a fat jar using the Maven assembly plugin;
- copy all jars into a separate deployment directory with the Maven dependency plugin, create a shell scripts that constructs the classpath and launches the application, and distribute that folder.

In this section, we explora the latter approach. 

# 1. Gather dependencies

Firstly, we collect all dependencies in the `dependencies` subfolder of the `target` build directory using the Maven dependency plugin:

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
            <execution>
                <id>copy-dependencies</id>
                <phase>prepare-package</phase>
                <goals>
                    <goal>copy-dependencies</goal>
                </goals>
                <configuration>
                    <includeScope>compile</includeScope>
                    <outputDirectory>${project.build.directory}/dependencies</outputDirectory>
                </configuration>
            </execution>
        </executions>
    </plugin>

# 2. 