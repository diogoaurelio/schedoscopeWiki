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

    case class TextFile(val fieldTerminator: String = null, collectionItemTerminator: String = null, mapKeyTerminator: String = null, lineTerminator: String = null, serDe: String = null, serDeProperties: Map[String, String] = null, fullRowFormatCreateTblStmt:Boolean=false) extends StorageFormat
  
The `TextFile` supports various optional parameters that can be used to override separation characters along the options offered by Hive `TEXTFILE`:
* `fieldTerminator`: character to use to separate fields;
* `collectionItemTerminator`: character to use to separate collection items in collection-typed fields;
* `mapKeyTerminator`: character to use to separate keys and values in map-type fields;
* `lineTerminator`: character to terminate the record.
* `serDe`: custom or native SerDe.
* `serDeProperties`: specify optionally SERDEPROPERTIES.
* `fullRowFormatCreateTblStmt`: for hive versions prior to 0.13, instead of using STORED AS TEXTFILE, expands CREATE TABLE with hive with: 
    STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat'
  

For example, if you define a View in the following way:

    storedAs(
        TextFile(
            fieldTerminator = """\\001""",
            collectionItemTerminator = """\002""",
            mapKeyTerminator = """\003""",
            lineTerminator = """\n"""))


.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clauses: 

    [...]
    ROW FORMAT DELIMITED
	    FIELDS TERMINATED BY '\\001'
	    LINES TERMINATED BY '\n'
	    COLLECTION ITEMS TERMINATED BY '\002'
	    MAP KEYS TERMINATED BY '\003'
	STORED AS TEXTFILE
    [...]

As one can see, all parameters are optional. One could specify simply:

    storedAs(TextFile())

.. and Schedoscope's HiveQL would simply add: 

    [...]
	STORED AS TEXTFILE
    [...]

For compatibility with Hive versions prior than 0.13, one could specify as follows:

    storedAs(TextFile(fullRowFormatCreateTblStmt = true))

.. and Schedoscope's HiveQL will change the sentence "STORED AS TEXTFILE" to: 

    [...]
	STORED AS
		INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat'
    [...]

Additionally, schedoscope provides a way to use custom Hive SerDes. One can simply specify Json storage format and the SerDe will be automatically added. 
However, let us assume that, for the sake of explanation, one would like to use a different SerDe. Here is an example of how one could do it:

    storedAs(TextFile(serDe = "com.amazon.elasticmapreduce.JsonSerde"))
    
Schedoscope's HiveQL will generate the following: 

    [...]
    ROW FORMAT SERDE 'com.amazon.elasticmapreduce.JsonSerde'
	STORED AS TEXTFILE
    [...]

Additionally, one might want to add custom SerDe properties, for example when using CSV/TSV. Here is an example of how one would do so:

    storedAs(TextFile(serDe = "org.apache.hadoop.hive.serde2.OpenCSVSerde", 
            serDeProperties = Map(
                        "separatorChar"->"""\t""",
                        "escapeChar"->"""\\"""
            ))
    )

Schedoscope's HiveQL will generate the following: 

    [...]
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
	WITH SERDEPROPERTIES (
		 'escapeChar' = '\\',
		 'separatorChar' = '\t'
	)
	STORED AS TEXTFILE
    [...]


