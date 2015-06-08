## Schedoscope Shell
Schedoscope can be controlled by a command line shell. This shell is accessible when schedoscope is not started as a daemon or by using the schedoscopeControl.sh script.

### views
lists all view actors, along with their status"
- -s, --status filter views by their status (e.g. 'transforming')"
- -v, --viewUrlPathview url path (e.g. 'my.database/MyView/Partition1/Partition2'). 
- -f, --filterregular expression to filter view display (e.g. 'my.database/.*/Partition1/.*'). "
- -d, --dependencies include dependencies
- -o, --overview show only overview, skip individual views


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