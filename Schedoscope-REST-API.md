## Introduction

Schedoscope offers a REST api for remote control. The schedoscopeControl shell makes use of this API, but it may be also used to trigger new materializations or notify the scheduler upon the arrival of new data.



## REST Methods
Currently all requests to schedoscope use method GET. Parameters are passed as URL parameters, e.g.
GET /views?status=transforming

## View Url Path Specification 
Schedoscope views are referenced by their name and their parameters. For easy specification of Views
and view ranges, Schedoscope offers a special view specification language named ViewUrl
* [Specification of View Urls](View-Pattern-Reference)

#### views
List all currently active views  

Method: GET  
Path: /views/  
or  
Path: /views/`ViewUrlPath`  

if a ViewUrlPath is given, only the specified View (with Parametrization) is returned

**Parameters:**  

- status=\[transforming,nodata,materialized,failed,waiting\]  
    passing this parameter will restrict the output to views with the given state.
- filter=String  
    filter regular expression to filter views to be invalidated (e.g. '?filter=my.database%2F.%2FPartition1%2F.')
- dependencies=[true|false]  
    if a specific view is requested, setting this to true will also return all dependent views
- overview=[true|false]  
    only return aggregate numbers  

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
        "status": "materializes"  
      }]  
    }  




### actions 

actions lists the status of all actions, i.e., executing transformations.


Method: GET  
Path:  /actions


**Parameters:**  

**Returns**  
	{
	  "overview": {
	    "idle": 860
	  },
	  "actions": [{
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
      "queues": [{  
        "typ": "oozie",  
       
      }, {  
        "typ": "oozie",  
 
      }]  
    }  

### materialize 
Materialize view(s) - i.e., load the data of the designated views and their dependencies, if not already materialized and current in terms of data and transformation version checksums.

The materialization command ID is returned as a result.

Method: GET  
Path: /views/`ViewUrlPath`  


**Parameters:**  

- `status=<status>`: materialize all views that have a given status (e.g. 'failed')
- `mode=RESET_TRANSFORMATION_CHECKSUMS`: ignore transformation version checksums when detecting whether views need to be rematerialized. The new checksum overwrites the old checksum. Useful when changing the code of transformations in way that does not require recomputation.


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
      "views": [{  
        "view": "example.osm/Stage/2015/01",  
        "status": "nodata"  
      }, {  
        "view": "example.osm/Stage/2015/02",  
        "status": "materializes"  
      }]  		
	}


### newdata 
**NOT YET IMPLEMENTED**
newdata allows schedoscope to operate in push mode. E.g. flume can signal the arrival of new data for a view by calling this endpoint. Subsequently schedoscope will recalculate all views that depend on the new/changed data

Method: GET  
Path: /newdata/`ViewUrlPath`


**Parameters:**  
none

**Returns**  

{
  
}

### shutdown 
Method: GET  
Path: 

**Parameters:**  

**Returns**  
