Internally, Schedoscope uses Typesafe's [Akka framework](http://akka.io/). As a consequence, Schedoscope makes use of the [Typesafe Config Library](https://github.com/typesafehub/config) for configuration purposes.

Hence, all configuration properties and their default values are defined in and taken from the file `reference.conf` on the classpath in the JSON-like HOCON format.

Individual instances of Schedoscope can override these properties. This can happen by either
- putting a file `application.conf` onto the classpath or
- setting the system property `-Dconfig.file` pointing to a conf file when launching the JVM.
- for more information regarding Metascope check [Metascope Configuration](https://github.com/ottogroup/schedoscope/wiki/Configuring-Metascope)

# Reference configuration

Here, you find the commented default configuration of Schedoscope's core as stored in artifact `schedoscope-conf`'s `reference.conf` file.

At the very minimum, you should check and set if necessary:
- the environment name (`schedoscope.app.environment`);
- the earliest day to consider (`schedoscope.scheduler.earliestDay`);
- the Resource Manager, Namenode, and Metastore hosts (`schedoscope.hadoop.*`, `schedoscope.metastore.*`);
- the Kerberos Metastore principal (`schedoscope.kerberos.principal`);
- the Hive JDBC URL (`schedoscope.metastore.jdbcUrl` )
- the number of concurrent driver actors for each transformation type (`schedoscope.transformations.*.concurrency`);
- the `future-driver-dispatcher` threadpool configuration depending on your concurrency settings.

For an example of how to override these settings, you can also take a look at the [tutorial](Open Street Map Tutorial).

## `reference.conf` (`schedoscope-conf`)

    #
    # Schedoscope configuration properties and default values
    #

    schedoscope {

      #
      # Basic settings
      #

      app {

        #
        # The environment name of your Schedoscope instance. This allows you to run
        # multiple instances of schedoscope, such as for development, testing or
        # production but also for different applications / datawarehouses.
        #
        # The environment impacts the default database and partition path naming
        # schemes.
        #
        # Common environment names are: dev, int, prod
        #

        environment = "dev"

        #
        # View augmentor (a class extending
        # org.schedoscope.dsl.views.ViewUrlParser.ParsedViewAugmentor)
        # to use for dynamic view instantiation via the view pattern URL scheme.
        # This can be used to modify instantiated views in application-specific
        # ways outside of view pattern URLs. Should rarely need change.
        #

        parsedViewAugmentorClass = "org.schedoscope.dsl.views.NoAugmentation"
      }

      #
      # Settings with regard to scheduling logic
      #

      scheduler {

        #
        # The earliest date to consider for MonthlyParameterization or
        # DailyParameterization view traits. Should be the earliest day you have
        # data for your application to avoid any needless no-data overhead.
        #

        earliestDay = "2013-12-01"

        #
        # The latest date to consider for MonthlyParameterization or
        # DailyParameterization view traits. Should usually be "now", but can be
        # reduced to a fixed date, for example in testing environments.
        #

        latestDay = "now"


        #
        # Various timeouts to obey
        #

        timeouts {

          #
          # Timeout waiting for metastore operations
          #

          schema = 1 hour


          #
          # Timeout waiting for scheduling command to return
          #

          schedulingCommand = 1 hour
        }

       #
       # Registered listeners for view scheduling state changes
       #
       
       viewSchedulingListeners = ["org.schedoscope.scheduler.listeners.ViewSchedulingMonitor"]

      }

      #
      # Configuration properties with regard Schedoscope's REST web service API.
      # Used for both Schedoscope server and the Schedoscope client tool.
      #

      webservice {


        #
        # Host where Schedoscope instance is running
        #

        host = "localhost"

        #
        # Port on which the Schedoscope web service listens for connections
        #

        port = 20698

        #
        # Directory for storing static web service resources
        #

        resourceDirectory = "www"

        #
        # Number of webservice actors to answer requests.
        #

        concurrency = 5
      }


      #
      # Kerberos-related settings
      #

      kerberos {

        #
        # The kerberos principal required for access to the Metastore
        # E.g.: hive/somemachine@SOMEDOMAIN.COM
        #
        principal = ""
      }

      #
      # Settings related to core Hadoop
      #

      hadoop {

        #
        # Address and port of the YARN resource manager. Note that any locally configured Hadoop environment
        # takes precedence.
        #

        resourceManager = "localhost:8032"

        #
        # Address and port of the HDFS namenode. Note that any locally configured Hadoop environment
        # takes precedence.
        #

        nameNode = "localhost:8020"

        #
        # HDFS URI
        #

        hdfs = "hdfs://localhost:8020"

        #
        # Schedoscope stores view data by default below this root folder.
        #

        viewDataHdfsRoot = "/hdp"
      }


      #
      # Settings related to the Hive metastore
      #

      metastore {

        #
        # Thrift URI to metastore service
        #
        metastoreUri = "thrift://localhost:9083"

        #
        # Hive JDBC URL to connect to Hive Server 2
        #

        jdbcUrl = "jdbc:hive2://localhost:10000/default"

        #
        # Path to hive config files on your cluster
        #

        hiveConfDir = "/etc/hive/conf/"

        #
        # Number of parallel connections to use for schema- and partition creation
        # via the Hive metastore
        #

        concurrency = 5

        #
        # Number of partitions to put into a batch for each connection when
        # creating partitions
        #

        writeBatchSize = 2500

        #
        # Number of partitions to read in batch with each request on each
        # connection
        #

        readBatchSize = 10000
      }

      #
      # Settings concerning automatic logic change detection
      #

      versioning {

        #
        # Compute transformation version checksums and detect need for
        # rematerialization based on those checksums. Should be kept as is.
        #

        transformations = true
      }

      #
      # Number of retries for transformations before view state is changed to
      # failed.
      #

      action.retry = 5

      #
      # Setting concerning the usage of external dependencies
      #

      external-dependencies {

        #
        # This setting allows you to use external dependencies and operate several schedoscope instances in conjunction.
        #

        enabled = false

        #
        # A list of prefixes of packages with internal views. Every package not starting with a string in this list
        # will be treated as external and can not be referenced from the client or used as dependency if not flagged as
        # exernal
        #

        home = ["${env}.test.datahub"]

        #
        # Toggles checks whether internal views are used as external views and vice versa
        #

        checks = true
      }

      #
      # Default settings for exportTo() exporters.
      #

      export {

        #
        # Export settings for all export types
        #

        salt = "vD75MqvaasIlCf7H"

        #
        # JDBC exporter settings.
        #

        jdbc {

          #
          # Number of reducers to use for parallel writing to JDBC.
          #

          numberOfReducers = 10

          #
          # Batch size for insert statements to use
          #

          insertBatchSize = 10000

          #
          # Storage engine to use (in case of MySQL JDBC targets)
          #

          storageEngine = "InnoDB"

        }

        #
        # Redis exporter.
        #

        redis {

          #
          # Number of reducers to use for parallel writing to Redis.
          #

          numberOfReducers = 10

          #
          # Usage of pipeline mode for writing to Redis.
          #

          usePipelineMode = false

          #
          # Batch size for Redis pipeline mode
          #

          insertBatchSize = 10000

        }

        #
        # Kafka exporter
        #

        kafka {

          #
          # Number of reducer to use for parallel writing to Kafka.
          #

          numberOfReducers = 10

        }

        #
        # Ftp exporter
        #

        ftp {

          #
          # Number of reducers to use for parallel exporting to a (S)Ftp Server.
          #

          numberOfReducers = 2

        }
      }

      #
      # Metascope related settings
      #

      metascope {

        #
        # Metascope webservice port
        #

        port = 8080

        #
        # Configure authentication method
        #

        auth {

          #
          # Possible methods = ["simple", "ldap"]
          # simple: custom user management
          # ldap: external ldap directory is used for authentication
          #

          authentication = "simple"

          #
          # ldap settings
          #

          ldap {

            #
            # ldap server url
            #

            url = ""

            #
            #	ldap manager dn
            #

            managerDn = ""

            #
            # ldap manager password
            #

            managerPassword = ""

            #
            # ldap user dn pattern
            #

            userDnPattern = ""

            #
            # ldap group search base
            #

            groupSearchBase = ""

            #
            # Allowed groups to access Metascope
            #

            allowedGroups = ""

            #
            #	Admin group(s): User will have admin rights
            #

            adminGroups = ""

          }

        }

        #
        # Database which is used as the metadata repository.
        # Default: embedded Apache Derby instance, which creates a local repository in the metascope directory ('.../metascope/repository')
        #

        repository {

          #
          # JDBC URL to database server
          #

          url = "jdbc:derby:{metascope.dir}/repository;create=true"

          #
          # database user
          #

          user = "sa"

          #
          # users password
          #

          password = ""

          #
          # Dialect to use to talk to database. For further information, visit:
          # http://docs.jboss.org/hibernate/orm/3.5/javadocs/org/hibernate/dialect/package-summary.html
          #

          dialect = "org.hibernate.dialect.DerbyTenSevenDialect"
        }

        #
        # Settigns relataed to the Apache Solr instance
        #

        solr {

          #
          # URL to Solr instance.
          # Default: embedded Apache Solr instance, which creates a local index in the metascope directory ('.../metascope/solr')
          #

          url = "{metascope.dir}/solr"
        }

        #
        # General settings concerning logging (via SLF4J)
        #

        logging {

          #
          # Path to logfile
          #

          logfile = "{metascope.dir}/logs/metascope-service.log"

          #
          # Log level granularity
          #

          loglevel = "DEBUG"
        }

      }

      #
      # Driver settings for the different transformation types.
      #

      transformations = {

        noop: {
          #
          # Class implementing the NoOp driver
          #
          driverClassName = "org.schedoscope.scheduler.driver.NoOpDriver"

          #
          # ignored
          #
          location = "/tmp/schedoscope/noop/"

          #
          # concurrency for NoOp transformations
          #
          concurrency = 5

          #
          # Timeout for NoOp transformations.
          #

          timeout = 1 day

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]

        }

        #
        # Settings for Hive transformations
        #

        hive: {

          #
          # Class implementing the Hive driver
          #

          driverClassName = "org.schedoscope.scheduler.driver.HiveDriver"

          #
          # Location where to put Hive UDFS in HDFS
          #

          location = "/tmp/schedoscope/hive/"

          #
          # Comma-separated list of directories where additional UDF library
          # jars can be found that are to be put onto HDFS when launching
          # Schedoscope.
          #

          libDirectory = ""

          #
          # Ignored
          #

          url = ""

          #
          # Do not change. Hive UDF jars should not be unpacked in HDFS.
          #

          unpack = false

          #
          # Number of parallel Driver actors to use for executing Hive
          # transformations
          #

          concurrency = 10

          #
          # Timeout for Hive transformations.
          #

          timeout = 1 day

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]

        },

        #
        # Settings for MapReduce transformations
        #

        mapreduce: {

          #
          # Class implementing the Mapreduce driver
          #

          driverClassName = "org.schedoscope.scheduler.driver.MapreduceDriver"

          #
          # Ignored.
          #

          location = "/tmp/schedoscope/mapreduce/"


          #
          # Ignored.
          #

          libDirectory = ""

          #
          # Ignored.
          #

          url = ""

          #
          # Ignored
          #

          unpack = false

          #
          # Number of parallel Driver actors to use for executing MapReduce
          # transformations
          #

          concurrency = 10

          #
          # Timeout for MapReduce transformations
          #

          timeout = 1 day

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]

        },

        #
        # File system driver settings
        #

        filesystem: {

          #
          # Class implementing the FileSystem driver
          #

          driverClassName = "org.schedoscope.scheduler.driver.FilesystemDriver"

          #
          # Number of parallel Driver actors to use for executing file system
          # transformations
          #

          concurrency = 1

          #
          # Ignored.
          #

          location = "/tmp/schedoscope/filesystem/"

          #
          # Ignored
          #

          libDirectory = ""

          #
          # Ignored
          #

          url = ""

          #
          # Ignored
          #

          unpack = false

          #
          # Timeout for file system transformations
          #

          timeout = 1 hour

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]

        },

        #
        # Sequence driver settings
        #

        seq: {

          #
          # Class implementing the Seq driver
          #

          driverClassName = "org.schedoscope.scheduler.driver.SeqDriver"

          #
          # Number of parallel Driver actors to use for executing seq
          # transformations
          #

          concurrency = 10

          #
          # Ignored.
          #

          location = "/tmp/schedoscope/seq/"

          #
          # Ignored
          #

          libDirectory = ""

          #
          # Ignored
          #

          url = ""

          #
          # Ignored
          #

          unpack = false

          #
          # Timeout for seq transformations
          #

          timeout = 1 day

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]

        }
      }
    }

    #
    # This section covers general Akka settings, in particular concerning the
    # various dispatchers / threadpools.
    #

    akka {

      #
      # General settings concerning logging (via SLF4J). Logback is given as a
      # default dependency to Schedoscope so SLF4J via Logback is the default
      # logging system.
      #

      log-config-on-start = off
      log-dead-letters = off
      loglevel = "INFO"
      loggers = ["akka.event.slf4j.Slf4jLogger"]

      #
      # Dispatcher settings
      #

      actor {

        guardian-supervisor-strategy = "org.schedoscope.TerminatingStoppingStrategy"

        #
        # Given the variety of component-specific dispatchers used by Schedoscope
        # we limit the Akka default dispatcher to 4 threads.
        #

        default-dispatcher {
          executor = "thread-pool-executor"
          type = Dispatcher

          thread-pool-executor {
            core-pool-size-min = 4
            core-pool-size-factor = 1.0
            core-pool-size-max = 4
            task-queue-size = -1
          }

          throughput = 5
        }


        #
        # Configuration of threads to use for unspecific future calls and
        # computations.
        # Again limited.
        #

        future-call-dispatcher {
          type = Dispatcher
          executor = "thread-pool-executor"

          thread-pool-executor {
            core-pool-size-min = 8
            core-pool-size-factor = 2.0
            core-pool-size-max = 8
            task-queue-size = -1
          }

          throughput = 5
        }


        #
        # The supervisor / message router for the Metastore-related actors is
        # assigned to one pinned dispatcher / thread. Should not need to be
        # changed.
        #

        schema-manager-dispatcher {
          executor = "thread-pool-executor"
          type = PinnedDispatcher
        }

        #
        # Threadpool / dispatcher configuration for the actor communicating
        # with the Metastore for creating / reading tables and partitions. Should
        # be aligned with the property schedoscope.metastore.concurrency
        #

        partition-creator-dispatcher {
          executor = "thread-pool-executor"
          type = Dispatcher

          thread-pool-executor {
            core-pool-size-min = 5
            core-pool-size-factor = 1.0
            core-pool-size-max = 5
            task-queue-size = -1
          }

          throughput = 5
        }


        #
        # The actor logging transformation metadata (version, timestamp) to the
        # Metastore has one dedicated pinned dispatcher / thread.
        #

        metadata-logger-dispatcher {
          executor = "thread-pool-executor"
          type = PinnedDispatcher
        }

        #
        # The actor managing, assigning, and collecting status information about
        # transformations to execute has one dedicated pinned dispatcher / thread
        # for reasons of responsiveness.
        #

        transformation-manager-dispatcher {
          executor = "thread-pool-executor"
          type = PinnedDispatcher
        }

        #
        # The actor managing and instantiating view actors for scheduling state
        # management has one dedicated pinned dispatcher / thread for reasons of
        # responsiveness.
        #

        view-manager-dispatcher {
          executor = "thread-pool-executor"
          type = PinnedDispatcher
        }

        #
        # The threadpool / dispatcher used by the view actors for communicating
        # with each other for view materialization and scheduling.
        #

        views-dispatcher {
          executor = "fork-join-executor"
          type = Dispatcher

          fork-join-executor {
            parallelism-min = 8
            parallelism-factor = 2.0
            parallelism-max = 8
            task-queue-size = -1
          }

          throughput = 5
        }

        #
        # The threadpool / dispatcher available for the transformation drivers.
        # As drivers execute transformations asynchronously - either by
        # communicating with servers like Oozie or the Resource Manager or by
        # futures, this pool should not need to grow much when increasing
        # concurrency settings of the various transformations.
        #

        driver-dispatcher {
          executor = "thread-pool-executor"
          type = Dispatcher

          thread-pool-executor {
            core-pool-size-min = 8
            core-pool-size-factor = 2.0
            core-pool-size-max = 8
            task-queue-size = -1
          }

          throughput = 5
        }

        #
        # Due to API limitations, we can currently only execute file system, Pig,
        # and Hive transformations asynchronously by employing futures.
        #
        # The following threadpool / dispatcher feeds those futures. Hence, it
        # should grow proportionally with the concurrency settings of the
        # mentioned transformation types.
        #

        future-driver-dispatcher {
          type = Dispatcher
          executor = "thread-pool-executor"

          thread-pool-executor {
            core-pool-size-min = 16
            core-pool-size-factor = 4.0
            core-pool-size-max = 16
            task-queue-size = -1
          }

          throughput = 5
        }
      }
    }

    #
    # The following are a list of "suggestions" for spray settings to use for the
    # Schedoscope web service and client. They are not automatically pulled
    # but have to be explicitly set in your application.conf / schedoscope.conf.
    #

    spray.can {
      client {
        parsing {
          max-content-length = 100m
        }

        # The time after which an idle connection will be automatically closed.
        # Set to `infinite` to completely disable idle timeouts.
        idle-timeout = 1200s

        #  The max time period that a client connection will be waiting for a response
        #  before triggering a request timeout. The timer for this logic is not started
        #  until the connection is actually in a state to receive the response, which
        #  may be quite some time after the request has been received from the
        #  application!
        #  There are two main reasons to delay the start of the request timeout timer:
        #    1. On the host-level API with pipelining disabled:
        #  If the request cannot be sent immediately because all connections are
        #  currently busy with earlier requests it has to be queued until a
        #  connection becomes available.
        #  2. With pipelining enabled:
        #    The request timeout timer starts only once the response for the
        #  preceding request on the connection has arrived.
        #  Set to `infinite` to completely disable request timeouts.
        request-timeout = 900s

        # The time period within which the TCP connecting process must be completed.
        # Set to `infinite` to disable.
        connecting-timeout = 10s

      }
      server {
        # If a request hasn't been responded to after the time period set here
        # a `spray.http.Timedout` message will be sent to the timeout handler.
        # Set to `infinite` to completely disable request timeouts.
        request-timeout = 900s

        # After a `Timedout` message has been sent to the timeout handler and the
        # request still hasn't been completed after the time period set here
        # the server will complete the request itself with an error response.
        # Set to `infinite` to disable timeout timeouts.
        timeout-timeout = 6s

        # The time after which an idle connection will be automatically closed.
        # Set to `infinite` to completely disable idle connection timeouts.
        idle-timeout = 1200s

        # Enables/disables the returning of more detailed error messages to
        # the client in the error response.
        # Should be disabled for browser-facing APIs due to the risk of XSS attacks
        # and (probably) enabled for internal or non-browser APIs.
        # Note that spray will always produce log messages containing the full
        # error details.
        verbose-error-messages = off

        # Enables/disables the logging of the full (potentially multiple line)
        # error message to the server logs.
        # If disabled only a single line will be logged.
        verbose-error-logging = off

        # Enables/disables support for statistics collection and querying.
        # Even though stats keeping overhead is small,
        # for maximum performance switch off when not needed.
        stats-support = on

        # Enables/disables automatic back-pressure handling by write buffering and
        # receive throttling
        automatic-back-pressure-handling = on
        back-pressure {
          # The reciprocal rate of requested Acks per NoAcks. E.g. the default value
          # '10' means that every 10th write request is acknowledged. This affects the
          # number of writes each connection has to buffer even in absence of back-pressure.
          noack-rate = 10
          # The lower limit the write queue size has to shrink to before reads are resumed.
          # Use 'infinite' to disable the low-watermark so that reading is resumed instantly
          # after the next successful write.
          reading-low-watermark = infinite
        }
      }
    }


