# Introduction

Schedoscope can be controlled with scheduling commands. There are two means of issuing these commands:

* using the Schedoscope shell command prompt opening when Schedoscope is not launched as a daemon;
* using the Schedoscope REST client based on the [REST API](Schedoscope REST API) 

## Schedoscope Shell

Schedoscope can be controlled by a command line shell. This shell is accessible when Schedoscope is launched as a REST service via the class `org.schedoscope.scheduler.api.SchedoscopeRestService` passing it the `--shell` option.

For example: 

    java -cp ${MY_SCHEDOSCOPE_CP} -Dlogback.configurationFile=${MY_SCHEDOSCOPE_FOLDER}/eci-logging.xml -Dconfig.file=${MY_SCHEDOSCOPE_FOLDER}/schedoscope.conf org.schedoscope.scheduler.api.SchedoscopeRestService --shell

## Schedoscope REST Client

There is also the Schedoscope REST client, which communicates with a running Schedoscope instance via the REST interface. The client can be started via the class `org.schedoscope.scheduler.api.SchedoscopeRestService` passing it the command to execute.

For example: 

    java -cp ${MY_SCHEDOSCOPE_CP} -Dlogback.configurationFile=${MY_SCHEDOSCOPE_FOLDER}/eci-logging.xml -Dconfig.file=${MY_SCHEDOSCOPE_FOLDER}/schedoscope.conf org.schedoscope.scheduler.api.SchedoscopeClientControl views -o

For this work, the configuration properties

* `schedoscope.webservice.host` and
* `schedoscope.webservice.port`

must point to the running Schedoscope REST service

# Commands

### views

`views` lists instantiated views, along with their status.

Supported options:
- `-s`, `--status`: filter views by their status (e.g. 'transforming')
- `-v`, `--viewPattern <viewPattern>`: select only views with URL paths matching a [view pattern](View Pattern Reference)  (e.g., `my.database/MyView/Partition1/Partition2`)
- `-f`, `--filterregular <regex>`: select only views with URL paths matching a regular expression (e.g., `my.database/.*/Partition1/.*`)
- `-d`, `--dependencies`: include dependencies in this view list
- `-o`, `--overview`: only show a count of views by status

Examples:

    views -o

    RESULTS
    =======
    materialized: 94788
    receive: 2
    nodata: 49736


### actions 
list status of currently action executors
- -s, --status filter actions by their status (e.g. 'queued, running, idle')
- -f, --filter regular expression to filter action display (e.g. '.*hive-1.*'). 

### queues 
list queued actions
- -t,--typ filter queued actions by their type (e.g. 'oozie', 'filesystem', ...)
- -f, --filter regular expression to filter queued actions (e.g. '.*my.dabatase/myView.*'). 

### commands 
list commands 
- -s, --status filter commands by their status (e.g. 'failed')
- -f, --filter regular expression to filter command display (e.g. '.*201501.*'). 

### materialize 
materialize view(s)
- -s, --status filter views to be materialized by their status (e.g. 'failed')
- -v, --viewUrlPath view url path (e.g. 'my.database/MyView/Partition1/Partition2'). 
- -f, --filter regular expression to filter views to be materialized (e.g. 'my.database/.*/Partition1/.*'). 
- -m, --mode materializatio mode. Supported modes are currently 'RESET_TRANSFORMATION_CHECKSUMS'"

### invalidate
invalidate view(s)")
- -s, --status filter views to be invalidated by their status (e.g. 'transforming')
- -v, --viewUrlPath  view url path (e.g. 'my.database/MyView/Partition1/Partition2'). 
- -f, --filter regular expression to filter views to be invalidated (e.g. 'my.database/.*/Partition1/.*'). 
- -d, --dependencies invalidate dependencies as well

### newdata 
notify schedoscope about newly arrived data
- -s, --status filter views to send 'newdata' to by their status (e.g. 'failed')
- -v, --viewUrlPath  view url path (e.g. 'my.database/MyView/Partition1/Partition2'). 
- -f, --filter regular expression to filter views to send 'newdata' to (e.g. 'my.database/.*/Partition1/.*'). 

### shutdown 
shutdown program
