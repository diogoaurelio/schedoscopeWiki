# Storage paths

A Schedoscope view represents a Hive table partition, which is stored on HDFS. Views offer the following properties referring to their HDFS path:

* `fullPath`: the absolute path to the folder of the Hive table partition represented by the view.  For a view `v`, `v.fullpath`is the concatenation of `v.locationPath` and `v.partitionSpec`. For example, the `fullPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash/year=2013/month=12` in the default environment `dev`.

* `locationPath`: the absolute path to the Hive table which the partition represented by the view resides in. For a view `v`, `v.location`is the concatenation of `v.moduleLocationPath` and the view name `v.n`. For example, the `locationPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash` in the default environment `dev`.