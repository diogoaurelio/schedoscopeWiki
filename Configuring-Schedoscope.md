# Configuration
Schedoscope is configured using a HOCON (JSON-like) configuration file that is shared with akka.

# Reference configuration

	schedoscope {
	
		# global settings for schedoscope
	    app {
	        # the environment allows you to run multiple instances of schedoscope for
	        # development, testing or production. The environment can be referenced within
	        # the configurtaion
	        environment = "dev"                                                         # environment we are in
	        # schedoscope can infer proper default values for view parameters using a
	        # custom view augmenter implementation
	        parsedViewAugmentorClass = "org.schedoscope.dsl.views.NoAugmentation"       # view augmentor (class extending com.ottogroup.bi.soda.dsl.views.ViewUrlParser.ParsedViewAugmentor) to use for dynamic view instantation in web service.
	    }
	
	    scheduler {
	        # the begin of your data collection
	        earliestDay = "2013-12-01"                                                  # earliest date to consider for date parameterization arithmetics
	        latestDay   = "now"                                                         # latest date to consider for date parameterization arithmetics (now or a date)
	        # 
	        timeouts {
	            schema = 1 hour                     # timeout waiting for schema operations
	            statusListAggregation = 60 seconds  # timeout waiting for status list compilations
	            viewManagerResponse = 60 seconds    # timeout waiting for viewManager
	            # change "completion" if you have really longrunning jobs
	            completion = 3 days                 # timeout waiting for completion of view jobs
	        }
	    }
	    # configuration of schedoscope's REST Service
	    webservice {
	        timeout = 10 minutes                # timeout for web service calls
	        host = "localhost"					# host of the web service
	        port = 20698                        # port of the web service
	        resourceDirectory = "www"           # location of web service resources (HTML, javascript, ...)
	        concurrency = 10                    # number of webservice actors to answer requests
	    }
	
	    kerberos {
	        principal = ""                      # hive kerberos principal, defaults to ""
	    }
	
	    hadoop {
	        resourceManager = "localhost:8032"  # Address and port of the YARN resource manager
	        nameNode = "localhost:8020"         # Address and port of the YARN resource manager       
	    }
	
	    metastore {
	        metastoreUri = "thrift://localhost:9083"                # Thrift URI to metastore service
	        jdbcUrl = "jdbc:hive2://localhost:10000/default"        # Hive JDBC URL
	        concurrency = 5                                         # number of parallel connections to metastore service
	        writeBatchSize = 2500                                   # size of write batches sent to metastore
	        readBatchSize = 10000                                   # size of read batches from metastore
	    }
	        
	    versioning {
	        transformations = true              # transformation versions contribute to the version of a view instance  
	    }
	    
	    action.retry = 5               # number of times an failed transformation is executed until the view is set to state failed
	
	    
	    transformations = {
	        hive : {
	            location = "/tmp/schedoscope/hive/"    # location where to put hive UDFs in HDFS
	            libDirectory = ""               # comma-separated directories with additional libs to put into location
	            url = "jdbc:hive2://localhost:10000/default" # Hive JDBC URL
	            unpack = false                  # need hive jars be unpacked? No!
	            concurrency = 10                # number of parallel actors to execute hive transformations
	            timeout = 1 day                 # timeout for hive transformations
	        },
	        pig : {
	            location = "/tmp/soda/pig/"     # location where to put library jars in HDFS
	            libDirectory = ""               # comma-separated directories with additional libs to put into location
	            url = ""                        # ignore
	            unpack = false                  # need pig jars be unpacked? No!
	            concurrency = 10                # number of parallel actors to execute pig transformations
	            timeout = 1 day                 # timeout for pig transformations
	        }, 
	        mapreduce : {
	            location = "/tmp/schedoscope/mapreduce/" # location where to put library jars in HDFS
	            libDirectory = ""               # comma-separated directories with additional libs to put into location
	            url = ""                        # FIXME: jobtracker url?
	            unpack = false                  # need mapreduce jars be unpacked? No!
	            concurrency = 10                # number of parallel actors to execute mapreduce transformations
	            timeout = 1 day                 # timeout for mapreduce transformations
	        },                 
	        oozie : {
	            location = "/tmp/schedoscope/oozie/"   # location where to put oozie bundles in HDFS
	            libDirectory = ""               # comma-separated directories with additional libs to put into location
	            url = "http://localhost:11000/oozie"    # URL to Oozie server
	            concurrency = 10                # number of parallel actors to execute oozie transformations
	            unpack = true                   # need hive oozie bundles be unpacked? Yes!
	            timeout = 1 day                 # timeout for oozie transformations
	        },    
	        filesystem : {
	            concurrency = 1                 # number of parallel actors to execute file systems transformations
	            timeout = 1 hour                # timeout for filesystem transformations
	        }   ,    
	        morphline : {
	            concurrency = 1                 # number of parallel actors to execute file systems transformations
	            timeout = 1 day                 # timeout for morphline transformations
	        }  
	    }
	}
	
	akka {
	    log-config-on-start = off
	    log-dead-letters = off
	    loglevel = "INFO"
	    loggers = ["akka.event.slf4j.Slf4jLogger"]
	    
	    actor {        
	        default-dispatcher {
	            executor = "thread-pool-executor"
	            type = Dispatcher
	                                 
	            thread-pool-executor {
	                core-pool-size-min = 8
	                core-pool-size-factor = 1.0
	                core-pool-size-max = 8
	                task-queue-size = -1
	            }
	
	            throughput = 5
	        }
	
	
	        future-call-dispatcher {
	            type = Dispatcher
	            executor = "thread-pool-executor"                
	             
	            thread-pool-executor {
	                core-pool-size-min = 8
	                core-pool-size-factor = 1.0
	                core-pool-size-max = 8
	                task-queue-size = -1
	            }
	
	            throughput = 5
	        }
	
	        root-actor-dispatcher {
	            executor = "thread-pool-executor"
	            type = PinnedDispatcher         
	        }   
	
	        schema-root-actor-dispatcher {
	            executor = "thread-pool-executor"
	            type = PinnedDispatcher         
	        }           
	
	        actions-manager-dispatcher {
	            executor = "thread-pool-executor"
	            type = PinnedDispatcher         
	        }
	
	        view-manager-dispatcher {
	            executor = "thread-pool-executor"
	            type = PinnedDispatcher         
	        }
	
	        metadata-logger-dispatcher {
	            executor = "thread-pool-executor"
	            type = PinnedDispatcher         
	        }
	
	        schema-actor-dispatcher {
	            executor = "thread-pool-executor"
	            type = Dispatcher
	                             
	            thread-pool-executor {
	                core-pool-size-min = 8
	                core-pool-size-factor = 1.0
	                core-pool-size-max = 8
	                task-queue-size = -1
	            }
	
	            throughput = 5
	        }
	
	        views-dispatcher {
	            executor = "thread-pool-executor"
	            type = Dispatcher
	                                 
	            thread-pool-executor {
	                core-pool-size-min = 8
	                core-pool-size-factor = 4.0
	                core-pool-size-max = 96
	                task-queue-size = -1
	            }
	
	            throughput = 5
	        }               
	
	        driver-dispatcher {
	            executor = "thread-pool-executor"
	            type = Dispatcher
	                             
	            thread-pool-executor {
	                core-pool-size-min = 8
	                core-pool-size-factor = 1.0
	                core-pool-size-max = 8
	                task-queue-size = -1
	            }
	
	            throughput = 5
	        }           
	
	        future-driver-dispatcher {
	            type = Dispatcher
	            executor = "thread-pool-executor"  
	
	            thread-pool-executor {
	                core-pool-size-min = 16
	                core-pool-size-factor = 2.0
	                core-pool-size-max = 16
	                task-queue-size = -1
	            }
	
	            throughput = 5
	        }
	    }
	}
	
	spray.can.client.parsing.max-content-length=100000000
	spray.can.client.request-timeout = infinite
	spray.can.client.connecting-timeout = infinite
	spray.can.client.idle-timeout = infinite
	spray.can.host-connector.idle-timeout = infinite
	spray.can.server.request-timeout = infinite
	spray.can.server.idle-timeout = infinite
	
