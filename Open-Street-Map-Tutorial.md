## Goals
The goals of this tutorial are to:

* get the [[Schedoscope tutorial code running|Open Street Map Tutorial#installing]] in the Cloudera Quickstart VM; 
* [[watch Schedoscope doing its work|Open Street Map Tutorial#watching-schedoscope-work]];
* [[explore the results|Open Street Map Tutorial#exploring-the-results]];
* [[watch Schedoscope dealing with change|Open Street Map Tutorial#dealing-with-change]];
* get the [[tutorial set up in your IDE|Open Street Map Tutorial#setting-up-scala-ide]];
* get familiar with the [[Schedoscope test framework|Open Street Map Tutorial#exploring-the-test-framework]];
* get the [[tutorial running on your own Hadoop Cluster|Open Street Map Tutorial#running-on-a-real-cluster]];
* show [[how to integrate Schedoscope with cron|Open Street Map Tutorial#scheduling]].

## Prerequisites
* Basic knowledge of [Apache Hive](http://hive.apache.org/)

## The Story
Imagine you had a fantastic business idea. You would like to buy a promising shop in Hamburg, Germany. However... **which shop in Hamburg is the one located best?**

For implementing this tutorial, we use geospatial data from the [Open Street Map](http://www.openstreetmap.org/copyright) project. The data is available in two TSV files, which for convenience we provide by the Maven artifact `schedoscope-tutorial-osm-data`. The structure of the files looks like this:

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
* The more other _shops_ are around, the fewer customers will show up. They rather might go to the competitors. (-)

To measure distance, we use a geo hash. Two nodes are close to each other if they lie in the same area. A node's area is defined by the first 7 characters of `GeoHash.geoHashStringWithCharacterPrecision(longitude, latitude)`.

### The Plan

1. Calculate each node's geohash

2. Filter for restaurants, trainstations, and shops 

3. Provide aggregated data such that the measures can be applied; the aggregation is stored in view `schedoscope.example.osm.datamart/ShopProfiles`. You can then find the best location for your shop by analyzing `schedoscope.example.osm.datamart/ShopProfiles`.

For the tutorial, we translated this plan into a hierarchy of interdependent Schedoscope views, which means Hive tables:
 
![dwh-structure.jpg](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-tutorial/docs/pictures/dwh-structure.jpg)

## Installing
Let's get started:

1. Download [Cloudera Quickstart VM 5.4](http://www.cloudera.com/content/www/en-us/downloads/quickstart_vms/5-4.html). The tutorial has been tested with the VirtualBox variant of it.

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

         [cloudera@quickstart ~]$ cd ~/schedoscope
         [cloudera@quickstart schedoscope]$ mvn install -DXX:MaxPermSize=512m


## Watching Schedoscope work

1. Launch Schedoscope:

        [cloudera@quickstart schedoscope]$ cd ~/schedoscope/schedoscope-tutorial
        [cloudera@quickstart schedoscope-tutorial]$ mvn exec:java

2. The Schedoscope Shell opens in the terminal. You can find the full [[command reference|Scheduling Command Reference]] in the Schedoscope wiki.

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

9. Type `shutdown` or `^C` in the Schedoscope shell if you want to stop Schedoscope. You should wait for it to complete the materialization of views to continue with the tutorial, though.

## Exploring the results

Open a new terminal. Use the Hive CLI to see the data materialized by Schedoscope:

1. Start Hive: 

		[cloudera@quickstart ~]$ hive

2. List the databases that Schedoscope automatically created: 

		hive> show databases;

   The database names look like `{environment}_{packagename}`, with the dots in the package name of the materialized views replaced by underscores. The environment is set in `~/schedscope/schedoscope-tutorial/src/main/resources/schedoscope.conf`.

3. List the tables of a database: 

		hive> use demo_schedoscope_example_osm_datamart;
		hive> show tables;

   A table name is the name of the corresponding view class extending base class `View` in lower case. E.g. `NodesWithGeohash` becomes a hive table named `nodes_with_geohash`.

4. List the columns of a table: 

		hive> describe shop_profiles;

   Column names are the same as the names of the fields specified in the corresponding view class, similarly transformed to lower case.

5. List the first 10 entries of a table: 

		hive> select * from shop_profiles limit 10;

6. Take a look around the tables yourself.

   As one can see, every tutorial table does contain columns `id`, `created_at` (when was the data loaded)
and `created_by` (which Job provided the data). These fields are defined using common  [[traits|View Traits]] carrying predefined fields.

7. Let's take a surprised look at the MySQL server running in the quickstart VM: 
		
		[cloudera@quickstart ~]$ mysql schedoscope_tutorial -u root -pcloudera

		mysql> select * from demo_schedoscope_example_osm_datamart_shop_profiles limit 10;

   A MySQL export has been configured with the `ShopProfiles` view. As a result, not only has `ShopProfiles` been materialized in hive; after transformation, Schedoscope created an equivalent table in MySQL and exported `ShopProfiles` to that table using a mapreduce job. Schedoscope's export module supports simple, parallel export to [JDBC](JDBC Export), [Redis](Redis Export), and [Kafka](Kafka Export).

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

_Way more interesting is to see Schedoscope discover change all by itself, however!_

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

The criteria for detecting changes to transformation logic depend on the respective transformation type. Please have a look at the various transformation type descriptions for more information on this topic.

## Setting up Scala IDE

Now it's time to design your own Schedoscope views and their dependencies. For this purpose, we need to
set up an IDE. 

1. You need a Scala IDE, e.g. [Scala IDE for Eclipse](http://scala-ide.org/download/sdk.html)

2. Import the maven projects schedoscope-core and schedoscope-tutorial. 

In case your Scala IDE has problems importing the schedoscope-tutorial project, disable the shift of build phases for mixed compilation in the maven-compiler-plugin and scala-maven-plugin configurations in the pom. You can do this by commenting the following section prior to import and then removing the comments after import:

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

## Exploring the Test Framework

The custom [Test Framework](Test Framework) of Schedoscope facilitates quick testing of code. 

1. You can run a test by right clicking on a test class in the Scala IDE's package explorer (e.g., `schedoscope-tutorial/src/test/scala/schedoscope/example/osm/datahub/RestaurantsTest.scala`  and choosing `Run As > Scala Test - File`

2. Initially, this will fail because you need to provide a `HADOOP_HOME` environment variable setting. Lookup the IDE's run configuration for your failed test and set `HADOOP_HOME` to `/<yourhomedir>/schedoscope/schedoscope-tutorial/target/hadoop`. The Schedoscope test framework will automatically deploy a local Hadoop installation in that folder.

![test_run_configurations](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-tutorial/docs/pictures/test_run_configurations.png)

3. Rerun the test.

3. The tests can also be executed via Maven: `mvn test`. 

## Running on a real cluster
You can get the tutorial running on your own hadoop cluster.
 
Install the Schedoscope tutorial on a gateway machine to your cluster:

1. Clone the source code (see [[Installation|Open Street Map Tutorial#installing]] Step 4).

2. Prepare a `/hdp` folder in your cluster's HDFS and give proper write permissions for the user with which you want to execute Schedoscope. (similar to [[Installation|Open Street Map Tutorial#installing]] Step 3).

3. Then change the [[configuration settings|Configuring Schedoscope]] in `schedoscope-tutorial/src/main/resources/schedoscope.conf` to match the needs of your cluster. The [[default configuration settings|Configuring Schedoscope]] are defined in`schedoscope-core/src/main/resources/reference.conf`. They are overwritten by the settings you define in `schedoscope.conf`. Perform the following changes:

        schedoscope {
          app {
             # The chosen environment name is set as root HDFS folder.
             # The data for each view will end up in `/hdp/${env}/${package_name}/${ViewName}`.
             environment = "yourenvironmentasyoulikeit"    
          }

          hadoop {
             resourceManager = "yourhost:yourport"
             nameNode = "hdfs://yourhost:yourport"
          }

          metastore {
            metastoreUri = "thrift://your/hive/metastore/uri"
            jdbcUrl = "your/hive/jdbc/uri" # include the kerberos principal if needed
          }

          kerberos {
            principal = "your/kerberos/principal"   # if needed
          }
          
          transformations = {
            hive : {
                libDirectory = "/your/local/absolute/path/to/schedoscope/schedoscope-tutorial/target/hive-libraries"
                url = ${schedoscope.metastore.jdbcUrl} #include the kerberos principal if needed
            }
          }
        }


4. Compile Schedoscope (similar to [[Installation|Open Street Map Tutorial#installing]] Step 5).

Go into directory `schedoscope/schedoscope-tutorial` and [[execute|Open Street Map Tutorial#execution]] the tutorial using your own hadoop cluster:

    mvn exec:java



## Scheduling 

Schedoscope offers an [[HTTP API|Schedoscope HTTP API]]. If you want your data up-to-date 5 minutes past every hour during business time (8am-8pm) simply register a cronjob, such as:

       5   8-20 * * *  curl http://localhost:20698/materialize/schedoscope.example.osm.datamart/ShopProfiles

The registered GET request is the HTTP API equivalent to the `materialize -v schedoscope.example.osm.datamart/ShopProfiles` command you have issued on the shell before. Upon receiving this request, Schedoscope will check for new or changed direct or indirect dependencies to `ShopProfiles`, for structural changes, and changes of transformation logic. Only if any of these criteria apply, Schedoscope will trigger a (re-)computation. 

## Acknowledgement
Our example data was taken from the [Open Street Map project](http://www.openstreetmap.org/copyright)