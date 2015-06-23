# Principles

Scheduling with Schedoscope is based on three principles:

1. _Goal orientation_: with Schedoscope, you specify the views you want and the scheduler takes care that the corresponding data are loaded.

2. _Self-sufficiency_: Schedoscope has all information about views available: structure, dependencies, transformation logic. The scheduler thus can start out from an empty metastore and create all tables and partitions as data are loaded.

3. _Change detection_: Schedoscope implements measures to automatically detect changes to view structure and computation logic; as it is self-sufficient, it can then automatically recompute potentially outdated views.

# Materialization

Loading views is called materialization. Materialization is issued with the `materialize` command issued via the [REST API](Schedoscope REST API), the [command client](Command Reference), or on the [shell](Command Reference). The views to materialize are addressed using a [pattern syntax](View Pattern Reference), which also allows for addressing multiple views.

For example:

    materialize -v app.eci.datamart/SearchExport/SHOP10/2015/05

Materialization proceeds in two phases:

## View Instantiation

Before materialization itself is started, all views impacted by a materialization command are instantiated. These are the views addressed by the materialization command plus their dependencies, recursively.

Instantiation comprises the following steps:
- instantiating an Akka actor for managing the view's state;
- creating the table for the view if it does not exist;
- create a partition for the view if it does not exist;
- pull transformation timestamp and version information out of the Hive Metastore.

Once Schedoscope has instantiated a view it remains instantiated. Thus, effectively, after a warmup period, less and less views will be initialized during materialization requests.

## Materialization Proper

Once all required views are initialized, materialization itself starts. During that process, essentially, each view impacted by materialization:

1. sends a materialization request to the views it depends on;
2. waits until the dependencies have materialized;
3. executes it own transformation by creating a transformation action and passing it to the actions manager;
4. stores transformation version and timestamp in the Metastore after successful execution of its transformation;
5. enters materialized state and notifies any dependent views on that state change.



