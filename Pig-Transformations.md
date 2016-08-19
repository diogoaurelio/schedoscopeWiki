# Summary

With Pig transformations, one can compute views by means of Pig Latin. In this sense, Pig transformations are very similar to [Hive transformations](Hive Transformations). They differ, however, by the fact that using and deploying Pig UDFs is not yet supported.

If you want to use Pig transformations, you need to [add the artifact `schedoscope-transformation-pig` to your `pom.xml`](Setting up a Schedoscope Project).


# Syntax

    case class PigTransformation(latin: String, dirsToDelete: List[String] = List())

# Description

Pig transformations support the following parameters:

* `latin`: the Pig Latin script computing the view. It supports ${parameter} style placeholders, which are replaced at query execution time by the values passed using the `.configureWith()` clause (see below).

* `dirsToDelete`: a list of HDFS paths that should be deleted before the Pig transformation is executed. When using `HCatStorer`, this list should be empty (which is the default) since the target partition folder must exist. When using `PigStorer`, the path `fullPath` should be deleted because the target partition folder must not exist.

# Helpers

The following functions help with the creation of Hive transformations:

## scriptFrom

`scriptFrom()` reads a Pig Latin script from a given file path.

    def scriptFrom(filePath: String): String

Parameters:
* `filePath`: the local file path from which to read the script.

## scriptFromResource

`scriptFromResource()` reads a Pig Latin script from a resource on the classpath.

    def scriptFromResource(resourcePath: String): String

Parameters:

* `resourcePath`: the resource path from which to read the script.

# Examples

An example of a Pig Latin script compressing a raw source text file.

    transformVia(() =>
      PigTransformation("""
        in = LOAD '${INPUT}' using PigStorage();
        out = STORE in INTO '${OUTPUT}' using PigStorage();""",
        List(this.fullPath)
     ).configureWith(
        Map("INPUT" -> rawLog().fullPath,
            "OUTPUT" -> this.fullPath,
            "output.compression.enabled" -> "true",
            "output.compression.codec" -> "org.apache.hadoop.io.compress.GzipCodec")             
        )
     )

Since `PigStorer` is used, the Pig transformation specifies that the `fullPath`of the given view is to be deleted prior to execution. 

The script can be factored out to a resource file using the `scriptFromResource()` helper:

    transformVia(() =>
      PigTransformation(scriptFromResource("/pig/stage/compress_raw.pig"),
        List(this.fullPath)
     ).configureWith(
        Map("INPUT" -> rawLog().fullPath,
            "OUTPUT" -> this.fullPath,
            "output.compression.enabled" -> "true",
            "output.compression.codec" -> "org.apache.hadoop.io.compress.GzipCodec")             
        )
     )


# Packaging and Deployment

Pig UDFs are not yet supported, so no application-specific code needs to be packaged and deployed. However, Schedoscope automatically bundles and deploys `HCatStorage`.

# Change detection

Schedoscope tries to automatically detect changes to Pigtransformation-based views and to initiate rematerialization of views if the tranformation logic has potentially changed. This is achieved by computing a checksum on the Pig Latin script.

