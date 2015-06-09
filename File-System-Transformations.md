# Summary

Schedoscope includes a group of file system transformations. File system transformation produce view data by copying files from diverse sources into a view's `fullPath`. As such, they are particularily suitable in the stage areas of a data warehouse.

# Syntax
    case class CopyFrom(val fromPattern: String, val toView: View, val recursive: Boolean = true) 
    case class StoreFrom(val inputStream: InputStream, val toView: View)
    case class Copy(val fromPattern: String, val toPath: String, val recursive: Boolean = true)
    case class Move(val fromPattern: String, val toPath: String)
    case class IfExists(val path: String, val op: FilesystemTransformation)
    case class IfNotExists(val path: String, val op: FilesystemTransformation)

# Description

## CopyFrom

`CopyFrom` copies source files and folders matching a GLOB pattern into a view's `fullPath` (i.e., the partition folder in HDFS). This can be done recursively or non-recursively. The transformation also supports copying resources from the classpath.

Parameters:
* `fromPattern`: The source file or resource files to copy. When given a file path or file pattern, the source is interpreted as an HDFS path. If given a `file://` URL, the source is interpreted as local file system path. If given a `classpath://`, the source is interpreted as a classpath resource. The latter is useful if you want to create source-code managed configuration and lookup tables in Hive.

* `toView`: The files are copied to the `fullPath` of `toView`.

* `recursive`: Copy folders recursively or not. Defaults to `true`.

## StoreFrom

`StoreFrom` dumps an input stream as a file into a view's `fullPath` (i.e., the partition folder in HDFS). This is useful for dynamically generating lookup or configuration tables during view materialization.

Parameters:
* `imputStream`: The input stream to dump.

* `toView`: The stream is dumped to the `fullPath` of `toView`.

## Copy

`Copy` copies source files and folders matching a GLOB pattern into some target folder. This can be done recursively or non-recursively. The transformation also supports copying resources from the classpath.

* `fromPattern`: The source file or resource files to copy. When given a file path or file pattern, the source is interpreted as an HDFS path. If given a `file://` URL, the source is interpreted as local file system path. If given a `classpath://`, the source is interpreted as a classpath resource. The latter is useful if you want to create source-code managed configuration and lookup tables in Hive.

* `toPath`: The path to copy files to. When given a file path or file pattern, the source is interpreted as an HDFS path. If given a `file://` URL, the source is interpreted as local file system path.

* `recursive`: Copy folders recursively or not. Defaults to `true`.

## Move

`Move` moves source files and folders matching a GLOB pattern into some target folder. 

* `fromPattern`: The source file or resource files to move. When given a file path or file pattern, the source is interpreted as an HDFS path. If given a `file://` URL, the source is interpreted as local file system path.

* `toPath`: The path to move the specified files to. When given a file path or file pattern, the source is interpreted as an HDFS path. If given a `file://` URL, the source is interpreted as local file system path.

## IfExists

`IfExists` performs a file transformation, but only if there are files matching a path pattern. 

* `path`: The path to check. It is interpreted as an HDFS path. If given a `file://` URL, the path is interpreted as local file system path.

* `op`: The file system transformation to execute if the `path` does exist.

## IfNotExists

`IfExists` performs a file transformation, but only if there are no files matching a path pattern. This is useful to avoid repetitive copying. 

* `path`: The path to check. It is interpreted as an HDFS path. If given a `file://` URL, the path is interpreted as local file system path.

* `op`: The file system transformation to execute if the `path` does not exist.

# Helpers

None.

# Examples

# Packaging and Deployment



# Change detection