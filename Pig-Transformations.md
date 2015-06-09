# Summary

With Pig transformations, one can compute views by means of Pig Latin. In this sense, Pig transformations are very similar to [Hive transformations]. They differ, however, by the fact that using and deploying Pig UDFs is not yet supported.

# Syntax

    case class PigTransformation(latin: String, dirsToDelete: List[String] = List())

# Description

Pig transformations support the following parameters:

* `latin`: the Pig Latin script computing the view. It supports ${parameter} style placeholders, which are replaced at query execution time by the values passed using the .configureWith() clause (see below).

* `dirsToDelete`: a list of HDFS paths that should be deleted before the Pig transformation is executed. When using `HCatStorer', this list should be empty (which is the default) since the target partition folder must exist. When using `PigStorer`, the path `fullPath` should be deleted, as the target partition folder must not exist.

# Helpers

# Examples

# Packaging and Deployment

# Change detection