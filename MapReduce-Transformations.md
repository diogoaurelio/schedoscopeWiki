# Summary

Views can be computed using MapReduce transformations. This provides the lowest but also the most complete level of implementation for transformation logic. 

# Syntax

    case class MapreduceTransformation(
         v: View, createJob: (Map[String, Any]) => Job,     
         cleanupAfterJob: (Job, MapreduceDriver, DriverRunState[MapreduceTransformation]) => DriverRunState[MapreduceTransformation] = (_, __, completionRunState) => completionRunState,
         dirsToDelete: List[String] = List())

# Description

MapReduce transformations have the following parameters:
* `v`: the view being computed by the MapReduce transformation.
* `createJob`: a function receiving a Map with config values passed by `.configureWith()` and returning a properly configured Hadoop `org.apache.hadoop.mapreduce.Job` object.
* `cleanupAfterJob`: a function being called after the job has completed, receiving the Hadoop `org.apache.hadoop.mapreduce.Job` object, a reference to the `MapreduceDriver` that executed the job, and the `DriverRunState` of the job. The function is expected to perform any relevant cleanup and to return a final run state.  The default implementation does nothing and simply passes on the run state.
* `dirsToDelete`: HDFS paths to be deleted prior to transformation execution. The `fullPath` of the view is always deleted prior to execution, because MapReduce jobs expect empty target directories.

# Helpers

None.

# Examples

The following MapReduce transformation is taken from the tutorial. It instantiates a Hadoop job object for calculating geo hashes.

    transformVia(() =>
        MapreduceTransformation(
          this,
          conf: Map[String, Any]) => {
            val job = Job.getInstance
            LazyOutputFormat.setOutputFormatClass(job, classOf[TextOutputFormat[Text, NullWritable]]);
            job.setJobName(this.urlPath)
            job.setJarByClass(classOf[GeohashMapper])
            job.setMapperClass(classOf[GeohashMapper])
            job.setNumReduceTasks(0)
            FileInputFormat.setInputPaths(job, conf.get("input_path").get.toString)
            FileOutputFormat.setOutputPath(job, new Path(conf.get("output_path").get.toString))
            val cfg = job.getConfiguration()
            if (System.getenv("HADOOP_TOKEN_FILE_LOCATION") != null) {
              cfg.set("mapreduce.job.credentials.binary",
                System.getenv("HADOOP_TOKEN_FILE_LOCATION"))
            }
            job
          }).configureWith(
            Map(
              "input_path" -> stageNodes().fullPath,
              "output_path" -> fullPath)))

# Packaging and Deployment

MapReduce jobs must be properly packaged into jar files such that Schedoscope can automatically deploy and start the MapReduce job on the cluster. By using `job.setJarByClass()` in the `createJob` function, the jar file will be uploaded to the cluster by the MapReduce framework.

Generally, there are two ways of deploying MapReduce job jar files:

## In-Project

The mapper and reducer classes can be part of the same codebase as the Schedoscope views. In this case, these classes should be bundled into a separate jar file. The name of this jar should end with `-mapreduce.jar` and be on the classpath when launching Schedoscope. 

With Maven, a `-mapreduce.jar` jar can be packaged using the Proguard plugin, for example:

    <plugin>
        <groupId>com.github.wvengen</groupId>
        <artifactId>proguard-maven-plugin</artifactId>
        <version>2.0.8</version>
        <executions>
            <execution>
                <id>package-mapreduce-resources</id>
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
                        <option>-keep public class example.osm.mapreduce.**.** { *; }</option>
                        <option>-dontnote **</option>
                        <option>-dontwarn</option>
                        <option>-dontshrink</option>
                        <option>-dontoptimize</option>
                        <option>-dontskipnonpubliclibraryclassmembers</option>
                    </options>
                    <outjar>${project.build.finalName}-mapreduce.jar</outjar>
                    <inFilter>**.class</inFilter>
                    <assembly>
                        <inclusions>
                            <inclusion>
                                <groupId>ch.hsr</groupId>
                                <artifactId>geohash</artifactId>
                            </inclusion>
                            <inclusion>
                                <groupId>org.apache.commons</groupId>
                                <artifactId>commons-lang3</artifactId>
                            </inclusion>
                        </inclusions>
                    </assembly>
                </configuration>
            </execution>
        </executions>
    </plugin>

Here, all classes in the package `example.functions` are bundled, along with the used classes from the geohash and Apache commons library. The resulting jar ends up in the Maven target directory.

## Ex-Project

Should a MapReduce job reside in an external jar file, it just needs to be referenced as a Maven dependency and put on the classpath. 

# Change detection

Schedoscope tries to automatically detect changes to MapReduce transformation-based views and to initiate rematerialization of views if the tranformation logic has potentially changed. For MapReduce transformations, this checksum is based on the jar file name the MapReduce job classes reside in. As a consequence, if you want to trigger automatic rematerialization of MapReduce-based views, you need to change the jar filename of the job, i.e., by incrementing a version number.