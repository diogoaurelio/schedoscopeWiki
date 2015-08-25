# Building a data platform for data driven products

For building a data driven product or data services organizations need to integrate large amounts of
data from diverse sources. Only combining such data will lead to powerful models.

The central place for consolidating and storing data is often a Hadoop cluster. At first, most organizations
start to collect all data in raw farm, building what is sometimes called a "data lake". 

(!Insert figure Data Lake)

In most cases however, users will need to request fast access already aggregated and enriched views on the data. Therefore, data management teams will add more and more interdependent tables. If for example, raw clickstream data is ingested, a derived table will contain sessions and visit. Those visits can be joined with product data to create the base table for recommendations and so on. Such a platform is sometimes referred to as "data hub" with different layers for raw, preprocessed and aggregated data.

(!insert figure datahub)

This way large organizations will soon have hundreds of interdependent tables in their Hadoop data warehouse.

# Difficulties with traditional tools like Oozie

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

Oozie will spawn a seperate Mapreduce job for each job that is going to be launched. Besides the latency this induces, each such launcher job will allocate 2 additional YARN containers for the Application Manager and the Mapper auf the launch jobs

## Oozie’s logging and error messages are cryptic and meaningless

E1011?



# How Schedoscope tackles most of these issues

## You define your data, not the workflow

Schedoscope encourages you to think in data, views on that data and dependencies between your views. This makes it much more easy to understand the semantics of your tables. Because you define what your view is supposed to contain, testing is quite really straight forward.

## Concise Syntax

Because Schedoscope's Syntax is actually backed by Scala, Schedoscope Views can be defined within an IDE. This has several advantages: The IDE will spot errors in your view definition while you type. 
You specify all of your view in one place. By following the DRY-Principle - do not repeat yourself - Schedoscope
avoids inconsistencies and mismatches between different views.

Schedoscope allows to define common parts of your table as templates (or traits in Scala terminology). If you, for example, use a timestamp column in every table, you may define this as a template.

## Powerful scheduling

Schedoscope tracks the version of your data and your view definitions. The scheduler will automatically detect changes to your data our your transformation and will initiate a recalculation of that data and all views that
depend on it. This way, all data is kept up-to-date automatically.

##  Built-in monitoring

We are currently on the way to build powerful integrations with Graphite and other monitoring tools,
stay tuned!

