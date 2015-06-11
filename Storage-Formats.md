# Customizing Storage paths

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