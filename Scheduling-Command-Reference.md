Schedoscope can be controlled with scheduling commands. There are two ways of issuing these commands:

* using the Schedoscope shell command prompt opening when Schedoscope is not launched as a daemon;
* using the Schedoscope client based on the [HTTP API](Schedoscope HTTP API)

## Schedoscope Shell

Schedoscope can be controlled by a command line shell. This shell is accessible when Schedoscope is launched as a REST service via the class `org.schedoscope.scheduler.rest.server.SchedoscopeRestService` passing it the `--shell` option.

For example:

    java -cp ${MY_SCHEDOSCOPE_CP} -Dlogback.configurationFile=${MY_SCHEDOSCOPE_FOLDER}/eci-logging.xml -Dconfig.file=${MY_SCHEDOSCOPE_FOLDER}/schedoscope.conf org.schedoscope.scheduler.api.SchedoscopeRestService --shell

## Schedoscope HTTP Client

There is also the Schedoscope HTTP client, which communicates with a running Schedoscope instance via the HTTP interface. The client can be started via the class `org.schedoscope.scheduler.rest.client.SchedoscopeRestCli` passing it the command to execute.

For example:

    java -cp ${MY_SCHEDOSCOPE_CP} -Dlogback.configurationFile=${MY_SCHEDOSCOPE_FOLDER}/eci-logging.xml -Dconfig.file=${MY_SCHEDOSCOPE_FOLDER}/schedoscope.conf org.schedoscope.scheduler.rest.client.SchedoscopeRestCli views -o

For this work, the configuration properties

* `schedoscope.webservice.host` and
* `schedoscope.webservice.port`

must point to the running Schedoscope HTTP service

## View Paths

Schedoscope views are referenced by their name and their parameters. This makes for view path. For easy specification of views and view ranges, Schedoscope makes use of a view path pattern language:

* [Specification of View Paths](View-Pattern-Reference)

## Commands

### views

`views` lists all instantiated views along with their status.

Supported options:
- `-s`, `--status <status>`: filter views by their status (e.g. 'transforming')
- `-v`, `--viewPattern <viewPattern>`: select only views with paths matching a [view pattern](View Pattern Reference)  (e.g., `my.database/MyView/Partition1/Partition2`)
- `-f`, `--filterregular <regex>`: select only views with URL paths matching a regular expression (e.g., `my.database/.*/Partition1/.*`)
- `-i`, `--issueFilter (errors|incomplete)`: filter views that have been computed based on failed dependencies or dependencies with no data.
- `-d`, `--dependencies`: include dependencies in this view list
- `-o`, `--overview`: only show a count of views by status

Examples:

    views -o

    materialized: 94788
    waiting: 2
    nodata: 49736

    views -s waiting

    Details:
    +----------------------------------------------+---------+-------+
    |                     VIEW                     |  STATUS | PROPS |
    +----------------------------------------------+---------+-------+
    | example.datamart/SearchExport/SHOP10/2015/05 | waiting |       |
    |               example.datahub/Search/2015/04 | waiting |       |
    +----------------------------------------------+---------+-------+

    views -v example.datamart/SearchExport/EC0601/2015/05

    Details:
    +----------------------------------------------+---------+-------+
    |                     VIEW                     |  STATUS | PROPS |
    +----------------------------------------------+---------+-------+
    | example.datamart/SearchExport/SHOP10/2015/05 | waiting |       |
    +----------------------------------------------+---------+-------+

    views -d -v example.datamart/SearchExport/EC0601/2015/05

    Details:
    +----------------------------------------------+--------------+-------+
    |                     VIEW                     |    STATUS    | PROPS |
    +----------------------------------------------+--------------+-------+
    | app.eci.datamart/SearchExport/SHOP10/2015/05 | waiting      |       |
    |               example.datahub/Search/2015/05 | waiting      |       |
    |                example.datahub/Event/2015/05 | transforming |       |
    +----------------------------------------------+--------------+-------+

    views -f *.datamart/.*/2015/05

    +----------------------------------------------------+--------------+-------+
    |                        VIEW                        |    STATUS    | PROPS |
    +----------------------------------------------------+--------------+-------+
    |     example.datamart/AffinityFeatureMatrix/2015/05 | materialized |       |
    |       example.datamart/SearchExport/SHOP10/2015/05 |      receive |       |
    +----------------------------------------------------+--------------+-------+

### transformations

