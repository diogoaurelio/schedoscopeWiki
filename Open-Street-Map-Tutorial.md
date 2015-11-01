## Goals
The goals of this tutorial are to:

* get the Schedoscope [[tutorial code running|Open Street Map Tutorial#installing]] in the Cloudera Quickstart VM; 
* [[watch Schedoscope doing its work|Open Street Map Tutorial#watching-schedoscope-work]];
* [[explore the results|Open Street Map Tutorial#exploring-the-results]];
* watch Schedoscope [[dealing with change|Open Street Map Tutorial#dealing-with-change]
* get [[the tutorial running on your own Hadoop Cluster|Open Street Map Tutorial#running-on-a-real-cluster]];
* get familiar with [[Schedoscope test framework|Open Street Map Tutorial#exploring-the-test-framework]];
* implement and schedule [[your own views|Open Street Map Tutorial#development]].

You can find a list of [[hints|Open Street Map Tutorial#hints]] at the end of this tutorial.

## Prerequisites
* Basic knowledge of [Apache Hive](http://hive.apache.org/)

## The Story
Imagine you had a fantastic business idea. You would like to buy a promising shop in Hamburg, Germany. However... **which shop in Hamburg is the one located best?**

For implementing this tutorial, we use geospatial data from the [Open Street Map](http://www.openstreetmap.org/copyright) project. The data is available in two TSV files, which for convenience we provide by the Maven artificat `schedoscope-tutorial-osm-data`. The structure of the files looks like this:

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

### The Plan

1. Calculate each node's geohash

2. Filter for restaurants, trainstations and shops 

3. Provide aggregated data such that the measures can be applied; the aggregation is stored in view `schedoscope.example.osm.datamart/ShopProfiles`. You can then find the best location for your shop by analyzing `schedoscope.example.osm.datamart/ShopProfiles`.

For the tutorial, we translated this plan into a hierarchy of interdependent Schedoscope views, which means Hive tables:
 
![dwh-structure.jpg](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-tutorial/docs/pictures/dwh-structure.jpg)

## Installing
Let's get started:

1. Download [Cloudera Quickstart VM 5.4](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cloudera_quickstart_vm.html)

2. Start Cloudera VM.

3. Open a terminal, create the base directory and set the appropriate permissions in HDFS:

        [cloudera@quickstart ~]$ sudo su - hdfs
        -bash-4.1$ hdfs dfs -mkdir /hdp /user/cloudera
        -bash-4.1$ hdfs dfs -chown -R cloudera:cloudera /hdp /user/cloudera
        -bash-4.1$ exit

4. Clone the Schedoscope git repository:

         [cloudera@quickstart ~]$ cd ~
         [cloudera@quickstart ~]$ git clone https://github.com/ottogroup/schedoscope.git

5. Go into directory `schedoscope` and build the project:

         [cloudera@quickstart schedoscope]$ cd ~/schedoscope
         [cloudera@quickstart schedoscope]$ mvn install


Hint: The Cloudera QuickStart VM comes with low memory settings for YARN/MapReduce which can result in memory problems. It is recommended to check the schedoscope and YARN logs. In case of `OutOfMemory` exceptions increase the following values:

        YARN: Cloudera Manager -> Hive -> Configuration -> mapreduce.map.memory.mb, mapreduce.reduce.memory.mb, mapreduce.map.java.opts.max.heap, mapreduce.reduce.java.opts.max.heap, yarn.nodemanager.resource.memory-mb, yarn.scheduler.maximum-allocation-mb
        Hive: Cloudera Manager -> Hive -> Configuration -> HiveServer2 Heap Size

It is also recommended to limit the number of simultaneously running applications to 2:

        YARN Scheduler: Cloudera Manager -> Clusters -> Resource Management -> Dynamic Resource Pools -> Configuration -> Edit -> YARN


## Watching Schedoscope work

1. Launch Schedoscope:

        [cloudera@quickstart schedoscope]$ cd ~/schedoscope/schedoscope-tutorial
        [cloudera@quickstart schedoscope-tutorial]$ mvn exec:java

2. The Schedoscope Shell opens in the terminal. You can find the full [[command reference|Command Reference]] in the Schedoscope wiki.

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

6. Type `transformations -s running` to see what is going on.

6. Take a look at the resource manager of the virtual Hadoop Cluster in Firefox (http://localhost:8088/cluster)
    and see whether you can identify the running actions on the cluster.

6. Opening another terminal window, you can also take a look at the activity in the log file:

        [cloudera@quickstart ~]$ tail -F ~/schedoscope/schedoscope-tutorial/target/logs/schedoscope.log

9. Type `shutdown` or `^C` in the Schedoscope shell if you want to stop Schedoscope. You should wait for it to complete the materialization of views to continue with the tutorial though.

## Exploring the results

7. Open a new terminal. Use the Hive CLI to see the data materialized by Schedoscope:

    * Start Hive: 

            [cloudera@quickstart ~]$ hive
 
    * List the databases that Schedoscope automatically created: 

            hive> show databases;

    The database names look like `{environment}_{packagename}`, with the the dots in the package name of the materialized views replaced by underscores. The environment is set in `~/schedscope/schedoscope-tutorial/src/main/resources/schedoscope.conf`.

    * List the tables of a database: 

            hive> use demo_schedoscope_example_osm_datamart;
            hive> show tables;

    A table name is the name of the corresponding view class extending base class `View` in lower case. E.g. `NodesWithGeohash` becomes a hive table named `nodes_with_geohash`.

    * List the columns of a table: 

            hive> describe shop_profiles;

    Column names are the same as the names of the fields specified in the corresponding view class, similarly transformed to lower case.

    * List the first 10 entries of a table: 

            hive> select * from shop_profiles limit 10;

    * Take a look around the tables yourself.

    As one can see, every tutorial table does contain columns `id`, `created_at` (when was the data loaded)
    and `created_by` (which Job provided the data). These fields are defined using common  [[traits|View Traits]] carrying predefined fields.

## Dealing with change

One way to deal with change is to explicitly retrigger computation of views:

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

9. Once everything has been materialized, type `shutdown` to stop Schedoscope.

_Way more more interesting is to see Schedoscope discover change all by itself, however!_

7. Make sure all views have been materialized and that you have quit the Schedoscope shell.

7. Go back to the `schedoscope-tutorial` folder: `cd ~/schedoscope/schedoscope-tutorial`

7. Open the query that computes the `Restaurants` view in an editor:

        [cloudera@quickstart schedoscope-tutorial]$ vim src/main/resources/hiveql/datahub/insert_restaurants.sql 

7. From now on, restaurant names are to be uppercase. So wrap the statement

        tags['name'] AS restaurant_name,

   into
   
        ucase(tags['name']) AS restaurant_name,

7. Save your work and go back to the shell.

7. Recompile the tutorial (skipping tests, as the restaurant test will fail now):

        [cloudera@quickstart schedoscope-tutorial]$ mvn install -DskipTests

7. Relaunch Schedoscope:

        [cloudera@quickstart schedoscope-tutorial]$ mvn exec:java

7. Materialize `schedoscope.example.osm.datamart/ShopProfiles/` again:

        schedoscope> materialize -v schedoscope.example.osm.datamart/ShopProfiles

8. Watch Schedoscope rematerialize `schedoscope.example.osm.datahub/Restaurants` and `schedoscope.example.osm.datamart/ShopProfiles` without any explicit migration commands from your side. No other views are recomputed because they are not affected by the change.


## Running on a real cluster
You can get the tutorial running on your own hadoop cluster.
 
Install the Schedoscope tutorial on a gateway machine to your cluster:

1. Clone the source code (see [[Installation|Open Street Map Tutorial#installation]] Step 4).

2. Prepare a `/hdp` folder in your cluster's HDFS and give proper write permissions for the user with which you want to execute Schedoscope. (similar to [[Installation|Open Street Map Tutorial#installation]] Step 3).

3. Then change the [[configuration settings|Configuring Schedoscope]] in `schedoscope-tutorial/src/main/resources/schedoscope.conf` as follows:

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
		    libDirectory = "/your/local/absolute/path/to/schedoscope/schedoscope-tutorial/target/hive-libraries"
		    url = ${schedoscope.metastore.jdbcUrl}
        }
      }
    }

The [[default configuration settings|Configuring Schedoscope]] are derived from `schedoscope-core/src/main/resources/reference.conf`. They are overwritten by the settings you define in your project's `schedoscope.conf`.

The chosen environment name is set as root HDFS folder for all data processed by schedoscope. The full path looks like `/hdp/${env}/${package_name}/${ViewName}`.

4. Compile Schedoscope (similar to [[Installation|Open Street Map Tutorial#installation]] Step 5).

Go into directory `schedoscope/schedoscope-tutorial` and [[execute|Open Street Map Tutorial#execution]] the tutorial using your own hadoop cluster:

    [cloudera@quickstart schedoscope-tutorial]$ mvn exec:java


## Extension

Now it's time to design your own Schedoscope views and their dependencies. For this purpose, you might also want to have a look at the [View DSL Primer](Schedoscope View DSL Primer).

### Preparation

1. You need a Scala IDE, e.g. [Scala IDE for Eclipse](http://scala-ide.org/download/sdk.html)

2. Import the maven projects schedoscope-core and schedoscope-tutorial. 

In case Scala IDE has problems importing the schedoscope-tutorial project, disable the shift of build phases for mixed compilation in the maven-compiler-plugin and scala-maven-plugin configurations in the pom. You can do this by commenting the following section prior to import and then remove the comments after import:

	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-compiler-plugin</artifactId>
		<version>3.3</version>
		<configuration>
			<source>1.7</source>
			<target>1.7</target>
		</configuration>
		<!--
		<executions>
			<execution>
				<id>default-compile</id>
				<phase>none</phase>
			</execution>
			<execution>
				<id>default-testCompile</id>
				<phase>none</phase>
			</execution>
		</executions>
		-->
	</plugin>
	...
	<plugin>
		<groupId>net.alchim31.maven</groupId>
		<artifactId>scala-maven-plugin</artifactId>
		<version>3.2.2</version>
		<configuration>
			<recompileMode>incremental</recompileMode>
			<args>
				<arg>-target:jvm-1.7</arg>
			</args>
			<javacArgs>
				<javacArg>-source</javacArg>
				<javacArg>1.7</javacArg>
				<javacArg>-target</javacArg>
				<javacArg>1.7</javacArg>
			</javacArgs>
		</configuration>
		<!--
		<executions>
			<execution>
				<id>scala-compile</id>
				<goals>
					<goal>compile</goal>
				</goals>
			</execution>
			<execution>
				<id>scala-test-compile</id>
				<goals>
					<goal>testCompile</goal>
				</goals>
			</execution>
		</executions>
		-->
	</plugin>

3. Make sure the scala compiler is set to version 2.11 in the projects' properties

4. Sometimes the scala folders need to be added as source folders manually; right click on the project go to Build Path > Configure Build Path, then choose "Java Build Path" on the left menu and tab "Source", click "Add Folder" and select the missing folders `src/main/scala` and `src/test/scala`.

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