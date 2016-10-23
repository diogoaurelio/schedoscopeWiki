# Summary

Views can be computed using Spark transformations. A Spark transformation simply calls a Spark job, which may be written in Scala or Python and use SQL and Hive contexts. The Spark job resides in jar files or Python script files. The Spark job is assumed to read the view's dependencies and write its results in the view's `fullPath` (the HDFS location of the Hive partition the view represents).

# Syntax

    case class SparkTransformation(
        applicationName: String, mainJarOrPy: String, mainClass: String = null,
        applicationArgs: List[String] = List(),
        master: String = "yarn-cluster", deployMode: String = "cluster",
        additionalJars: List[String] = List(),
        additionalPys: List[String] = List(),
        additionalFiles: List[String] = List(),
        propertiesFile: String = null
      ) extends Transformation

# Description

Spark transformations have the following parameters:
* `applicationName`: The logical name of the Spark job used for creating a logging ID among other things. Required.
* `mainJarOrPy`: The absolute path to the jar or Python file the Spark job resides in. Required.
* `mainClass`: In case the Spark job is implemented in Scala, the fully qualified class name with the job's main method.
* `applicationArgs`: The list of command line arguments to pass to the spark job.
* `master`: The master supposed to execute the job. This defaults to `yarn-cluster`. Note that local masters are not available for normal use as Schedoscope assumes that such Jobs are to be executed within the context of the test framework, which you do not want in production.
* `deployMode`: The deployment mode to use. Should be `cluster`, usually.
* `additionalJars`: An optional list of additional jar files to submit with the job.
* `additionalPys`: An optional list of additional Python files to submit with the job.
* `additionalFiles`: An optional list of additional other files to submit with the job.
* `propertiesFile`: The absolute path to a properties file with Spark configuration properties to submit with the job.

Environment variables, Spark arguments, and Spark configuration values can be passed to the job using the tranformation's configuration. The configuration entries are interpreted as follows:

* if the configuration key starts with `--` the key / value pair is considered a Spark argument;
* if the configuration key starts with `spark.` the key / value pair is considered a Spark configuration;
* otherwise the key / value pair is considered an environment variable.

In order for Hive contexts to work, the appropriate `hive-site.xml` file needs to be on Spark's extra classpath and reside in the given location on all nodes of your cluster. By default, Schedoscope adds the configuration value of schedoscope.tranformation.spark.libDirectory to the Spark extra classpath, assuming that `hive-site.xml` is available on all nodes of the cluster in that location. The default value for this configuration property is `/etc/hive/conf`. 

# Helpers

The `SparkTransformation` companion object provides the following helper methods:

### jarOf

`jarOf(classImplementingSparkJob: Object)` returns the absolute path of the jar file where the passed class / object implementing a Spark job is located. This is useful, if the Spark job resides on Schedoscope's classpath.

### classNameOf

`classNameOf(classImplementingSparkJob: Object)` returns the fully qualified class name of the passed class / object implementing a Spark job. This is useful, if the Spark job resides on Schedoscope's classpath.

### runOnSpark

`runOnSpark(hiveTransformation: HiveTransformation)` creates a Spark transformation from a given Hive transformation in order to execute it with SparkSQL. Note that there may be syntactical, semantical, and configuration differences between HiveQL and SparkSQL.

`runOnSpark()` can also accept the following optional parameters of `SparkTransformation` for further configuration:
* `master`
* `deployMode`
* `additionalJars`
* `additionalPys`
* `additionalFiles`
* `propertiesFile`

# Examples

Here is a basic Spark transformation launching a Spark job somewhere on the filesystem on YARN and passing it some command line parameters:

    transformVia(() =>
        SparkTransformation(
             "aJob", "/usr/local/spark/some-spark-job.jar", "org.example.ASparkJob",
             List("argument1", "argument2")
        )
    )

If the Spark job resides on the classpath of Schedoscope, this can be simplified to:

    transformVia(() =>
        SparkTransformation(
             classNameOf(ASparkJob), jarOf(ASparkJob), classNameOf(ASparkJob),
             List("argument1", "argument2")
        )
    )
    
A Python job can be launched like this:

    transformVia(() =>
        SparkTransformation(
             "aJob", "/usr/local/spark/some-spark-job.py",
             applicationArgs = List("argument1", "argument2")
        )
    )

    
If you need to pass Spark configurations and arguments, you can do this with `.configureWith`:

    transformVia(() =>
        SparkTransformation(
             classNameOf(ASparkJob), jarOf(ASparkJob), classNameOf(ASparkJob),
             List("argument1", "argument2")
        ).configureWith(
            Map(
                "--executor-memory" -> "2G",
                "spark.driver.memory" -> "1G",
                "someEnvironmentVariableForASparkJob" -> "value"
            )
        )
    )
    
Finally, you can launch SparkSQL jobs using Hive transformations, as shown by the following example from the tutorial:

    transformVia { () =>
        runOnSpark(
          HiveTransformation(
            insertInto(
              this,
              queryFromResource("hiveql/datahub/insert_restaurants.sql")))
            .configureWith(defaultHiveQlParameters(this))
        )
    }

    
# Packaging and Deployment

Spark jobs must be properly packaged into jar files such that Schedoscope can deploy and start the job on the cluster. 

Generally, there are two ways of deploying Spark job jar files:

## In-Project

The classes of the Spark job can be part of the same codebase as the Schedoscope views. In this case, these classes should be bundled into a separate jar file. The name of this jar should end with `-spark.jar` and be on the classpath when launching Schedoscope. 

With Maven, a `-spark.jar` jar can be packaged using the Proguard plugin, for example:

    <plugin>
        <groupId>com.github.wvengen</groupId>
        <artifactId>proguard-maven-plugin</artifactId>
        <version>2.0.8</version>
        <executions>
            <execution>
                <id>package-spark-resources</id>
                <phase>package</phase>
                <goals>
                    <goal>proguard</goal>
                </goals>
                <configuration>
                    <obfuscate>false</obfuscate>
                    <injar>classes</injar>
                    <libs>
                        <lib>${java.home}/lib/rt.jar</lib>
                    </libs>
                    <options>
                        <option>-keep public class example.some.package.with.spark.jobs.**.** { *; }</option>
                        <option>-dontnote **</option>
                        <option>-dontwarn</option>
                        <option>-dontshrink</option>
                        <option>-dontoptimize</option>
                        <option>-dontskipnonpubliclibraryclassmembers</option>
                    </options>
                    <outjar>${project.build.finalName}-spark.jar</outjar>
                    <inFilter>**.class</inFilter>
                    <assembly>
                        <inclusions>
                            <inclusion>
                                <groupId>some.additional.dependencies</groupId>
                                <artifactId>the-spark-jobs-depend-on</artifactId>
                            </inclusion>
                        </inclusions>
                    </assembly>
                </configuration>
            </execution>
        </executions>
    </plugin>

Here, all classes in the package `example.some.package.with.spark.jobs` are bundled, along with used classes from the `the-spark-jobs-depend-on` library. The resulting jar ends up in the Maven target directory.

## Ex-Project

Should a Spark job reside in an external jar file, it just needs to be referenced as a Maven dependency and put on the classpath. 

# Change detection

Schedoscope tries to automatically detect changes to Spark transformation-based views and to initiate rematerialization of views if the tranformation logic has potentially changed. For Spark transformations, this checksum is based on the jar file name the MapReduce job classes reside in as well as the name of the class implementing the job. As a consequence, if you want to trigger automatic rematerialization of Spark-based views, you need to change the jar filename of the job, e.g., by incrementing a version number.