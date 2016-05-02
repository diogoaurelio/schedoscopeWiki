Internally, Schedoscope uses Typesafe's [Akka framework](http://akka.io/). As a consequence, Schedoscope makes use of the [Typesafe Config Library](https://github.com/typesafehub/config) for configuration purposes. 

Hence, all configuration properties and their default values are defined in and taken from the file `reference.conf` on the classpath in the JSON-like HOCON format. 

Individual instances of Schedoscope can override these properties. This can happen by either
- putting a file `application.conf` onto the classpath or
- setting the system property `-Dconfig.file` pointing to a conf file when launching the JVM.
- for more information regarding Metascope check [Metascope Configuration](#)

# Reference configuration

Here, you find the commented default configuration of Schedoscope's `reference.conf`. 

At the very minimum, you should check and set if necessary:
- the environment name (`schedoscope.app.environment`);
- the earliest day to consider (`schedoscope.scheduler.earliestDay`);
- the Resource Manager, Namenode, and Metastore hosts (`schedoscope.hadoop.*`, `schedoscope.metastore.*`);
- the Kerberos Metastore principal (`schedoscope.kerberos.principal`);
- the Hive JDBC URL (`schedoscope.metastore.jdbcUrl`, `schedoscope.transformations.hive.url` )
- the number of concurrent driver actors for each transformation type (`schedoscope.transformations.*.concurrency`);
- the `future-driver-dispatcher` threadpool configuration depending on your concurrency settings.

For an example of how to override these settings, you can also take a look at the [tutorial](Open Street Map Tutorial).

    schedoscope {

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
              # ldap manager dn
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
              # Admin group(s): User will have admin rights
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

            url="{metascope.dir}/solr"
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

      }