`transformations` lists the status of transformation drivers, i.e., executing transformations.

Supported options:
- `-s`, `--status`: filter by status (e.g. 'queued, running, idle')
- `-f`, `--filter <regex>: filter by regular expression (e.g. '.*hive-1.*').

Examples:

    transformations -s idle

    +----------------------------------+--------+---------+------+-------------+-------+
    |       TRANSFORMATION DRIVER      | STATUS | STARTED | DESC | TARGET_VIEW | PROPS |
    +----------------------------------+--------+---------+------+-------------+-------+
    |     /user/root/actions/oozie-154 |   idle |         |      |             |       |
    |        /user/root/actions/pig-40 |   idle |         |      |             |       |
    ...

    transformations -f .*hive.*

    +----------------------------------+--------+---------+------+-------------+-------+
    |               ACTOR              | STATUS | STARTED | DESC | TARGET_VIEW | PROPS |
    +----------------------------------+--------+---------+------+-------------+-------+
    |       /user/root/actions/hive-40 |   idle |         |      |             |       |
    |       /user/root/actions/hive-41 |   idle |         |      |             |       |
    ...   

### materialize

Materialize view(s) - i.e., load the data of the designated views and their dependencies - if not already materialized and current in terms of data and transformation version checksums.

The views being materialized are returned along with their status prior to materialization.

Supported options:
- `-s`, `--status <status>`: materialize all views that have a given status (e.g. 'failed')
- `-v`, `--viewPattern <viewPattern>`: materialize all views with paths matching a [view pattern](View Pattern Reference)  (e.g., `my.database/MyView/Partition1/Partition2`)
- `-f`, `--filterregular <regex>`: materliaze only views with URL paths matching a regular expression (e.g., `my.database/.*/Partition1/.*`). These views must have been initialized before, e.g, by a views command.
- `-i`, `--issueFilter (errors|incomplete)`: materialize views that have been computed based on failed dependencies or dependencies with no data.
- `-m`, `--mode RESET_TRANSFORMATION_CHECKSUMS`: ignore transformation version checksums when detecting whether views need to be rematerialized. The new checksum overwrites the old checksum. Useful when changing the code of transformations in way that does not require recomputation.
- `-m`, `--mode RESET_TRANSFORMATION_CHECKSUMS_AND_TIMESTAMPS`: perform a "dry run" where transformation checksums and timestamps are set along the usual rules, however with no actual transformations taking place. As a result, all checksums in the metastore should be current and transformation timestamps should be consistent, such that no materialization will take place upon subsequent normal materializations.
- `-m`, `--mode TRANSFORM_ONLY`: materialize the given views, but without asking the views' dependencies to materialize as well. This is useful when a transformation higher up in the dependency lattice has failed and you want to retry it without potentially rematerializing all dependencies.
- `-m`, `--mode SET_ONLY`: force the given views into the materialized state. No transformation is performed, and all the views' transformation timestamps and checksums are set to current.

Examples:

    materialize -s failed

    materialize -v  app.eci.datamart/SearchExport/SHOP10/2015/05

    materialize -v  app.eci.datamart/SearchExport/e(SHOP10,SHOP11)/rym(201505-201410)

    materialize -v  app.eci.datamart/SearchExport/SHOP10/2015/05 -m RESET_TRANSFORMATION_CHECKSUMS

### invalidate

Invalidate previously materialized view(s) to enforce rematerialization upon the next `materialize` command.

The views invalidated are returned along with their status prior to invalidation

Supported options:
- `-s`, `--status <status>`: invalidate all views that have a given status (e.g. 'failed')
- `-v`, `--viewPattern <viewPattern>`: invalidate all views with paths matching a [view pattern](View Pattern Reference)  (e.g., `my.database/MyView/Partition1/Partition2`)
- `-f`, `--filterregular <regex>`: invalidate all views with URL paths matching a regular expression (e.g., `my.database/.*/Partition1/.*`)
- `-i`, `--issueFilter (errors|incomplete)`: invalidate views that have been computed based on failed dependencies or dependencies with no data.
- `-d`, `--dependencies`: invalidate the dependencies of the views as well

Examples:

    invalidate -s failed

    invalidate -v  app.eci.datamart/SearchExport/SHOP10/2015/05

    invalidate -v  app.eci.datamart/SearchExport/e(SHOP10,SHOP11)/rym(201505-201410)

    invalidate -v  app.eci.datamart/SearchExport/SHOP10/2015/05 -d
