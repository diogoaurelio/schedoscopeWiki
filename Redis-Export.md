# Summary

You can configure a parallel export of view data to a Redis key-value store by specifying an `exportTo()` clause with `Redis` in a view. Whenever the view performs its transformation successfully and materializes, Schedoscope triggers a mapreduce job that writes the view's data (i.e., the data of the Hive partition the view represents) to Redis using a specified field as the key. 

# Syntax
    def Redis(
        v: View,
        redisHost: String,
        key: Field[_],
        value: Field[_] = null,
        keyPrefix: String = "",
        replace: Boolean = true,
        flush: Boolean = false,
        redisPort: Int = 6379,
        redisKeySpace: Int = 0,
        numReducers: Int = Schedoscope.settings.redisExportNumReducers,
        pipeline: Boolean = Schedoscope.settings.redisExportUsesPipelineMode,
        isKerberized: Boolean = !Schedoscope.settings.kerberosPrincipal.isEmpty(),
        kerberosPrincipal: String = Schedoscope.settings.kerberosPrincipal,
        metastoreUri: String = Schedoscope.settings.metastoreUri) 

# Description

The `Redis` export supports two modes: 

1. Full export: each record of your view is assigned to the specified to key in form of a map, with the names of each field and partition parameter being the keys of that map. The values of primitive fields are stored using the corresponding Redis primitive type. The values of complex fields are serialized into a JSON structure or list and stored as a string. 

2.  Value export: Alternatively, you can specify a value field in addition to the key field. Only that field's value is then assigned to the respective key. This makes particular sense for complex fields since the export will translate Hive sets, maps, and structures to Redis sets, maps, and maps, respectively. In case of nested complex fields, these are translated to JSON strings as with the full export. In case of primitive fields, the value field's value is directly assigned to the key.

Here is a description the parameters you must or can pass to `Redis` exports:

- `v`: a reference to the view being exported, usually `this` (mandatory)
- `redisHost`: domain name or IP address of the Redis host being the target of the export (mandatory)
- `key`: the field which provides the values for the Redis key in your export, usually the ID of your view (mandatory)
- `value`: the value field to use if value export is desired. Defaults to null, which means full export
- `keyPrefix`: an optional string prefix to put in front of the key field's name to avoid clashes with other applications
- `replace`: indicates whether a key should be erased before a new value is written during the export. Defaults to true.
- `flush`: indicates whether the target keyspace should be flushed (completely erased) prior to the export. Defaults to false.
- `redisPort`: the port number the target Redis service is listening on. Defaults to 6379
- `redisKeySpace`: the key space to use for the export. Defaults to 0
- `numReducers`: the number of reducers to use during the exports, which defines the parallelism of the export. Defaults to the config setting `schedoscope.export.redis.numberOfReducers` (i.e., 10)
- `pipeline`: use pipeline connection mode for the export? Defaults to the config setting `schedoscope.export.redis.usePipelineMode` (i.e., false)
- `isKerberized`: is this a kerberized cluster environment? Defaults to true, if `schedoscope.kerberos.principal` is set in the configuration
- `kerberosPrincipal`: the Kerberos principal to use in a Kerberos cluster enviroment. Defaults to `schedoscope.kerberos.principal` as set in the configuration
- `metastoreUri`: the connection URI to use for the Hive metastore. Defaults to `schedoscope.metastore.metastoreUri` as set in the configuration

 
# Example

    case class ClickWithFullRedisExport(
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]) extends View
           with DailyParameterization {

      val id = fieldOf[String]
      val url = fieldOf[String]

      val stage= dependsOn(() => Stage(year, month, day))

      transformVia(
        () => HiveTransformation(
          insertInto(this, s"""SELECT id, url FROM ${stage().tableName} WHERE date_id='${dateId}'""")))

      // Use the field id as the Redis key, write all fields (id and url) as a map to that key.
      // The resulting Redis structure would look like this:
      // { id : { "id" : id, "url": url, "year": year, "month": month, "day": day, "date_id", date_id } }
      exportTo(() => Redis(this, "redishost", id))

    }

    case class ClickWithValueRedisExport(
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]) extends View
           with DailyParameterization {

      val id = fieldOf[String]
      val url = fieldOf[String]

      val stage= dependsOn(() => Stage(year, month, day))

      transformVia(
        () => HiveTransformation(
          insertInto(this, s"""SELECT id, url FROM ${stage().tableName} WHERE date_id='${dateId}'""")))

      // Use the field id as the Redis key, write only the value field url to that key.
      // The resulting Redis structure would look like this:
      // { id : url }
      exportTo(() => Redis(this, "redishost", id, url))

    }