## `reference.conf` (`schedoscope-transformation-oozie`)

If you use Oozie transformations, you will likely want to set up concurrency and the Oozie server URL. Here is the `reference.conf` with the default configs for Oozie:

    #
    # Oozie transformation settings
    #

    schedoscope {

      transformations = {

        oozie: {

          #
          # Class implementing the Oozie driver
          #

          driverClassName = "org.schedoscope.scheduler.driver.OozieDriver"

          #
          # Where to put Oozie bundles in HDFS
          #

          location = "/tmp/schedoscope/oozie/"

          #
          # Comma-separated list of directories where additional Oozie workflow
          # oozie.bundle jars can be found that are to be put onto HDFS when launching
          # Schedoscope.
          #

          libDirectory = ""

          #
          # URL of Oozie Server
          #

          url = "http://localhost:11000/oozie"

          #
          # Number of parallel Driver actors to use for executing Oozie
          # transformations
          #

          concurrency = 10

          #
          # Oozie oozie.bundle jars need to be unpacked in HDFS.
          #

          unpack = true

          #
          # Timeout for Oozie transformations
          #

          timeout = 1 day

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]

        }

      }
    }

## `reference.conf` (`schedoscope-transformation-pig`)

If you use Pig transformations, you will likely want to set up concurrency for this transformation type. Here is the `reference.conf` with the default configs for Pig:

    #
    # Pig transformation settings
    #

    schedoscope {

      transformations = {

        pig: {

          #
          # Class implementing the Pig driver
          #

          driverClassName = "org.schedoscope.scheduler.driver.PigDriver"

          #
          # Location where to put Pig library jar in HDFS
          #

          location = "/tmp/schedoscope/pig/"

          #
          # Ignored
          #

          libDirectory = ""

          #
          # Ignored.
          #

          url = ""

          #
          # Do not change. Pig jars should not be unpacked in HDFS.
          #

          unpack = false

          #
          # Number of parallel Driver actors to use for executing Pig
          # transformations
          #

          concurrency = 10

          #
          # Timeout for Pig transformations.
          #

          timeout = 1 day

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]
        }
      }
    }

