# Open Street Map Tutorial

_**This Tutorial is under construction.**_

This Tutorial can be used without having your own hadoop cluster at hand. For testing the examples on your own cluster please read section [[Adaptation|Open Street Map Tutorial##Adaptation]].

## Installation
Let's get started:

1. Download [Cloudera Quickstart VM](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cloudera_quickstart_vm.html)
2. Start Cloudera VM.
3. Open a terminal and clone the Schedoscope git repository:

    `[cloudera@quickstart ~]$ git clone https://github.com/ottogroup/schedoscope.git`
4. Change directory to `schedoscope` and build the project:

    `[cloudera@quickstart schedoscope]$ mvn install`

## Execution

1. Change directory to `~/schedoscope/schedoscope-tutorial` and execute the tutorial:

    `[cloudera@quickstart schedoscope-tutorial]$ mvn exec:java`
1. The Schedoscope Shell opens in the terminal. Find the full [[command reference|Command Reference]] in the Schedoscope wiki.
1. Type  `materialize -v schedoscope.example.osm.datamart/ShopProfiles`

    and see how all data is digested which is needed in order to provide the requested view of shop profiles.

1. Type  `actions`
    and see which kind of transformations are running.

1. Type  `views`
    and see which views are already materialized with current data.
1. Have a look in the browser at the application manager of your Hadoop Cluster  <http://localhost:8088/cluster>
    and see the MR-Jobs running on the cluster.

## What's happening?


## Adaptation
Thus the example code is running now on the Cloudera Quickstart VM. 
Try get it running with your own hadoop cluster. Simply change the config settings in `schedoscope-tutorial/src/main/resources/schedoscope.conf` as follows:

**VM's schedoscope.conf:**

    schedoscope {
      app {
         environment = "demo"
      }

    transformations = {
      hive : {
              libDirectory = "/home/cloudera/schedoscope-tutorial/target/hive-libraries"
              concurrency = 2                # number of parallel actors to execute hive transformations
        }
      }
    }

**your new schedoscope.conf:**

    schedoscope {
      app {
         environment = "test"
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
		    libDirectory = "/home/cloudera/schedoscope-tutorial/target/hive-libraries"
		    url = ${schedoscope.metastore.jdbcUrl}
        }
      }
    }
The default settings are derived from `schedoscope-core/src/main/resources/reference.conf`. They are overwritten by the settings you define in your project's `schedoscope.conf`.

## Test-driven development

## Extension
Now it's time to design your own views and dependencies.


## Hints

