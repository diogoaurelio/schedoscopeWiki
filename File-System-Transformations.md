# Summary

Schedoscope includes a group of file system transformations. File system transformation produce view data by copying files from diverse sources into a view's `fullPath`. Hence, they are particularily suitable in the stage areas of a data warehouse.

# Syntax
    case class CopyFrom(val fromPattern: String, val toView: View, val recursive: Boolean = true) 
    case class StoreFrom(val inputStream: InputStream, val toView: View)
    case class Copy(val fromPattern: String, val toPath: String, val recursive: Boolean = true)
    case class Move(val fromPattern: String, val toPath: String)
    case class IfExists(val path: String, val op: FilesystemTransformation)
    case class IfNotExists(val path: String, val op: FilesystemTransformation)

# Description


# Helpers

# Examples

# Packaging and Deployment

# Change detection