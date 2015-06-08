## Introduction

Schedoscope offers a rest api for remote control. The schedoscopeControl shell makes use of this API, but it may be also used to trigger new materializations or notify the scheduler upon the arrival of new data.

## REST Endpoints

### views



### actions 

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


### newdata 

### shutdown 

