Metascope is a web application which allows you to discover, search, annotate and document your Hadoop datahub.

Metascope uses the internal view descriptions from Schedoscope to build a metadata repository. A lightweight web interface and a REST API exposes all critical metadata to the user.

Metascope offers
* a **complete overview of all views** of your datahub
* an search index which allows **faceted navigation** and **full-text search**
* **data lineage** throughout all views
* an detailed overview of **all critical metadata** in one spot
* **collaboration features**: create in-line documentation and comments on many different levels
* the possibility to create **custom taxonomies** and to **tag and categorize** your data
* data samples, data distribution and many more ...

In the following, we will present and describe each feature of Metascope in detail. 

## Faceted navigation and full-text search
Metascope uses a relational database to store the Schedoscope data and further metadata from various Hadoop frameworks. For navigation and search purposes, all data is indexed by [Apache Solr](http://lucene.apache.org/solr/). This allows Metascope to offer the user a faceted navigation and a full-text search.

![Navigation](images/filtersearch.png)

The faceted navigation gives the user a simple mechanism to find a specific view in seconds. The full-text search can be used to refine the search results. The facets can be combined arbitrary and always show the number of resulting hits.

There are facets for:
* **Type** (Table or Partition)
* **Database name**
* **Table name**
* **Table status**
* **Table transformations**
* **Table exports**
* **Creation time**
* **Last transformation time**
* [**Taxonomies and tags**](#taxonomies-and-tags)
* **Partition parameters**

The full-text search also offers a auto-complete function depending on the indexed data to help the user to get more and precise search results.

## Rich inline documentation
Metascope gives you the ability to document your view. A WYSIWYG editor allows to create rich documentation (Fonts, Size, Colors, HTML structures, ...) and stores it right at your view. There is no need to use external portals or wikis like [Confluence](https://www.atlassian.com/software/confluence) anymore, which allows more consistent documentation of your datahub.

![WYSIWYG documentation](images/docu.png)

Besides documenting on table level, each table field and parameter can be documented similarly.

## Taxonomies and tags
Taxonomies can be created, maintained, and assigned to the datasets. This allows to categorize your views throughout the entire datahub and increase the search and browsing capabilities for the users.

![Taxonomy administration](images/ctaxonomy.png)

Each taxonomy consists of a taxonomy name and a set of categories, which in turn also have a name and a set of category objects. The category object is an entity which can be assigned to a view. It contains a object name and an suitable description.

![Add taxonomy and tags](images/taxonomy.png)

Users are able to assign appropriate category objects to the datasets via a fixed drop-down menu. Metascope will automatically assign the parent taxonomies and categories to the view. If no suitable taxonomy or category object is present, the user is able to assign free tags. The taxonomy information is present at the taxonomy section as well as in the faceted navigation. The user is now able to query for specific taxonomies, categories and/or category objects. 

## View schema
The schema section shows the table structure of the view. It displays the field name, type and description. Just like on table level, the users are able to create documentation for each field.

![Schema](images/schema.png)

If the view is partitioned, a seperate section named 'Parameters' shows a similar table with the partition parameters.

## Data Lineage
One of the key features of Metascope is the data lineage. The views have explicit definition of dependencies, which makes it easy for Metascope to gather the dependency information and display it to the users. There is no need for retroactive crawling through the Hadoop logs to find data lineage information, which could be inaccurate.

![Dependencies](images/dependency.png)

The dependency section shows all views which are used by as well as depend on the selected view. It is indispensable to know the data flow through the entire datahub to rely on analysis and reports. 

![Lineage](images/lineage.png)

The default table will only show direct dependencies. *Show transitive dependencies* will recursively navigate through the lineage graph and find all direct and indirect dependencies. This could help the developers when they have to change the schema of a view and need to find all views which will be affected by the change.

The lineage graph is a visual representation of the data lineage information. It helps the user to get a better understanding of the relations between the datasets and allows him directly to navigate to depended or subsequent views. 

## Table partitions
This section displays all partitions on the table. It shows the partition parameter and their values, the status of the partition, the last successful transformation and the duration.

![Partitions](images/partitions.png)

Furthermore detailed lineage information on partition level is given. The table shows for each partition the direct dependencies and successors. This enables the user to navigate through the data on partition level. 

## Data sample and distribution
The sample sections shows a small (ten rows) data sample of the current table. If the table is partitioned, the user is able to retrieve a sample for a specific parameter or partition.

![Data sample](images/sample.png)

Metascope will automatically generate insights into your data. Automatic jobs will calculate different metrics for each table field, interpret and displays them to the user.

![Data distribution](images/data.png)

The calculation depend on the field type. Following metrics are created for the appropriate types:
* int/double/long: min, max, average, standard derivation, sum
* boolean/String: min, max, enum (If the field contains <255 distinct values, Metascope will interpret the field as an enumeration and displays the values)

## View storage and transformation information
The storage section shows technical metadata and storage information

![Storage](images/storage.png)

Following metric are displayed if available:
* **Owner**: the techincal user which owns the underlying data files
* **Permissions**: the permissions of underlying data files
* **Storage format**: storage format defined in Schedoscope
* **Input format**: the appropriate input format from the Hive metastore
* **Output format**: the appropriate output format from the Hive metastore
* **Location**: the location of the underlying data files
* **Size**: the total size of the underlying data files
* **Partitions**: the total amount of partitions for this view
* **Table created**: timestamp of table creation (in Hive Metastore)
* **Last transformation**: last materialization of the view
* **Last data from**: the latest data row, which illustrates the up-to-dateness of the view (see [View administration](#view-administration))
* **Last partition created**: timestamp of last partition creation (in Hive Metastore)

The transformation describes the job which is used to calculate the view.

![Transformation](images/transformation.png)

Schedoscope allows to [export the views](https://github.com/ottogroup/schedoscope/wiki/Schedoscope%20View%20DSL%20Primer#view-exports) into numerous systems. Each export and its parameters will be displayed next to the transformation table.

## Collaboration
Metascope comes with various collaborative features: 
* Create documentation for each view, its fields and its parameters
* Create taxonomies and categorize the data with category objects and free tags
* Discuss a certain view with your colleagues and create comments on documentation
* The activity stream shows the recent changes and discussions in Metascope 
* Most viewed datasets shows the most clicked views
* Add datasets to your 'Favourites' to create a short cut to your views

## User management
Metascope has a custom user management. As an admin, you are able to create, edit and delete users to administrate access and privileges. Furthermore Metascope comes with an LDAP integration: Check [the configuration](https://github.com/ottogroup/schedoscope/wiki/Configuring%20Metascope) on how to use LDAP with Metascope

## View administration
* **Invalidate and materialize views**: As an admin user, you are able to trigger invalidation and materialization commands from Metascope.
* **Set a data owner**: Besides the technical user, an 'data owner' or 'person responsible' can be set for each view. This feature can be found on the documentation section and allows to enter the name of a contact person for the dataset. If the user is registered in Metascope (a list of users will be suggested), further information will be displayed.
* **Set the timestamp field**: The storage sections contains the metadata 'Last data from', which shows the timestamp of the *latest* record. Metascope tries to guess the timestamp field and its format, but sometimes the information needs to be set manually. This is possible in the 'Administration' section.
(e.g. you have a table named *sales* with a field called *ts* which contains a timestamp with ISO 8601 format, say *YYYY-MM-DDThh:mm:ss*. Set the timestamp field to *ts* and the timestamp format to *YYYY-MM-DD'T'hh:mm:ss*)