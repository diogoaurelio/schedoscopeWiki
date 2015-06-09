# Summary

Views can be computed using MapReduce transformations. This provides the lowest but also the most complete level of implementation for transformation logic. 

# Syntax

    case class MapreduceTransformation(view: View, createJob: (Map[String, Any]) => Job, dirsToDelete: List[String] = List())

# Description

MapReduce transformations have the following parameters:
* `view`: the view being computed by the MapReduce transformation.
* `createJob`: a function receiving a Map with config values passed by `.configureWith()` and returning a properly configured Hadoop `org.apache.hadoop.mapreduce.Job` object.
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

# Change detection

Schedoscope tries to automatically detect changes to MapReduce transformation-based views and to initiate rematerialization of views if the tranformation logic has potentially changed. For MapReduce transformations, this checksum is based on the jar file name the MapReduce job classes reside in. As a consequence, if you want to trigger automatic rematerialization of MapReduce-based views, you need to change the jar filename of the job, i.e., by incrementing a version number.