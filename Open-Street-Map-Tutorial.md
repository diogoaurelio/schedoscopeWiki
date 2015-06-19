_**This Tutorial is under construction.**_

This Tutorial can be used without having your own hadoop cluster at hand. For testing the examples on your own cluster please read section [[Adaptation|Open Street Map Tutorial#adaptation]].

# Goals
* Get Schedoscope tutorial running in Cloudera VM 
* Watch Schedoscope working, play around
* Get Schedoscope tutorial running with your own Hadoop Cluster
* Get familiar with Schedoscope test framework
* Implement and schedule your own views

# Prerequisites
* basic knowledge of [Apache Hive](http://hive.apache.org/)

# The Storyline
Imagine you had a fantastic business idea. You would like to open a new shop in Hamburg, Germany. So you need a place. Why not use the rooms of one of the numerous shops in Hamburg? However... **which shop in Hamburg is the one located best?**

For implementing this tutorial we used data of [Open Street Map](http://www.openstreetmap.org/copyright) limited on Hamburg, Germany. The data is stored in TSV files and provided by archive schedoscope-tutorial-osm-data.

**The raw data structure:**

        nodes  -- points on the map defined by longitude and latitude 

          id BIGINT
          version INT
          user_id INT
          tstamp TIMESTAMP
          changeset_id BIGINT
          longitude DOUBLE
          latitude DOUBLE

        node_tags  -- tags describing a node's feature
          node_id BIGINT
          k STRING
          v STRING

**The measures:**

* The more _restaurants_ are around, the more customers will show up. (+)
* The more _trainstations_ are around, the more customers will show up. (+)
* The more other _shops_ are around, the less customers will show up. They rather might go to the competitors. (-)


**The auxiliary measure:**
Two nodes are close to each other if they lie in the same area. A node's area is defined by the first 7 characters of `GeoHash.geoHashStringWithCharacterPrecision(longitude, latitude)`.

**The execution plan:**

1. Calculate each node's geohash
2. Filter for restaurants, trainstations and shops 
3. Provide aggregated data such that the measures can be applied; the aggregation is stored in view `schedoscope.example.osm.datamart/ShopProfiles`

Finally, find the best location for your shop by analyzing view `schedoscope.example.osm.datamart/ShopProfiles`.


# Installation
Let's get started:

1. Download [Cloudera Quickstart VM](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cloudera_quickstart_vm.html)
2. Start Cloudera VM.
3. Open a terminal and clone the Schedoscope git repository:

    `[cloudera@quickstart ~]$ git clone https://github.com/ottogroup/schedoscope.git`
4. Change directory to `schedoscope` and build the project:

    `[cloudera@quickstart schedoscope]$ mvn install`

# Execution

1. Change directory to `~/schedoscope/schedoscope-tutorial` and execute the tutorial:

    `[cloudera@quickstart schedoscope-tutorial]$ mvn exec:java`
2. The Schedoscope Shell opens in the terminal. Find the full [[command reference|Command Reference]] in the Schedoscope wiki.
3. Type  `materialize -v schedoscope.example.osm.datamart/ShopProfiles`

    and see how all data is digested which is needed in order to provide the requested view of shop profiles.
4. Type  `actions`
    and see which kind of transformations are running.
5. Type  `views`
    and see which views are already materialized with current data.

        schedoscope> views
        Starting VIEWS ...

        RESULTS
        =======
        Details:
        +-----------------------------------------------------+--------------+-------+
        |                         VIEW                        |    STATUS    | PROPS |
        +-----------------------------------------------------+--------------+-------+
        | schedoscope.example.osm.processed/NodesWithGeohash/ | materialized |       |
        |     schedoscope.example.osm.processed/Nodes/2014/04 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/10 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/07 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/01 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2015/05 | transforming |       |
        |        schedoscope.example.osm.datahub/Restaurants/ |      waiting |       |
        |     schedoscope.example.osm.processed/Nodes/2015/02 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/12 | transforming |       |
        |              schedoscope.example.osm.datahub/Shops/ |      waiting |       |
        |     schedoscope.example.osm.processed/Nodes/2014/09 | transforming |       |
        |                schedoscope.example.osm.stage/Nodes/ | materialized |       |
        |     schedoscope.example.osm.processed/Nodes/2014/03 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/06 | transforming |       |
        |      schedoscope.example.osm.datamart/ShopProfiles/ |      waiting |       |
        |             schedoscope.example.osm.stage/NodeTags/ | materialized |       |
        |     schedoscope.example.osm.processed/Nodes/2015/04 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/05 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/11 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2015/01 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/08 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2014/02 | transforming |       |
        |      schedoscope.example.osm.datahub/Trainstations/ |      waiting |       |
        |     schedoscope.example.osm.processed/Nodes/2015/03 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2015/06 | transforming |       |
        |     schedoscope.example.osm.processed/Nodes/2013/12 | transforming |       |
        +-----------------------------------------------------+--------------+-------+
        Total: 26

        materialized: 3
        transforming: 19
        waiting: 4

6. Have a look in the browser at the application manager of your Hadoop Cluster  <http://localhost:8088/cluster>
    and see the MR-Jobs running on the cluster.
9. Type `shutdown` if you want to stop Schedoscope.

# Exploration
7. Open a new terminal. Use the hive CLI to see the data generated by triggering the ETL with Schedoscope
    1. list the databases; _the database name looks like_ `{environment}_{packagename}` _whereas the the dots in the packagename are replaced by underscores, the environment is set in `schedoscope.conf`_
    2. list the tables of a database; _a table name is the name of a class extending `View` in lower case whereas e.g. `NodesWithGeohash` becomes a hive table named `nodes_with_geohash`_
    3. list the columns of a table; _column names are the same as the field names defined in the class, same rule as for table names apply_
    4. list the first 10 entries of a table
    5. perform an analysis on the data provided
7. Obviously every tutorial table does contain columns `id`, `created_at` (when was the data loaded) and `created_by` (which Job provided the data). These fields are set with [[traits|View Traits]] which are predefined fields.
7. Type  `invalidate -v schedoscope.example.osm.datahub/Restaurants` in the schedoscope shell.
    This is how to manually tell Schedoscope that this view shall be recalculated.

        schedoscope> views
        Starting VIEWS ...

        RESULTS
        =======
        Details:
        +-----------------------------------------------------+--------------+-------+
        |                         VIEW                        |    STATUS    | PROPS |
        +-----------------------------------------------------+--------------+-------+
        | schedoscope.example.osm.processed/NodesWithGeohash/ | materialized |       |
        |     schedoscope.example.osm.processed/Nodes/2015/05 | materialized |       |
        |        schedoscope.example.osm.datahub/Restaurants/ |  invalidated |       |
        |      schedoscope.example.osm.datamart/ShopProfiles/ | materialized |       |
...

        |     schedoscope.example.osm.processed/Nodes/2015/06 | materialized |       |
        |     schedoscope.example.osm.processed/Nodes/2013/12 | materialized |       |
        +-----------------------------------------------------+--------------+-------+
        Total: 26

        materialized: 25
        invalidated: 1

8. Type  `materialize -v schedoscope.example.osm.datamart/ShopProfiles`

    Type  `views` to see that only `demo_schedoscope_example_osm_datahub.restaurants` and its depending view `demo_schedoscope_example_osm_datamart.ShopProfiles` are recalculated. Schedoscope knows that the other views' data still is up-to-date.

        | schedoscope.example.osm.processed/NodesWithGeohash/ | materialized |       |
        |     schedoscope.example.osm.processed/Nodes/2014/04 | materialized |       |
        |     schedoscope.example.osm.processed/Nodes/2015/05 | materialized |       |
        |        schedoscope.example.osm.datahub/Restaurants/ | transforming |       |
        |              schedoscope.example.osm.datahub/Shops/ | materialized |       |
        |                schedoscope.example.osm.stage/Nodes/ | materialized |       |
        |      schedoscope.example.osm.datamart/ShopProfiles/ |      waiting |       |
        |             schedoscope.example.osm.stage/NodeTags/ | materialized |       |
        |      schedoscope.example.osm.datahub/Trainstations/ | materialized |       |
        +-----------------------------------------------------+--------------+-------+
        Total: 26

        materialized: 24
        transforming: 1
        waiting: 1
8. Switch to hive CLI and compare column `created_at` of `restaurants` and `shops`. As you can see table `restaurants` has been written again during recalculation. Table `shops` has not been touched.
9. Have a look at the logfile `schedoscope/schedoscope-tutorial/target/logs/schedoscope.log`.
9. Type `shutdown` if you want to stop Schedoscope.


#Adaptation
Thus the example code is running now in Cloudera Quickstart VM. Try get it running with your own hadoop cluster now.
 
Simply install the Schedoscope tutorial on your own machine (see [[Installation|Open Street Map Tutorial#installation]] step 3 and 4). Then change the [[configuration settings|Configuring Schedoscope]] in `schedoscope-tutorial/src/main/resources/schedoscope.conf` as follows:

**VM's schedoscope.conf:**

    schedoscope {
      app {
         environment = "demo"
      }

    transformations = {
      hive : {
              libDirectory = "/home/cloudera/schedoscope/schedoscope-tutorial/target/hive-libraries"
              concurrency = 2                # number of parallel actors to execute hive transformations
        }
      }
    }

**your new schedoscope.conf:**

    schedoscope {
      app {
         environment = "test"    # choose your favourite environment name
      }

      metastore {
        metastoreUri = "your/hive/metastore/uri"
        jdbcUrl = "your/hive/jdbc/uri"
      }

      kerberos {
        principal = "your/kerberos/principal"   # if needed
      }
      
      transformations = {
      	hive : {
		    libDirectory = "/your/absolute/path/to/schedoscope/schedoscope-tutorial/target/hive-libraries"
		    url = ${schedoscope.metastore.jdbcUrl}
        }
      }
    }
The [[default configuration settings|Configuring Schedoscope]] are derived from `schedoscope-core/src/main/resources/reference.conf`. They are overwritten by the settings you define in your project's `schedoscope.conf`.

Change directory to `schedoscope/schedoscope-tutorial` and [[execute|Open Street Map Tutorial#execution]] the tutorial using your own hadoop cluster:

    [cloudera@quickstart schedoscope-tutorial]$ mvn exec:java


# Extension
Now it's time to design your own views and their dependencies.

## Preparation
1. You need a Scala IDE, e.g. [Scala IDE for Eclipse](http://scala-ide.org/download/sdk.html)
2. Import the maven project schedoscope-tutorial
3. Set scala compiler on version 2.10 in the project's properties
4. Sometimes the scala library needs to be added to the project in the IDE manually; right click on the project go to Scala > Add Scala Library to Build Path

## Development
Use other Open Street Map TSV-files provided by schedoscope-tutorial-osm-data:

* `ways.txt`  (way is a sequence of 2-2000 nodes)

        id BIGINT
        version INT
        user_id INT
        tstamp TIMESTAMP
        changeset_id BIGINT

* `way_nodes.txt`  (mapping of nodes on ways)

        way_id BIGINT
        node_id BIGINT
        sequence_id INT

* `way_tags.txt`  (tags for ways)

        way_id BIGINT
        k STRING
        v STRING

These files can be read in from classpath. Have a look at the tutorial class `schedoscope.example.osm.stage.Nodes` .

The custom [Test Framework](Test Framework) allows **test-driven development** for schedoscope.

1. Create a stub for your view.
2. Implement your testclass.
3. Implement your view while testing its behaviour using the provided test framework.

Based on your new views `ways`, `way_tags` and `way_nodes` you can implement a view `schedoscope.example.osm.processed.Ways` that comprises all information about ways according to view `schedoscope.example.osm.processed.Nodes`. Therefore, use UDF `collect` to create a map of tags as in `Nodes` and an array of node_id which are part of this way.

## Deployment
Restart Schedoscope (your project).


#Scheduling 
Schedoscope is a Webservice with [[REST API|Schedoscope REST API]]. If you want your data up-to-date every hour during business time (8am-8pm) simply register cronjobs like:

       5   8-20 * * *  curl http://localhost:20698/materialize/schedoscope.example.osm.datamart/ShopProfiles

This is the default webservice configured in `schedoscope/schedoscope-core/src/main/resources/reference.conf`, the command `materialize` and the [[View URL|View Pattern Reference]] of the view you're interested in. Reloading the requested view `ShopProfiles` only starts if any of its source views `ShopProfiles` depends on has changed!


# Hints

