# Summary

NoOp transformations check for the existence of `_SUCCESS` flags in the `fullPath`s of the views they belong to, i.e., the views' partition folders in HDFS. Should such a flag exists, the view can enter the `materialized` state. Otherwise, it remains in `nodata` state. 

NoOp transformations are the default transformation applied if you do not specify a `transformVia()` clause.

# Syntax

    case class NoOp

# Description

NoOp transformations are useful in the staging layers of a data warehouse. External ETL jobs can import raw data into HDFS beneath the partition folders of NoOp views and then signal data availability by setting `_SUCCESS` flags. 

This implies that field and storage format declarations of NoOp views must match the raw data format, so that depending views can access the raw data via Hive and other transformations.

# Helpers

None 

# Examples

An example of a stage view expecting to get its data delivered by an external ETL process:

                case class Productfeed(
                  year: Parameter[String],
                  month: Parameter[String],
                  day: Parameter[String]) extends View
                  with DailyParameterization {

                  val productId = fieldOf[String]
                  val productName = fieldOf[String]
                  val productPrice = fieldOf[Double]
                  
                  comment("Raw product master data")
                  
                  storedAs(TextFile(fieldTerminator = "|", lineTerminator = "\\n"))
                }

Note that it defines the storage format as a `|` delimited CSV text file.

# Packaging and Deployment

NoOp transformations are self-contained, so there are no packaging and deployment aspects to consider.

# Change detection

NoOp transformations have no changeable logic. As a consequence, Schedoscope will not detect any changes to NoOp views and not automatically schedule rematerialization. If you want to rematerialize a NoOp view, e.g., because an external ETL job copied corrected raw data files after the view has already been materialized, you need to explizitly invalidate the view.