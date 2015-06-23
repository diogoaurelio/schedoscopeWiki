# Principles

Scheduling with Schedoscope is based on three principles:

1. _Goal orientation_: with Schedoscope, you specify the views you want and the scheduler takes care that the corresponding data are loaded.

2. _Self-sufficiency_: Schedoscope has all information about views available: structure, dependencies, transformation logic. The scheduler thus can start out from an empty metastore and create all tables and partitions as data is loaded.

3. _Change detection_: Schedoscope implements measures to automatically detect changes to view structure and computation logic; as it is self-sufficient, it can then automatically recompute potentially outdated views.

# Materialization

Loading views is called materialization. Materialization is issued with the `materialize` command issued via the [REST API](Schedoscope REST API), the [command client](Command Reference), or on the [shell](Command Reference). The views to materialize are addressed using a [pattern syntax](View Pattern Reference), which also allows for addressing multiple views.

For example:

    materialize -v app.eci.datamart/SearchExport/SHOP10/2015/05

