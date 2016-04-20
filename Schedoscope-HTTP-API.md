Schedoscope can be remotely controlled via a simple HTTP API. It can be used for triggering new materializations or notify the scheduler upon the arrival of new data, but also for checking view scheduling states remotely.

## API Request Types
Currently, all remote requests to Schedoscope use the HTTP GET method. Parameters are passed as URL parameters and need to be URL-encoded appropriately. Results are returned as JSON documents. In case of errors, HTTP error codes <> 200 are set.

The currently implemented request types are as follows:

#### views
List all currently active views  

Method: GET  
Path: /views/  
or  
Path: /views/`ViewPattern`  

if a `ViewPattern` is given, only information about views matching the [pattern](View-Pattern-Reference) is returned

**Parameters:**  

- status=\[transforming,nodata,materialized,failed,waiting\]  
    passing this parameter will further restrict the output to views with the given state.
- filter=Regexp
    apply a regular expression filter on the view path to further limit information to certain views (e.g. '?filter=.*Visit.*')
- dependencies=[true|false]  
    if a specific view is requested, setting this to true will also return information about all dependent views
- overview=[true|false]  
    only return aggregate counts about view scheduling states and not information about individual views.

**Returns**  

     {  
       "overview": {  
         "nodata": 1  
         "materialized" : 1  
      },  
      "views": [{  
        "view": "example.osm/Stage/2015/01",  
        "status": "nodata"  
      }, {  
        "view": "example.osm/Stage/2015/02",  
        "status": "materialized"  
      }]  
    }  




### transformations

transformations lists the transformation status, i.e., executing transformations.


Method: GET  
Path:  /transformations


**Parameters:**  

**Returns**  

    {
	  "overview": {
	    "idle": 860
	  },
	  "transformations": [{
	    "actor": "/user/root/actions/oozie-154",
	    "typ": "oozie",
	    "status": "idle"
	  }, {
	    "actor": "/user/root/actions/pig-40",
	    "typ": "pig",
	    "status": "idle"
	  }, {
	    "actor": "/user/root/actions/pig-61",
	    "typ": "pig",
	    "status": "idle"
	  }, {
	    "actor": "/user/root/actions/hive-162",
	    "typ": "hive",
	    "status": "idle"
	  }]
	}

### queues 
Returns the existing queues and their contents. Queues contain Tranformations and Actions
that did not yet get processed.

Method: GET  
Path: /queues

**Parameters:**  
- typ=String  
    filter by transformation type (e.g. oozie,hive,mapreduce,filesystem,morphline,pig)
- filter=String  
    filter regular expression (e.g. '?filter=my.database%2F.%2FPartition1%2F.')


**Returns**  


	{  
       "overview": {  
        "hive": 1  
        "oozie" : 1  
      },  
      "queues": {  
        "oozie": [],
        "hive : [] 
    }  

### materialize 
Materialize view(s) - i.e., load the data of the designated views and their dependencies, if not already materialized and current in terms of data and transformation version checksums.

The materialization command ID is returned as a result.

Method: GET  
Path: /materialize/`ViewUrlPath`  


**Parameters:**  

- `status=<status>`: materialize all views that have a given status (e.g. 'failed')
- `mode=RESET_TRANSFORMATION_CHECKSUMS`: ignore transformation version checksums when detecting whether views need to be rematerialized. The new checksum overwrites the old checksum. Useful when changing the code of transformations in way that does not require recomputation.
- `mode=RESET_TRANSFORMATION_CHECKSUMS_AND_TIMESTAMPS`: perform a transformation dry run, only update checksums and timestamps. Useful when data is put on HDFS manually and shall be accepted as is by schedoscope.


**Returns**  

	{
 	 "id": "materialize_view::schedoscope.example/Example/2015/06::20150617183559",
 	 "start": "6/17/15 6:35 PM",
 	 "status": {
  	  "submitted": 1
 	 }
	
### invalidate
Method: GET  
Path: /invalidate/`ViewUrlPath`  


**Parameters:**  
- `status=<status>`: invalidate all views that have a given status (e.g. 'failed')
- `viewPattern=viewPattern>`: invalidate all views with URL paths matching a [view pattern](View Pattern Reference)  (e.g., `my.database/MyView/Partition1/Partition2`)
- `=filterregular=<regex>`: invalidate all views with URL paths matching a regular expression (e.g., `my.database/.*/Partition1/.*`)
- `dependencies=true`: invalidate the dependencies of the views as well

**Returns**  

	{
	  "id": "invalidate::schedoscope.example/Example/2015/06::20150617183733",
	  "start": "6/17/15 6:37 PM",
	  "status": {
	    "submitted": 1
	  }

