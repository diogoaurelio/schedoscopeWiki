# Introduction

Schedoscope operates as a REST web service. There are three ways of launching this service

# Launching without Command Shell

The simplest way of launching the Schedoscope web service is by executing the `org.schedoscope.scheduler.api.SchedoscopeRestService` main method. After launch, the service expects scheduling commands [via the REST API](Schedoscope REST API).  The [Schedoscope REST Client](Command Reference) can be used to issue these.

In order to execute, the Schedoscope web service needs a logback configuration file and a Schedoscope configuration file. These can be passed using the environment properties `${logback.configurationFile}` and `${config.file}`.


Example:

    java -cp ${CP} -Dlogback.configurationFile=logback.xml -Dconfig.file=schedoscope.conf org.schedoscope.scheduler.api.SchedoscopeRestService 