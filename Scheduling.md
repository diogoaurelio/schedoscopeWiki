Scheduling with Schedoscope is based on three principles:

1. _Goal orientation_: with Schedoscope, you specify the views you want and the scheduler takes care that the corresponding data are materialized.

2. _Self-sufficiency_: Schedoscope has all information about views available: structure, dependencies, transformation logic. The scheduler thus can start out from an empty metastore and create all tables and partitions as data are materialized.

3. _Reloading is loading_: Schedoscope implements measures to automatically detect changes to view structure and computation logic; as it is self-sufficient, it can then automatically rematerialize potentially outdated views.

## Materialization

Loading views is called materialization. Materialization is triggered with the `materialize` command via either the [REST API](Schedoscope REST API), the [command client](Command Reference) or the [shell](Command Reference). The views to materialize are addressed using a [pattern syntax](View Pattern Reference), which also allows for addressing multiple views.

For example:

    materialize -v app.eci.datamart/SearchExport/SHOP10/2015/05

Materialization proceeds in two phases:

#### View Instantiation

Before materialization itself is started, all views impacted by a materialization command are instantiated. These are the views addressed by the materialization command plus their dependencies, recursively.

Instantiation comprises the following steps:
- instantiating an Akka actor for managing the view's state;
- creating the table for the view if it does not exist;
- create a partition for the view if it does not exist;
- pull transformation timestamp and version information out of the Hive Metastore.

Once Schedoscope has instantiated a view, it remains instantiated. Thus, effectively, after a warmup period, less and less views will be initialized during materialization requests.

#### Materialization Proper

Once all required views are initialized, materialization itself starts. During that process, essentially, each view (actor) impacted by materialization:

1. sends a materialization request to the views (i.e., the view actors) it depends on;
2. waits until the dependencies have materialized;
3. triggers the execution of its own transformation by passing an appropriate transformation action to the actions manager;
4. waits until its transformation has successfully executed;
5. stores transformation version and timestamp in the Metastore;
6. enters materialized state and notifies any dependent views about its state change.

## Change Detection

Schedoscope attempts to automatically detect changes to data, data structure, and application logic and reschedule computation of views accordingly. Change detection is again spread across the phases view instantion and materialization proper and is based on

- _View DDL checksums_: a checksum on a view's `CREATE TABLE` DDL statement;
- _Transformation version checksums_: a transformation type-specific checksum on the `transformVia` clause of the view (see the respective transformation reference documentation sections in this wiki) ;
- _Transformation timestamps_: timestamp of the last materialization of the view.

#### View Instantiation

During a view's instantiation, Schedoscope checks for the existence of a table for the view in the metastore:

- In case the table exists, the current view DDL checksum is compared with the checksum of stored with the table in the metastore; 

- If the checksums differ - i.e., if the table structure has changed in any way - the table and all its partitions are dropped and the current `CREATE TABLE` statement is executed. The new view DDL checksum is stored with the table in the metastore.

Moreover, view instantiation also takes care that a table partition is available for each view. Missing partitions are created; for partitions that already exist, view instantiation loads the 

- transformation version checksums and
- transformation timestamps 

into the respective view actors for use during the following materialization proper phase.

#### Materialization Proper

During materialization proper, transformation version checksums and transformation timestamps are used for change detection:

- when the dependencies of a view have been materialized, the most recent transformation timestamp of the dependencies is compared to the transformation timestamp of the view. If that timestamp is newer, the current view is materialized again because input data might have changed;

- if the current view's transformation version checksum differs from the one previously stored, the view is rematerialized because transformation logic might have changed;

After rematerialization, transformation version checksum and timestamp are updated in the Hive metastore.

Note that rematerialization of a view results in a new transformation timestamp, which might affect the views depending on that view and trigger their rematerialization.

## Scheduling States

![View Scheduling States](https://github.com/ottogroup/schedoscope/blob/master/schedoscope-tutorial/docs/pictures/scheduling%20states.png)

On the way its materialization, a view can go through the following states (see picture above):

| State | Description | Next possible states |
----------------------------------------------
| receive |   | |