# Building a data platform for data driven products

For building a data driven product or data services organizations need to integrate large amounts of
data from diverse sources. Only combining such data will lead to powerful models.

The central place for consolidating and storing data is often a Hadoop cluster. At first, most organizations
start to collect all data in raw farm, building what is sometimes called a "data lake". 

(!Insert figure Data Lake)

In most cases however, users will need to request fast access already aggregated and enriched views on the data. Therefore, data management teams will add more and more interdependent tables. If for example, raw clickstream data is ingested, a derived table will contain sessions and visit. Those visits can be joined with product data to create the base table for recommendations and so on. Such a platform is sometimes referred to as "data hub" with different layers for raw, preprocessed and aggregated data.

(!insert figure datahub)

This way large organizations will soon have hundreds of interdependent tables in their Hadoop data warehouse.

Managing such a platform can be difficult, because of its complexity.  Let us outline the main obstacles we faced with maintaining a large Hadoop DWH with traditional means such as hive scripts executed by Oozie.

## Agility

If data in the base table changes, all tables depending in them need to be recalculated. 

## Complicated and error-prone configuration

Using Hive with Oozie requires you to create and maintain a myriad of script and configuration files for each job:
	1.	Schema definition in Hive DDL
	2.	The actual Hive or Pig script
	3.	Oozie workflow.xml defintition
	4.	Extend Oozie datasets.xml definition. This file must be duplicated for each workflow.
	5.	Write a coordinator.xml. 
	6.	Update bundle.xml

## Lack of dependency management and packaging

Most workflows need additional resources such as jars for user-defined functions (UDFs), configuration files or static data such as user-agent strings. You need to version, manage these dependencies and make sure that they get deployed together with your workflow.

The next paragraphs reflect our gripes with Oozie in particular. It might well be, that other workflow schedulers won’t suffer from the same problems.
## Oozie does not support partial reload of data. 

In practical environments you will have to partially reload tables. Reasons might be:
	1.	One of your datasources delivered incomplete or defect data
	2.	You had  a bug in one of your workflows
	3.	Your analysts need a new metric which needs to be extracted from unstructured data

Oozie requires you to manage the state of your data partitions manually. For this purpose we first implemented a tool that will create remove _SUCCESS flags for a given time-range and tables. 

## Oozie is inefficient

## Oozie’s logging and error messages are cryptic and meaningless


# How Schedoscope tackles most of these issues



