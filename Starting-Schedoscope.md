# Introduction

Schedoscope operates as a REST web service. There are three ways of launching this service

# Launching without Command Shell

The simplest way of launching the Schedoscope web service is by executing the `org.schedoscope.scheduler.api.SchedoscopeRestService` main method. After launch, the service expects [scheduling commands](Command Reference) via the [REST API](Schedoscope REST API).  The [Schedoscope REST Client](Command Reference) can be used to issue these.

In order to execute, the Schedoscope web service needs a logback configuration file and a Schedoscope configuration file. These can be passed using the environment properties `-Dlogback.configurationFile` and `-Dconfig.file`.


Example:

    java -cp ${CP} -Dlogback.configurationFile=logback.xml -Dconfig.file=schedoscope.conf org.schedoscope.scheduler.api.SchedoscopeRestService 

# Launching with Command Shell

The Schedoscope web service can also be launched in such a way that it opens up a command shell on `stdout` in addition to exposing the HTTP API. This shell allows for the comfortable issuing of [scheduling commands](Command Reference) and features a command history.

Opening the shell triggered by passing the `--shell` parameter.

Example:

    java -cp ${CP} -Dlogback.configurationFile=logback.xml -Dconfig.file=schedoscope.conf org.schedoscope.scheduler.api.SchedoscopeRestService --shell

# Launching as a Daemon

The Schedoscope web service can also be started as a daemon using [jsvc](http://commons.apache.org/proper/commons-daemon/jsvc.html) and [Apache Commons Daemon](http://commons.apache.org/proper/commons-daemon/). 