## `reference.conf` (`schedoscope-transformation-shell`)

If you use shell transformations, you will likely want to set up concurrency for this transformation type. Here is the `reference.conf` with the default configs for shell:

    #
    # Shell transformation settings
    #

    schedoscope {

      transformations = {

        shell: {

          #
          # Class implementing the Shell driver
          #

          driverClassName = "org.schedoscope.scheduler.driver.ShellDriver"

          #
          # Number of parallel Shell Driver actors to use
          #

          concurrency = 1

          #
          # Ignored
          #

          location = "/tmp/schedoscope/shell/"

          #
          # Ignored
          #

          libDirectory = ""

          #
          # Ignored
          #

          url = ""

          #
          # Ignored
          #

          unpack = false

          #
          # Timeout for Shell transformations
          #

          timeout = 1 day

          #
          # The handlers being notified after each driver run has
          # finished (succeeded or failed). These must implement the
          # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
          #

          driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]
        }
      }
    }

## `reference.conf` (`schedoscope-transformation-spark`)

If you use spark transformations, you will likely want to set up concurrency for this transformation type. Here is the `reference.conf` with the default configs for spark:


   spark: {

      #
      # Class implementing the Oozie driver
      #

      driverClassName = "org.schedoscope.scheduler.driver.SparkDriver"

      #
      # Ignored
      #

      location = "/tmp/schedoscope/spark/"

      #
      # Paths to add to the spark.executor.extraClassPath and spark.driver.extraClassPath properties
      #

      libDirectory = "/etc/hive/conf:/etc/hadoop/conf"

      #
      # Ignored
      #

      url = ""

      #
      # Number of parallel Driver actors to use for executing Spark transformations
      #

      concurrency = 10

      #
      # Ignored
      #

      unpack = false

      #
      # Timeout for Spark transformations
      #

      timeout = 1 day

      #
      # The handlers being notified after each driver run has
      # finished (succeeded or failed). These must implement the
      # trait org.schedoscope.scheduler.driver.DriverRunCompletionHandler
      #

      driverRunCompletionHandlers = ["org.schedoscope.scheduler.driver.DoNothingCompletionHandler"]

    }
