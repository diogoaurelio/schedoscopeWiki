# Storage paths

A Schedoscope view represents a Hive table partition, which is stored on HDFS. Views offer the following properties referring to their HDFS path:

* `fullPath`: the absolute path to the folder of the Hive table partition represented by the view.  For a view `v`, `v.fullpath`is the concatenation of `v.locationPath` and `v.partitionSpec`. For example, in the default environment `dev`, the `fullPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash/year=2013/month=12`.

* `locationPath`: