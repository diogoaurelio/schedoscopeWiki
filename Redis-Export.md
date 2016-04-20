# Summary

You can configure parallel export of view data to a Redis key-value store by specifying an `exportTo()` clause with `Redis` in a view. Whenever the view performs its transformation successfully and materializes, the view's data (i.e., the data of the Hive partition the view represents) is written to Redis using a specified field as the key. 

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

# Description

The `Redis` export supports two modes: 

1. Full export: each record of your view is written as a map to the specified to key, with each field and partition parameter being a key of that map. Complex fields are serialized into a JSON structure or list and stored as a string within that map while fields of primitive types are assigned a corresponding Redis primitive type.

2.  Value export: Alternatively, you can select an additional value field and only that field's value is written to each key. This makes particular sense for complex fields, as the export will translate Hive sets, maps, and structures to Redis sets, map, and maps respectively. In case of nested complex fields, these are translated to JSON strings as with the full export.

Here is a description the parameters you must or can pass to `Redis` exports:

- `v`: a reference to the view being exported, usually `this` (mandatory)
- `redisHost`: domain name or IP address of the Redis host being the target of the export (mandatory)
- `key`: the field which provides the values for the Redis key in your export, usually the ID of your view (mandatory).

# Example

    case class ClickWithRedisExport(
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
      // id : { "id" : id, "url": url, "year": year, "month": month, "day": day, "date_id", date_id }
      exportTo(() => Redis(this, "redishost", id))

    }
