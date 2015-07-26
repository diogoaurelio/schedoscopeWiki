# Building a data platform for data driven products

For building a data driven product or data services organizations need to integrate large amounts of
data from diverse sources. Only combining such data will lead to powerful models.

The central place for consolidating and storing data is often a Hadoop cluster. At first, most organizations
start to collect all data in raw farm, building what is sometimes called a "data lake". 

(!Insert figure Data Lake)

In most cases however, users will need to request fast access already aggregated and enriched views on the data. Therefore, data management teams will add more and more interdependent tables. If for example, raw clickstream data is ingested, a derived table will contain sessions and visit. Those visits can be joined with product data to create the base table for recommendations and so on. Such a platform is sometimes referred to as "data hub" with different layers for raw, preprocessed and aggregated data.

(!insert figure datahub)

This way large organizations will soon have hundreds of interdependent tables in their Hadoop data warehouse.

Managing such a platform can be difficult, because of its complexity. If data in the base table changes, all tables depending in them need to be recalculated. Moreover, defining and modeling the workflows and dependencies in tools like oozie is error-prone because it is scattered to various XML configuration files.

Moreover, with different people working on such a data hub, automatic testing and deployment becomes important to ensure integrity between the all the tables.

After facing these challenges, Otto Group BI developed Schedoscope as a holistic solution for creating a datahub.

With Schedoscope you just define your data and its dependencies in one place, the view definition.
