Metascope shares the same config file as Schedoscope. All possible properties can be found in the according section `schedoscope.metascope`. For more information regarding the settings and the HCON format, see [Schedoscope Configuration](https://github.com/ottogroup/schedoscope/wiki/Configuring Schedoscope).

# Reference configuration

Here, you find the commented default configuration of Metascope in Schedoscope's `reference.conf`. 

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

# Repository configuration

Metascope uses a relational database as the main metadata repository and an [Apache Solr](http://lucene.apache.org/solr/) instance as an index for searching and filtering purposes. If Metascope is run in embedded mode (which is the default), it will create an embedded instance of [Apache Derby](https://db.apache.org/derby/) and a embedded Solr core in the metascope target folder:

* `.../schedoscope-metascope/target/repository/`: Local metadata repository (Apache Derby)
* `.../schedoscope-metascope/target/solr/`: Local index (Apache Solr)

In production, especially when you have a huge amount of tables and partitions, it is recommended to use a standalone database and Solr server.

***

**External (Standalone) Metadata Repository**

Metascope uses Hibernate to communicate with the relational database, which means that most of the common relational databases should suit Metascope (check https://developer.jboss.org/wiki/SupportedDatabases2) 

Note: At the moment, only MySQL has been tested. Feel free to use other databases like Oracle or PostgreSQL.

Here is an example for a MySQL configuration:

    repository {
        url = "jdbc:mysql://localhost/anyDatabase?createDatabaseIfNotExist=true&rewriteBatchedStatements=true"
        user = "username"
        password = "secret"
        dialect = "org.hibernate.dialect.MySQL5Dialect"
    }

***

**External (Standalone) Metadata Index**

To use a standalone Solr server, set the `metascope.solr.url` in the `application.conf` file to `http://solrhost:solrport`. The Solr instance needs a core (the actual index) named `metascope` in which it persists the indexed data. At the moment, it is not possible to create a Solr core on the fly.
Create a core named `metascope` with the appropriate [solrconfig.xml](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-metascope/src/main/resources/solr/metascope/conf/solrconfig.xml) and [schema.xml](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-metascope/src/main/resources/solr/metascope/conf/schema.xml) files. For further information, check
[How do I create a Solr Core](https://www.codeenigma.com/host/faq/how-do-i-create-solr-core-my-server#solr4)