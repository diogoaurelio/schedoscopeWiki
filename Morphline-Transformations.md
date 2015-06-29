# Summary



# Syntax

    case class MorphlineTransformation(definition: String = "",
                                      imports: Seq[String] = List(),
                                      sampling: Int = 100,
                                      anonymize: Seq[Named] = List(),
                                      fields: Seq[Named] = List(),
                                      fieldMapping: Map[FieldLike[_], FieldLike[_]] = Map())

# Description

Morphline transformations support the following parameters:

* `definition`: the Kite Morphline script computing the view. 


* `dirsToDelete`: a list of HDFS paths that should be deleted before the Pig transformation is executed. When using `HCatStorer`, this list should be empty (which is the default) since the target partition folder must exist. When using `PigStorer`, the path `fullPath` should be deleted because the target partition folder must not exist.



# Examples

An example of a kite morphline script for filtering.

    transformVia(() =>
      MorphlineTransformation(
       )
     )


# Change detection

Schedoscope tries to automatically detect changes to morphlinetransformation-based views and to initiate rematerialization of views if the tranformation logic has potentially changed. This is achieved by computing a checksum on the Morphline script.
