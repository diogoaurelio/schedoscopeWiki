# Summary

With Oozie transformations, one can compute views with Oozie workflows. 

# Syntax

    case class OozieTransformation(bundle: String, workflow: String, workflowAppPath: String)

# Description

Oozie transformations take the following parameters:

* `bundle`: logical name for the bundle the workflow belongs to
* `workflow`: logical name of the workflow
* `workflowAppPath`: the HDFS folder where the workflow is deployed (i.e., `OozieClient.APP_PATH`)

# Helpers

## oozieWFAppPath

Schedoscope supports an autodeployment mechanism for Oozie bundles. If that mechanism is used, `workflowAppPath` should point to the correct path used for autodeployment (see below). `oozieWFPath` constructs this path from the `bundle` and `workflow` names of the transformation.

    def oozieWFPath(bundle: String, workflow: String): String

# Examples


# Packaging and Deployment


# Change detection
