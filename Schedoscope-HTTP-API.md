Schedoscope can be remotely controlled via a simple HTTP API. It can be used for triggering new materializations or notify the scheduler upon the arrival of new data, but also for checking view scheduling states remotely.

## API Request Types
Currently, all remote requests to Schedoscope use the HTTP GET method. Parameters are passed as URL parameters and need to be URL-encoded appropriately. Results are returned as JSON documents. 

In case of errors, HTTP error codes <> 200 are set. Currently these are:
* Bad Request (400): invalid parameters have been passed. Fix the call.
* Internal Server Error (500): some unforeseen problem occurred internally within Schedoscope. This should not happen and you need to check the logs to find out what's wrong.

The currently implemented request types are as follows:

### views
List all instantiated views  

Method: GET  
Path: /views/  
or  
Path: /views/`ViewPattern`  

If a `ViewPattern` is given, only information about views matching the [pattern](View-Pattern-Reference) is returned

**Parameters:**  

- `status=(transforming|nodata|materialized|failed|retrying|waiting)`
    passing this parameter will further restrict the output to views with the given state.
- `filter=Regexp`
    apply a regular expression filter on the view path to further limit information to certain views (e.g. '?filter=.*Visit.*')
- `dependencies=(true|false)`  
    if a specific view is requested, setting this to true will also return information about all dependent views
- `overview=(true|false)`  
    only return aggregate counts about view scheduling states and not information about individual views.

**Returns**  

A record consisting of an overview with the views returned summarized according to the view status and the list of views with their individual status.

E.g.,

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
Lists the status of transformation drivers, in particular the currently running transformations.

Method: GET  
Path:  /transformations

**Parameters:**  
- `status=(running|idle)`
    passing this parameter will further restrict the output to transformation drivers with the given state.
- `filter=Regexp`
    apply a regular expression filter on driver name (e.g. '?filter=.*hive.*')

**Returns**  

A record consisting of an overview with the transformation drivers returned summarized according to their status and the list of drivers with their individual status.

E.g.,

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
Returns the existing transformation queues and their contents. All transformations to be executed are lined up in those queues.

Method: GET  
Path: /queues

**Parameters:**  
- `typ=String`  
    filter by transformation type (e.g. oozie,hive,mapreduce,filesystem,pig)

**Returns**  

A record consisting of an overview summarizing the number of transformations queued up per type and the heads of the transformation type-specific queues.

E.g.,

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

The views being materialized are returned along with their state prior to receiving the materialize command. 

Method: GET  
Path: /materialize/`ViewPattern`  

Refer to [the view pattern reference](View-Pattern-Reference) for how to specify a valid `ViewPattern`.

**Parameters:**  

- `status=(transforming|nodata|materialized|failed|retrying|waiting)`
   materialize all views that have a given status (e.g. 'failed')
- `mode=RESET_TRANSFORMATION_CHECKSUMS`
  ignore transformation version checksums when detecting whether views need to be rematerialized. The new checksum overwrites the old checksum. Useful when changing the code of transformations in way that does not require recomputation.
- `mode=RESET_TRANSFORMATION_CHECKSUMS_AND_TIMESTAMPS`
   perform a "dry run" where transformation checksums and timestamps are set along the usual rules, however with no actual transformations taking place. As a result, all checksums in the metastore should be current and transformation timestamps should be consistent, such that no materialization will take place upon subsequent normal materializations.
- `mode=TRANSFORM_ONLY`
  materialize the given views, but without asking the views' dependencies to materialize as well. This is useful when a transformation higher up in the dependency lattice has failed and you want to retry it without potentially rematerializing all dependencies.
- `mode=SET_ONLY`
  force the given views into the materialized state. No transformation is performed, and all the views' transformation timestamps and checksums are set to current.

**Returns**  

A record consisting of an overview of the views being materialized summarized according to the view status prior to materialization and the list of views being materialized with their individual status.

E.g.,

     {  
       "overview": {  
         "nodata": 1  
         "materialized" : 1  
      },  
      "views": [{  
        "view": "example.osm/Stage/2015/01",  
        "status": "receive"  
      }, {  
        "view": "example.osm/Stage/2015/02",  
        "status": "materialized"  
      }]  
    }  
	
### invalidate
Invalidate views, i.e., force a materialization upon subsequent materialize.

The views invalidated are returned along with their state prior to invalidation.

Method: GET  
Path: /invalidate/`ViewPattern`  

Refer to [the view pattern reference](View-Pattern-Reference) for how to specify a valid `ViewPattern`.


**Parameters:**  
- `status=(transforming|nodata|materialized|failed|retrying|waiting)`
   materialize all views that have a given status (e.g. 'failed')
- `filter=Regexp`
    invalidate all views with their path matching regular expression (e.g. '?filter=.*Visit.*')
- `dependencies=(true|false)`
   invalidate the dependencies of the views as well

**Returns**  

A record consisting of an overview of the views invalidated summarized according to the view status prior to invalidation and the list of invalidated views with their individual status.

     {  
       "overview": {  
         "nodata": 1  
         "materialized" : 1  
      },  
      "views": [{  
        "view": "example.osm/Stage/2015/01",  
        "status": "materialized"  
      }, {  
        "view": "example.osm/Stage/2015/02",  
        "status": "materialized"  
      }]  
    }  