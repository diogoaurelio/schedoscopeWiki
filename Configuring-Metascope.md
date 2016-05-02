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