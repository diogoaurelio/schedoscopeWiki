## Introduction

Schedoscope offers a rest api for remote control. The schedoscopeControl shell makes use of this API, but it may be also used to trigger new materializations or notify the scheduler upon the arrival of new data.



## REST Methods
Currently all requests to schedoscope use method GET. Parameters are passed as URL parameters, e.g.
GET /views?status=transforming

## View Url Path Specification 
Schedoscope views are referenced by their name and their parameters. For easy specification of Views
and view ranges, Schedoscope offers a special view specification language named ViewUrl
* [Specification of View Urls](ViewUrlSpecification)

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

### commands

List available API-Endpoints 
Method: GET  
Path: /commands

**Parameters:**  

**Returns**  
  
	{
		

### materialize 
Materialize view(s) - i.e., load the data of the designated views and their dependencies, if not already materialized and current in terms of data and transformation version checksums.

The materialization command ID is returned as a result.

Method: GET  
Path: /views/`ViewUrlPath`  


**Parameters:**  

**Returns**  

### invalidate
Method: GET  
Path: /invalidate/`ViewUrlPath`  


**Parameters:**  

**Returns**  


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
