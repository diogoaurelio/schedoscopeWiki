# Storage paths

A Schedoscope view represents a Hive table partition, which is stored on HDFS. Views offer the following properties referring to their HDFS path:

* `fullPath`: the absolute path to the folder of the Hive table partition represented by the view.  For a view `v`, `v.fullpath`is the concatenation of `v.locationPath` and `v.partitionSpec`. For example, the `fullPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash/year=2013/month=12` in the default environment `dev`.

* `locationPath`: the absolute path to the storage location of Hive table to which the partition represented by the view belongs. For a view `v`, `v.location` is the concatenation of `v.moduleLocationPath` and the view's storage name `v.n`. For example, the `locationPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed/nodes_with_geohash` in the default environment `dev`.

* `n`: a view's storage name. The name is derived from the view's class name by lowercasing all characters and adding underscores at camel-case boundaries. For instance, the storage name `n` of `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `nodes_with_geohash`.

* `moduleLocationPath`: the folder where all Hive tables belonging to a module are stored. The name is derived from a view's package name, prefixing it with `/hdp` and the Schedoscope environment. For example, the `moduleLocationPath` of the view `schedoscope.example.osm.processed.NodesWithGeohash(p("2013"), p("12"))` would be `/hdp/dev/schedoscope/example/osm/processed` in the default environment `dev`.

The reason why we have introduced the different constituents of `fullPath` above is because these particles are created by anonymous builder functions which can be overridden on a per-view basis. Thus, it is possible to configure different path naming schemes if so desired.

The builders are as follows and can be replaced by assigning custom functions to them:

