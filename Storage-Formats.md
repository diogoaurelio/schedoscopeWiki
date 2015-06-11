Storage formats specify the format in which view data will be stored. Schedoscope distinguishes internal and external storage formats. 

For both kinds of formats, storage is declared using the `storedAs` clause. In case no `storedAs` clause is given, the `TextFile` storage format is used.

    def storedAs(f: StorageFormat, additionalStoragePathPrefix: String, additionalStoragePathSuffix: String)

Parameters:
* `f`: the storage format to use (see below);
* `additionalStoragePathPrefix`: optional prefix to put before the view's storage name (see description of `locationPathBuilder in the Section "Customizing Storage Paths".
* `additionalStoragePathSuffix`: optional suffix to put after the view's storage name (see description of `locationPathBuilder in the Section "Customizing Storage Paths".

Examples:

A simple Parquet storage format declaration: 
    storedAs(Parquet())

A Parquet storage format declaration with an additional storage path prefix:
    storedAs(Parquet(), additionalStoragePathPrefix="tables")

# Internal Storage Formats

Internal storage formats are used for Hive table storage in HDFS. The following are supported:

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
* `schemaPath`: path to the Avro schema file.


# External Storage Formats

Schedoscope supports special storage formats used for storing view data outside of HDFS / Hive. With these formats, Schedoscope can also be used to schedule exports of views to targets like CSV files, key-value stores, or relational databases.


# Customizing Storage Paths

A Schedoscope view represents a Hive table partition stored in HDFS. Views offer the following properties with regard to their HDFS storage location:

* `fullPath`: the absolute path to the folder of the Hive table partition represented by the view.  For example, the `fullPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash/year=2013/month=12` in the default environment `dev`. For a view `v`, `v.fullpath`is the concatenation of `v.locationPath` and `v.partitionSpec`.

* `locationPath`: the absolute path to the storage location of Hive table to which the partition represented by the view belongs. For example, the `locationPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash` in the default environment `dev`.  For a view `v`, `v.location` is the concatenation of `v.moduleLocationPath` and the view's storage name `v.n`.

* `n`: a view's storage name. The name is derived from the view's class name by lowercasing all characters and adding underscores at camel-case boundaries. For instance, the storage name `n` of `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `nodes_with_geohash`.

* `moduleLocationPath`: the folder where all Hive tables belonging to a module are stored. The name is derived from a view's package name, prefixing it with `/hdp` and the Schedoscope environment. For example, the `moduleLocationPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed` in the default environment `dev`.

The reason why we have introduced the different constituents of `fullPath` above is because these particles are created by anonymous builder functions which can be overridden on a per-view basis. Thus, it is possible to implement different path naming schemes if so desired.

These builders are as follows and can be replaced by assigning custom functions to them:
* `var moduleNameBuilder: () => String`: by default returns the name of a views package.
* `var moduleLocationPathBuilder: String => String`: for a given environment, the default implementation produces the `moduleLocationPath` using `moduleNameBuilder`.
* `locationPathBuilder: String => String`: for a given environment, the default builder computes `locationPath` using `moduleLocationPathBuilder` and `n`. The latter will be surrounded by `additionalStoragePathPrefix` and `additionalStoragePathSuffix`, if set.
* `partitionPathBuilder: () => String`: this builder creates a view's `partitionSpec`. By default, this is the standard Hive `/partitionColumn=partitionValue` pattern.