For more information about Hive native SerDes, please consult [Hive's developer guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)


## Avro

View data can also be stored in Avro format.

    case class Avro(schemaPath: String, fullRowFormatCreateTblStmt = true)

Parameters:
* `schemaPath`: path to the Avro schema file. The path is relative to the view's `avroSchemaPathPrefix`, which defaults to `/hdp/dev/global/datadictionary/schema/avro` for the `dev` environment. This prefix can be changed by setting a different `avroSchemaPathPrefixBuilder` for a view.
* `fullRowFormatCreateTblStmt`: for hive versions prior to 0.13, instead of using STORED AS AVRO, expands CREATE TABLE with hive with: 
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe' STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'

For example, if you define a View in the following way:

    storedAs(Avro("myUniqueExamplePath"))

.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clauses: 

    [...]
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
    WITH SERDEPROPERTIES (
            'avro.schema.url' = 'hdfs:///hdp/<ENV>/global/datadictionary/schema/avro/myUniqueExamplePath'
    )
    STORED AS
    		INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
    		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
    [...]

Please note that Avro case class has an exceptional default behavior compared to other Storage Formats, as Schedoscope builds automatically avro schema path, and thus sets it as SerDe properties 'avro.schema.url'. 

For more information about Hive native SerDes, please consult [Hive's developer guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)


## Parquet

View data can also be stored using Parquet:

    case class Parquet(fullRowFormatCreateTblStmt = false)

Parameters:
* `fullRowFormatCreateTblStmt`: for hive versions prior to 0.13, instead of using STORED AS PARQUET, expands CREATE TABLE with hive with: 
    ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'


For example, if you define a View in the following way:

    storedAs(Parquet())

.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clause: 

    [...]
    STORED AS PARQUET
    [...]

For compatibility with Hive versions prior than 0.13, one could specify as follows:

    storedAs(Parquet(fullRowFormatCreateTblStmt = true))

.. and Schedoscope's HiveQL will change the sentence "STORED AS PARQUET" to: 

    [...]
	ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'
	STORED AS
		INPUTFORMAT 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat'
		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'
    [...]

For more information about Hive native SerDes, please consult [Hive's developer guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)


## Optimized Row Columnar

View data can also be stored using Optimized Row Columnar format:

    case class OptimizedRowColumnar(fullRowFormatCreateTblStmt = false)

Parameters:
* `fullRowFormatCreateTblStmt`: for hive versions prior to 0.13, instead of using STORED AS ORC, expands CREATE TABLE with hive with: 
    ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.orc.OrcSerde' STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'


For example, if you define a View in the following way:

    storedAs(OptimizedRowColumnar())

.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clause: 

    [...]
    STORED AS ORC
    [...]

For compatibility with Hive versions prior than 0.13, one could specify as follows:

    storedAs(OptimizedRowColumnar(fullRowFormatCreateTblStmt = true))

.. and Schedoscope's HiveQL will change the sentence "STORED AS ORC" to: 

    [...]
	ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.orc.OrcSerde'
	STORED AS
		INPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat'
		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
    [...]

For more information about Hive native SerDes, please consult [Hive's developer guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)


## Record Columnar File

View data can also be stored using older formats, such as Record Columnar format:

    case class RecordColumnarFile(fullRowFormatCreateTblStmt = false)

Parameters:
* `fullRowFormatCreateTblStmt`: for hive versions prior to 0.13, instead of using STORED AS RCFILE, expands CREATE TABLE with hive with: 
    STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.RCFileInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.RCFileOutputFormat'


For example, if you define a View in the following way:

    storedAs(RecordColumnarFile())

.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clause: 

    [...]
    STORED AS RCFILE
    [...]

For compatibility with Hive versions prior than 0.13, one could specify as follows:

    storedAs(RecordColumnarFile(fullRowFormatCreateTblStmt = true))

.. and Schedoscope's HiveQL will change the sentence "STORED AS ORC" to: 

    [...]
	STORED AS
		INPUTFORMAT 'org.apache.hadoop.hive.ql.io.RCFileInputFormat'
		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.RCFileOutputFormat'
    [...]

For more information about Hive native SerDes, please consult [Hive's developer guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)


## JSON 

View data can also be stored using JSON format:

    case class Json(serDe: String = "org.apache.hive.hcatalog.data.JsonSerDe", serDeProperties: Map[String, String] = null, fullRowFormatCreateTblStmt:Boolean=false)

Parameters:
* `serDe`: custom or native SerDe.
* `serDeProperties`: specify optionally SERDEPROPERTIES.
* `fullRowFormatCreateTblStmt`: for hive versions prior to 0.13, instead of using STORED AS TEXTFILE, expands CREATE TABLE with hive with: 
    STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat'
  

For example, if you define a View in the following way:

    storedAs(Json())

.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clause: 

    [...]
    ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
	STORED AS TEXTFILE
    [...]

For compatibility with Hive versions prior than 0.13, one could specify as follows:

    storedAs(Json(fullRowFormatCreateTblStmt = true))

.. and Schedoscope's HiveQL will change the sentence "STORED AS TEXTFILE" to: 

    [...]
	ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
	STORED AS
		INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat'
    [...]

For more information about Hive native SerDes, please consult [Hive's developer guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)


## CSV/TSV 

View data can also be stored using JSON format:

    case class Csv(serDe: String = "org.apache.hadoop.hive.serde2.OpenCSVSerde", serDeProperties: Map[String, String] = null, fullRowFormatCreateTblStmt:Boolean=false)

Parameters:
* `serDe`: custom or native SerDe.
* `serDeProperties`: specify optionally SERDEPROPERTIES.
* `fullRowFormatCreateTblStmt`: for hive versions prior to 0.13, instead of using STORED AS TEXTFILE, expands CREATE TABLE with hive with: 
    STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat'
  

For example, if you define a View in the following way:

    storedAs(Csv())

.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clause: 

    [...]
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
	STORED AS TEXTFILE
    [...]

In case one would like to specify SerDe properties (e.g. for TSV), one could do as follows:

    storedAs(Csv())
    serDeProperties(
        Map("separatorChar"->"""\t""",
            "escapeChar"->"""\\""",
            "quoteChar"->"'"
            )
    )

.. which would generate the following SQL:

    [...]
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
	WITH SERDEPROPERTIES (
		 'quoteChar' = ''',
		 'escapeChar' = '\\',
		 'separatorChar' = '\t'
	)
	STORED AS TEXTFILE
    [...]

For compatibility with Hive versions prior than 0.13, one could specify as follows:

    storedAs(Csv(fullRowFormatCreateTblStmt = true))

.. and Schedoscope's HiveQL will change the sentence "STORED AS TEXTFILE" to: 

    [...]
	ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
	STORED AS
		INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat'
    [...]

For more information about Hive native SerDes, please consult [Hive's developer guide](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)


## AWS S3 [Experimental]

Note: this feature is still in experimental mode. Please test it first before you roll it out to production

Besides HDFS, Schedoscope is extending support for storing files in Cloud providers, starting with AWS (S3). 

    case class S3(bucketName: String, storageFormat: StorageFormat, uriScheme: String = "s3n")

Parameters:
* `bucketName`: S3 bucket name.
* `storageFormat`: the Schedoscope case class defining the Storage format used.
* `uriScheme`: URI scheme associated with hadoop version used, as well AWS region, with default value "s3n"; for more information, please check [AWS documentation](https://wiki.apache.org/hadoop/AmazonS3) 

Note that it assumes s3n URI Scheme by default. If one would require, for example, to use AWS Frankfurt datacenter (eu-central-1), one could specify "s3a", such as the following example:

    storedAs(S3("schedoscope-bucket-test", OptimizedRowColumnar(), "s3a"))

.. Schedoscope's HiveQL will automatically add in the SQL "CREATE EXTERNAL table ..." statement the following correspondent clause: 

    [...]
	STORED AS ORC
	LOCATION 's3a://schedoscope-bucket-test/dev/test/views/<your-view-name-here>'
    [...]


## Using custom INPUT OUTPUT

If, for example, one is using older/newer Hive versions whose storage format was not contemplated in Schedoscope base case class, there is still one additional alternative, namely with InOutputFormat case class:

    case class InOutputFormat(input: String, output: String, serDe: Option[String] = None)

The following example illustrates an alternative way of specifying ORC format:

    storedAs(
        InOutputFormat(
            "org.apache.hadoop.hive.ql.io.orc.OrcInputFormat",
            "org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat",
            "org.apache.hadoop.hive.ql.io.orc.OrcSerde"
            )
    )
    
Schedoscope's HiveQL will generate the following: 

    [...]
    ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.orc.OrcSerde'
	STORED AS
		INPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat'
		OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
    [...]


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