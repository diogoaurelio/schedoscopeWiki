
This tutorial can be used without having your own Hadoop cluster at hand. For getting the examples to run on your own cluster please read section [[Adaptation|Open Street Map Tutorial#adaptation]].

## Goals
The goals of this tutoral are:

* get the Schedoscope [[tutorial code running|Open Street Map Tutorial#installation]] in the Cloudera Quickstart VM; 
* watch Schedoscope working, [[play around|Open Street Map Tutorial#exploration]];
* get Schedoscope tutorial running [[with your own Hadoop Cluster|Open Street Map Tutorial#adaptation]];
* get familiar with [[Schedoscope test framework|Open Street Map Tutorial#exploring-the-test-framework]];
* implement and schedule [[your own views|Open Street Map Tutorial#development]].

Find a list of [[hints|Open Street Map Tutorial#hints]] at the end of this tutorial.

## Prerequisites
* basic knowledge of [Apache Hive](http://hive.apache.org/)

## The Story
Imagine you had a fantastic business idea. You would like to buy a promising shop in Hamburg, Germany. However... **which shop in Hamburg is the one located best?**

For implementing this tutorial, we use data of [Open Street Map](http://www.openstreetmap.org/copyright). The data is stored in TSV files and provided by the dependency `schedoscope-tutorial-osm-data` and looks like this:

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

For measuring the "best" shop location, we assume the following:
* The more _restaurants_ are around, the more customers will show up. (+)
* The more _trainstations_ are around, the more customers will show up. (+)
* The more other _shops_ are around, the less customers will show up. They rather might go to the competitors. (-)


To measure distance, we use a geo hash. Two nodes are close to each other if they lie in the same area. A node's area is defined by the first 7 characters of `GeoHash.geoHashStringWithCharacterPrecision(longitude, latitude)`.

## The Plan

1. Calculate each node's geohash

2. Filter for restaurants, trainstations and shops 

3. Provide aggregated data such that the measures can be applied; the aggregation is stored in view `schedoscope.example.osm.datamart/ShopProfiles`

Finally, you can find the best location for your shop by analyzing `schedoscope.example.osm.datamart/ShopProfiles`.

![dwh-structure.jpg](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-tutorial/docs/pictures/dwh-structure.jpg)

## Installation
Let's get started:

1. Download [Cloudera Quickstart VM](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cloudera_quickstart_vm.html)

2. Start Cloudera VM.

3. Open a terminal, create the base directory and set the appropriate permissions in HDFS:

        [cloudera@quickstart ~]$ sudo su - hdfs
        -bash-4.1$ hdfs dfs -mkdir /hdp /user/cloudera
        -bash-4.1$ hdfs dfs -chown -R cloudera:cloudera /hdp /user/cloudera
        -bash-4.1$ exit

4. Clone the Schedoscope git repository:

    `[cloudera@quickstart ~]$ git clone https://github.com/ottogroup/schedoscope.git`

5. Change directory to `schedoscope` and build the project:

    `[cloudera@quickstart schedoscope]$ mvn install`


Hint: The Cloudera QuickStart VM comes with low memory settings for YARN/MapReduce which can result in memory problems. It is recommended to check the schedoscope and YARN logs. In case of `OutOfMemory` exceptions increase the following values:

        YARN: Cloudera Manager -> Hive -> Configuration -> mapreduce.map.memory.mb, mapreduce.reduce.memory.mb, mapreduce.map.java.opts.max.heap, mapreduce.reduce.java.opts.max.heap, yarn.nodemanager.resource.memory-mb, yarn.scheduler.maximum-allocation-mb
        Hive: Cloudera Manager -> Hive -> Configuration -> HiveServer2 Heap Size

It is also recommended to limit the number of simultaneously running applications to 2:

        YARN Scheduler: Cloudera Manager -> Clusters -> Resource Management -> Dynamic Resource Pools -> Configuration -> Edit -> YARN


## Execution

1. Go into directory `~/schedoscope/schedoscope-tutorial` and execute the tutorial:

    `[cloudera@quickstart schedoscope-tutorial]$ mvn exec:java`

2. The Schedoscope Shell opens in the terminal. Find the full [[command reference|Command Reference]] in the Schedoscope wiki.

3. Type  `materialize -v schedoscope.example.osm.datamart/ShopProfiles`

    to trigger all data ingestions and view computations leading to the requested view of shop profiles.

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

6. Type `actions -s running` to see what is going on.

6. Have a look in the browser at the resource manager of your Hadoop Cluster  <http://localhost:8088/cluster>
    and see whether you can identify the running actions on the cluster.

6. You can also take a look at the activity in the log file `~/schedoscope/schedoscope-tutorial/target/logs/schedoscope.log`

9. Type `shutdown` if you want to stop Schedoscope.

## Exploration

7. Open a new terminal. Use the hive CLI to see the data generated by triggering the ETL with Schedoscope
    1. list the databases; _the database name looks like_ `{environment}_{packagename}` _whereas the the dots in the packagename are replaced by underscores, the environment is set in `schedoscope.conf`_
    2. list the tables of a database; _a table name is the name of a class extending `View` in lower case whereas e.g. `NodesWithGeohash` becomes a hive table named `nodes_with_geohash`_
    3. list the columns of a table; _column names are the same as the field names defined in the class, same rule as for table names apply_
    4. list the first 10 entries of a table
    5. perform an analysis on the data provided

    As one can see, every tutorial table does contain columns `id`, `created_at` (when was the data loaded)
    and `created_by` (which Job provided the data). These fields are set with [[traits|View Traits]] which are 
    predefined fields.

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


##Adaptation
The tutorial is now in a Cloudera Quickstart VM. You should try to get it running with your own hadoop cluster.
 
Simply install the Schedoscope tutorial on your own machine (see [[Installation|Open Street Map Tutorial#installation]] step 3 and 4). Then change the [[configuration settings|Configuring Schedoscope]] in `schedoscope-tutorial/src/main/resources/schedoscope.conf` as follows:

**VM's schedoscope.conf:**

    schedoscope {
      app {
        environment = "demo"
      }

      metastore {
        jdbcUrl = "jdbc:hive2://localhost:10000/default;user=cloudera;password=cloudera"
      }

      transformations = {
        hive : {
          libDirectory = "/home/cloudera/schedoscope/schedoscope-tutorial/target/hive-libraries"
          concurrency = 2                # number of parallel actors to execute hive transformations
          url = "jdbc:hive2://localhost:10000/default;user=cloudera;password=cloudera"
        }
      }
    }

**your new schedoscope.conf:**

    schedoscope {
      app {
         environment = "test"    # read about the environment name below
      }

      hadoop {
         resourceManager = "yourhost:yourport"
         nameNode = "yourhost:yourport"
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

The chosen environment name is set as root HDFS folder for all data processed by schedoscope. The full path looks like `/hdp/${env}/${package_name}/${ViewName}`.

Change directory to `schedoscope/schedoscope-tutorial` and [[execute|Open Street Map Tutorial#execution]] the tutorial using your own hadoop cluster:

    [cloudera@quickstart schedoscope-tutorial]$ mvn exec:java


## Extension

Now it's time to design your own views and their dependencies. For this purpose, you might also want to have a look at the [View DSL Primer](Schedoscope View DSL Primer).

### Preparation

1. You need a Scala IDE, e.g. [Scala IDE for Eclipse](http://scala-ide.org/download/sdk.html)

2. Import the maven projects schedoscope-core and schedoscope-tutorial

3. Set scala compiler on version 2.10 in the projects' properties

4. Sometimes the scala library needs to be added to the project in the IDE manually; right click on the project go to Scala > Add Scala Library to Build Path

5. Sometimes the scala folders need to be added as source folders manually; right click on the project go to Build Path > Configure Build Path, then choose "Java Build Path" on the left menu and tab "Source", click "Add Folder" and select the missing folders `src/main/scala` and `src/test/scala`.

### Exploring the Test Framework

The custom [Test Framework](Test Framework) of Schedoscope facilitates quick testing of code. For each test called a self-contained local hadoop installation is set up in `${baseDir}/target/hadoop`.

1. Set environment variables HADOOP_HOME to `~/schedoscope/schedoscope-tutorial/target/hadoop` and JAVA_HOME.
![test_run_configurations](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-tutorial/docs/pictures/test_run_configurations.png)

2. You can run a test by right clicking on the test class in the Scala IDE's package explorer and choosing Run As > Scala Test - File

3. When building the project (as in [[Installation|Open Street Map Tutorial#installation]] Step 4) you can see how all tests are executed.

### Development

There are other Open Street Map TSV-files provided by the dependency `schedoscope-tutorial-osm-data`:

* `ways.txt`  (a way is a sequence of 2-2000 nodes)

        id BIGINT
        version INT
        user_id INT
        tstamp TIMESTAMP
        changeset_id BIGINT

* `way_nodes.txt`  (mapping nodes to ways)

        way_id BIGINT
        node_id BIGINT
        sequence_id INT

* `way_tags.txt`  (tags for ways)

        way_id BIGINT
        k STRING
        v STRING

You should create stage views `schedoscope.example.osm.stage.Ways`, `schedoscope.example.osm.stage.WayNodes`, and `schedoscope.example.osm.stage.WayTags` that capture those files by reading them from the classpath. The tutorial class `schedoscope.example.osm.stage.Nodes` can serve as your template for how this can be done.

You can implement a view `schedoscope.example.osm.processed.Ways` that comprises all information about ways. This view should have `schedoscope.example.osm.processed.Nodes` as well as the new stage views you created above as dependencies. Establishing a monthly partitioning from `tstmp` would also be appropriate. You can use a Hive transformation that uses the brickhouse UDF `collect` to create a map of tags and an array of with all `node_id` which each way comprises. Take look at `Nodes` for a template of how to use `collect`. 

### Deployment

Rebuild and restart Schedoscope:

    mvn install
    mvn exec:java

In case of trouble have a look at the logfile `schedoscope/schedoscope-tutorial/target/logs/schedoscope.log`.


## Scheduling 

Schedoscope is a Webservice with [[REST API|Schedoscope REST API]]. If you want your data up-to-date every hour during business time (8am-8pm) simply register cronjobs like:

       5   8-20 * * *  curl http://localhost:20698/materialize/schedoscope.example.osm.datamart/ShopProfiles

This is the default webservice configured in `schedoscope/schedoscope-core/src/main/resources/reference.conf`, the command `materialize` and the [[View URL|View Pattern Reference]] of the view you're interested in. Reloading the requested view `ShopProfiles` only starts if any of its source views `ShopProfiles` depends on or `ShopProfiles` itself has changed!


## Hints
* Defining a field of type BIGINT in hive, works with LONG in Schedoscope: `val id = fieldOf[Long]`
* Testing views of the same name (but different layers): Run the test by right clicking on the word "should" in the test class source code and choosing Run As > Scala Test - Test
* Schedoscope shell cannot cope with leading white spaces. It will prompt a list of possible commands.

## Acknowledgement
Our example data was taken from the [Open Street Map project](http://www.openstreetmap.org/copyright)