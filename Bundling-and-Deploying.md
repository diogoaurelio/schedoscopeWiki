# Introduction

Schedoscope provides an internal Scala DSL for what is essentially programming data sets, their dependencies, and calculation logic - albeit on a very high and declarative level. 

Hence, a Schedoscope views declarations form a [Scala software development project](Setting up a Schedoscope Project), with jar files and their dependencies as resulting artifacts. Bundling and deploying Schedoscope therefore means bundling and deploying jar files as for any other JVM project.

When using Maven as your build tool, the basic bundling options are:
- create and distribute a fat jar using the Maven assembly plugin;
- copy all jars into a separate deployment directory with the Maven dependency plugin, create a shell scripts that constructs the classpath and launches the application, and distribute that folder.

In this section, we explore the latter approach. 

## 1. Gather dependencies

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

## 2. Copy to Deployment Folder

Next, we copy the project's jar files and the dependency files to using the Maven assembly plugin

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>2.4</version>
        <executions>
            <execution>
                <id>bin</id>
                <phase>package</phase>
                <goals>
                    <goal>single</goal>
                </goals>
                <configuration>
                    <outputDirectory>deployment</outputDirectory>
                    <finalName>deployment</finalName>
                    <descriptor>src/main/assemble/deployment-package.xml</descriptor>
                </configuration>
            </execution>
        </executions>
    </plugin>

with the assembly descriptor `deployment-package.xml`

    <assembly
        xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
        <id>package</id>
        <includeBaseDirectory>false</includeBaseDirectory>
        <formats>
            <format>dir</format>
        </formats>
        <files>
            <file>
                <source>${basedir}/src/main/resources/eci-logback.xml</source>
                <outputDirectory>/</outputDirectory>
            </file>
            <file>
                <source>${basedir}/src/main/resources/schedoscope.conf</source>
                <outputDirectory>/</outputDirectory>
            </file>
            <file>
                <source>${basedir}/src/main/resources/start.sh</source>
                <outputDirectory>/</outputDirectory>
            </file>
        </files>
        <fileSets>
            <fileSet>
                <directory>${basedir}/target/</directory>
                <outputDirectory>/</outputDirectory>
                <includes>
                    <include>*.jar</include>
                </includes>
            </fileSet>
            <fileSet>
                <directory>${basedir}/target/dependencies/</directory>
                <outputDirectory>/lib/</outputDirectory>
                <includes>
                    <include>*.jar</include>
                </includes>
            </fileSet>
        </fileSets>
    </assembly>

This results in the following directory structure:

    ${baseDir}
    |
    +-- deployment
        |
        +-- deployment-package
            |
            +-- lib
            |   |
            |   +-- dependency.jar
            |   |
            |   +-- ...
            |
            +-- schedoscope.conf
            |
            +-- eci-logback.xml
            |
            +-- start.sh
            |
            +-- project.jar
            |
            +-- project-hive.jar
            |
            +-- ...

Note that the project's build artificats end up in the folder `${baseDir}/deployment/deployment-package`. The dependencies end up in the folder `${baseDir}/deployment/deployment-package/lib`.

Additionally, a Schedoscope configuration file and logback configuration file have been copied from the `resources` to the `deployment-package` folder. Please refer to the [Schedoscope Configuration](Schedoscope Configuration) section for more information on those.

Moreover, a launch script `start.sh` has been copied to `${baseDir}/deployment/deployment-package` which we cover in the next section.

## 3. Create Launch Script

