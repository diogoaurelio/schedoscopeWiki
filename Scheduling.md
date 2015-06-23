# Principles

Scheduling with Schedoscope is based on three principles:

1. _Goal-driven_: with Schedoscope, you specify the views you want and the scheduler takes care that the corresponding data are loaded.

2. _Self-sufficiency_: Schedoscope has all information about views available: structure, dependencies, transformation logic. The scheduler thus start out from an empty metastore and create all tables and partitions as data is loaded.

3. _Change detection_: Schedoscope implements measures to automatically detect changes to data structures and computation logic; as it is self-sufficient, it can then automatically recompute potentially outdated data.

