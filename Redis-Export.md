# Summary

You can configure parallel export of view data to a Redis key-value store using the `exportTo()` clause and `Redis` within a view.

# Syntax
    def Redis(
        v: View,
        redisHost: String,
        key: Field[_],
        value: Field[_] = null,
        keyPrefix: String = "",
        replace: Boolean = false,
        flush: Boolean = false,
        redisPort: Int = 6379,
        redisKeySpace: Int = 0,
        numReducers: Int = Schedoscope.settings.redisExportNumReducers,
        pipeline: Boolean = Schedoscope.settings.redisExportUsesPipelineMode,
        isKerberized: Boolean = !Schedoscope.settings.kerberosPrincipal.isEmpty(),
        kerberosPrincipal: String = Schedoscope.settings.kerberosPrincipal,
        metastoreUri: String = Schedoscope.settings.metastoreUri) 

# Example

        case class ClickOfEC0101WithRedisExport(
          year: Parameter[String],
          month: Parameter[String],
          day: Parameter[String]) extends View
            with Id
            with DailyParameterization {

          val url = fieldOf[String]

          val click = dependsOn(() => Click(p("EC0101"), year, month, day))

          transformVia(
            () => HiveTransformation(
              insertInto(this, s"""
                    SELECT ${click().id.n}, ${click().url.n}
                    FROM ${click().tableName}
                    WHERE ${click().shopCode.n} = '${click().shopCode.v.get}'""")))

          exportTo(() => Redis(this, "localhost", id))

        }
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

