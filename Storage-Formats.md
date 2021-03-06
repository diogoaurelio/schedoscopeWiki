Storage formats specify the format in which view data ist to be stored. Storage is declared using the `storedAs` clause. In case no `storedAs` clause is given, the `TextFile` storage format is used as a default.

    def storedAs(f: StorageFormat, additionalStoragePathPrefix: String, additionalStoragePathSuffix: String)

Parameters:
* `f`: the storage format to use (see below);
* `additionalStoragePathPrefix`: optional prefix to put before the view's storage name (see description of `locationPathBuilder in the Section "Customizing Storage Paths").
* `additionalStoragePathSuffix`: optional suffix to put after the view's storage name (see description of `locationPathBuilder in the Section "Customizing Storage Paths").

Examples:

A simple Parquet storage format declaration: 
    
    storedAs(Parquet())

A Parquet storage format declaration with an additional storage path prefix:
    
    storedAs(Parquet(), additionalStoragePathPrefix="tables")

Here is a description of currently supported storage formats:

## TextFile

With this storage format, view data is stored in the Hive `TEXTFILE` format.

    case class TextFile(val fieldTerminator: String, collectionItemTerminator: String, mapKeyTerminator: String, lineTerminator: String) extends StorageFormat
  
The `TextFile` supports various optional parameters that can be used to override separation characters along the options offered by Hive `TEXTFILE`:
* `fieldTerminator`: character to use to separate fields;
* `collectionItemTerminator`: character to use to separate collection items in collection-typed fields;
* `mapKeyTerminator`: character to use to separate keys and values in map-type fields;
* `lineTerminator`: character to terminate the record.

## Avro

View data can also be stored in Avro format.

    case class Avro(schemaPath: String)

Parameters:
* `schemaPath`: path to the Avro schema file. The path is relative to the view's `avroSchemaPathPrefix`, which defaults to `/hdp/dev/global/datadictionary/schema/avro` for the `dev` environment. This prefix can be changed by setting a different `avroSchemaPathPrefixBuilder` for a view.

## Parquet

View data can also be stored using Parquet:

    case class Parquet()

# Storage Paths

A Schedoscope view represents a Hive table partition stored in HDFS. Views offer the following properties with regard to their HDFS storage location:

* `fullPath`: the absolute path to the folder of the Hive table partition represented by the view.  For example, the `fullPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash/year=2013/month=12` in the default environment `dev`. For a view `v`, `v.fullpath`is the concatenation of `v.tablePath` and `v.partitionPath`.

* `tablePath`: the absolute path to the storage location of Hive table to which the partition represented by the view belongs. For example, the `tablePath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash` in the default environment `dev`.  For a view `v`, `v.tablePath` is the concatenation of `v.dbPath` and the view's storage name `v.n`.

* `n`: a view's storage name. The name is derived from the view's class name by lowercasing all characters and adding underscores at camel-case boundaries. For instance, the storage name `n` of `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `nodes_with_geohash`.

* `dbPath`: the folder where all Hive tables belonging to a module are stored. The name is derived from a view's package name, prefixing it with `/hdp` and the Schedoscope environment. For example, the `dbPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed` in the default environment `dev`.

The reason why we have introduced the different constituents of `fullPath` above is because these particles are created by anonymous builder functions which can be overridden on a per-view basis. Thus, it is possible to implement different path naming schemes if so desired.

These builders are as follows and can be replaced by assigning custom functions to them:
* `var moduleNameBuilder: () => String`: by default returns the name of a views package, in lower-case-underscore format.
* `var dbPathBuilder: String => String`: for a given environment, the default implementation produces the `dbPath` using `moduleNameBuilder`. It does this by building a path from the lower-case-underscore format of `moduleNameBuilder`, replacing `_` with `/` and prepending `hdp/dev/` for the default dev environment.
* `tablePathBuilder: String => String`: for a given environment, the default builder computes `tablePath` using `dbPathBuilder` and `n`. The latter will be surrounded by `additionalStoragePathPrefix` and `additionalStoragePathSuffix`, if set.
* `partitionPathBuilder: () => String`: this builder creates a view's `partitionPath`. By default, this is the standard Hive `/partitionColumn=partitionValue` pattern.
* `fullPathBuilder: String => String`: for a given environment, the default builder produces `fullPath` by concatenating `tablePathBuilder` and `partitionPathBuilder`. 

Moreover, the schema path prefix for the Avro storage format can be changed by assigning a custom function to
* `avroSchemaPathPrefixBuilder: String => String`: for a given environment `env`, the default implementation produces the path `/hdp/${env}/global/datadictionary/schema/avro`.