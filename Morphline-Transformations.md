# Summary

Morphline Transformations allow the embedding of morphline scripts into schedoscope. see http://kitesdk.org

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

* `imports`: Morphline-Commands to be imported (Fully qualified class names)

* `sampling`: Sampling rate for export, e.g. sampling=1 will only output 1% of the data 

* `anonymize`: Fields to be anonymized. Values in this field will be replaced by a hash-value. (Fields that are declared as PrivacySensitive will be anonymized autmatically)

* `fields`:
* 'fieldMapping':


# Examples

An example of a kite morphline script for filtering.

    transformVia(() =>
      MorphlineTransformation(
       )
     )


# Change detection

Schedoscope tries to automatically detect changes to morphlinetransformation-based views and to initiate rematerialization of views if the tranformation logic has potentially changed. This is achieved by computing a checksum on the Morphline script.
