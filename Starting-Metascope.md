Metascope operates as a REST web service. There are two ways of launching this service.

# Launching via start-metascope.sh script

The simplest way of launching the Metascope web service is by executing `org.schedoscope.metascope.Metascope`'s main method. 

The Schedoscope web service needs a Schedoscope configuration file. This can be passed using the system properties `-Dconfig.file`. Check the `start-metascope.sh` script.


Example:

    java -Xmx1024m -XX:MaxPermSize=512M -cp ${CP} -Dconfig.file=/path/to/schedoscope.conf org.schedoscope.metascope.Metascope 


# Launching as a Daemon

The Schedoscope web service can also be started as a daemon using [jsvc](http://commons.apache.org/proper/commons-daemon/jsvc.html) and [Apache Commons Daemon](http://commons.apache.org/proper/commons-daemon/).

Example:

    jsvc -Xmx1024m -XX:MaxPermSize=512M -cp ${CP} -Dconfig.file=/path/to/schedoscope.conf  org.schedoscope.metascope.MetascopeDaemon
