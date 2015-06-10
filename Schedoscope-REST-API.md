## Introduction

Schedoscope offers a rest api for remote control. The schedoscopeControl shell makes use of this API, but it may be also used to trigger new materializations or notify the scheduler upon the arrival of new data.

## REST Methods
Currently all requests to schedoscope use method GET. Parameters are passed as URL parameters, e.g.
GET /views/status=transforming

#### views
List all currently active views  

Method: GET  
Path: /views/   

**Parameters:**  

- status=[transforming,nodata,materialized,failed,waiting]  
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
Method: GET  
Path: 

**Parameters:**  

**Returns**  

### queues 
Method: GET  
Path: 

**Parameters:**  

**Returns**  


### commands 
Method: GET  
Path: 

**Parameters:**  

**Returns**  

### materialize 
Method: GET  
Path: 

**Parameters:**  

**Returns**  

### invalidate
Method: GET  
Path: 

**Parameters:**  

**Returns**  


### newdata 
Method: GET  
Path: 

**Parameters:**  

**Returns**  

### shutdown 
Method: GET  
Path: 

**Parameters:**  

**Returns**  
