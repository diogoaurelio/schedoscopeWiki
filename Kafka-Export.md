# Summary

You can configure a parallel export of view data to a Kafka topic by specifying an `exportTo()` clause with `Kafka` in a view. Whenever the view performs its transformation successfully and materializes, Schedoscope triggers a mapreduce job that writes the view's data (i.e., the data of the Hive partition the view represents) to Kafka.

# Syntax

    def Kafka(
        v: View,
        key: Field[_],
        kafkaHosts: String,
        zookeeperHosts: String,
        replicationFactor: Int = 1,
        numPartitons: Int = 3,
        exportSalt: String = Schedoscope.settings.exportSalt,
        producerType: ProducerType = ProducerType.sync,
        cleanupPolicy: CleanupPolicy = CleanupPolicy.delete,
        compressionCodec: CompressionCodec = CompressionCodec.gzip,
        encoding: OutputEncoding = OutputEncoding.string,
        numReducers: Int = Schedoscope.settings.kafkaExportNumReducers,
        isKerberized: Boolean = !Schedoscope.settings.kerberosPrincipal.isEmpty(),
        kerberosPrincipal: String = Schedoscope.settings.kerberosPrincipal,
        metastoreUri: String = Schedoscope.settings.metastoreUri)

# Description

The `Kafka` export can transport a view's data in one of two ways: it can export each record as a one-line JSON string or as Avro with the Avro schema tailored to the view's structure. 

Note that in both cases the data export is lossless, i.e., even complex or nested field types are recursively and adequately translated to JSON or Avro. View fields and parameters marked with `isPrivacySensitive` are hashed with MD5 during export.

The name of the topic the data is exported to is the name of the database and table name associated with the view concatenated by underscore. The topic is created if it does not exist already.

Here is a description the parameters you must or can pass to `Kafka` exports:

- `v`: a reference to the view being exported, usually `this` (mandatory)
- `key`: the field which provides the values for the key of the topic you are exporting to, usually the ID of your view (mandatory)
- `kafkaHosts`: domain names or IP addresses of the hosts in the Kafka cluster being the target of the export (mandatory)
- `zookeeperHosts`: domain names or IP addresses of the hosts in the Zookeeper cluster which Kafka operates on (mandatory)
- `replicationFactor`: the replication factor to use on the target topic. Defaults to 1
- `numPartitions`: the number of partitions to divide the target topic into. Defaults to 3
- `exportSalt: the salt to use for MD5-hashing of view fields and parameters marked with `isPrivacySensitive`. Defaults to the config setting `schedoscope.export.salt`.
- `producerType`: indicates whether to use a synchronous (`sync`) or asynchronous producer (`async`) to write to the target topic. Defaults to synchronous
- `cleanupPolicy`: indicates whether log compaction is enabled for the topic (`compact`). Defaults to a `delete` policy.
- `compressionCodec`: indicicates whether compression is to be applied during export and which codec to use. Can be `none`, `snappy`, or `gzip`(default)
- `encoding`: Specifies how the view's records are to be encoded for the export: as one-line JSON strings (`string`) or Avro records (`avro`). The default is JSON.
- `numReducers`: the number of reducers to use during the exports, which defines the parallelism of the export. Defaults to the config setting `schedoscope.export.redis.numberOfReducers` (i.e., 10)
- `isKerberized`: is this a kerberized cluster environment? Defaults to true, if `schedoscope.kerberos.principal` is set in the configuration
- `kerberosPrincipal`: the Kerberos principal to use in a Kerberos cluster enviroment. Defaults to `schedoscope.kerberos.principal` as set in the configuration
- `metastoreUri`: the connection URI to use for the Hive metastore. Defaults to `schedoscope.metastore.metastoreUri` as set in the configuration

 
# Example

    package test.export

    case class ClickWithKafkaExport(
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

      // Export the view's data to the Kafka topic dev_test_export_click_with_kafka_export in JSON format.
      // Each record would be translated to a JSON line of the form of:
      // { id: "id", url: "url", year: "year", month: "month", day: "day", date_id: "date_id" }
      // The value of id is specified to be the record's key in the topic.
      exportTo(() => Kafka(this, id, "kafka-01:9092", "zookeeper-01:2182"))

    }
