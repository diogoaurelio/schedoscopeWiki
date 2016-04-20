# Summary

You can configure parallel export of view data to a relational database via JDBC by specifying an `exportTo()` clause with `Jdbc` in a view. Whenever the view performs its transformation successfully and materializes the view's data (i.e., the data of the Hive partition the view represents) is written into a table within the specified database. 

Currently, the DBMSs Derby, MySQL, PostgreSQL, and ExaSolutions are supported.

# Syntax

    def Jdbc(
        v: View,
        jdbcConnection: String,
        dbUser: String = null,
        dbPass: String = null,
        distributionKey: Field[_] = null,
        storageEngine: String = Schedoscope.settings.jdbcStorageEngine,
        numReducers: Int = Schedoscope.settings.jdbcExportNumReducers,
        commitSize: Int = Schedoscope.settings.jdbcExportBatchSize,
        isKerberized: Boolean = !Schedoscope.settings.kerberosPrincipal.isEmpty(),
        kerberosPrincipal: String = Schedoscope.settings.kerberosPrincipal,
        metastoreUri: String = Schedoscope.settings.metastoreUri)

# Description

For the `Jdbc` export to work you need a target database and a database user who has create / drop table privileges on that database. This is required because during export each reducer initially writes into its own temporary table to avoid lock congestion before the data from these tables is merged into the target table with a single `UNION ALL` statement at the end.

The target table for a view is created automatically if it does not exist already. The name of the target table is the name of the Hive database and the Hive table name of the view, concatenated by "_". The target table has a column for each view field and partition parameter. Primitive Hive types are mapped to the corresponding data types of the target database. For strings, the used data type varies with the target database system:

* MySQL, PostgreSQL: `text`
* Derby: `varchar(32000)`
* ExaSolutions: `varchar(100000)`

Values of complex types (maps, structures, lists) are serialized to corresponding JSON structures, maps, or lists and stored as strings in the target table.

Should a view be exported more than once, the rows of previous exports are deleted in the target table to avoid duplicates. In order to achieve this, each table gets an additional column `used_filter` which contains the parameter values of the exported view.

Here is a description the parameters you must or can pass to `Jdbc` exports:

- `v`: a reference to the view being exported, usually `this` (mandatory)
- `jdbcConnection`: the JDBC connection string to the target (mandatory). Please note that you have to add the JDBC driver's jar file tp the Schedoscope classpath. Schedoscope will automatically identify the jar file based on `jdbcConnection` and upload it into the cache of the export mapreduce job
- `dbUser`: the database user for accessing the target database. No authentication by default.
- `dbPass`: the database user's password. No authentication by default
- `distributionKey`: if the target database system supports the notion of distribution keys for table partitioning suchas ExaSolutions, you can pass the view field that provides the values for the distribution key here. By default, no distribution key is enabled.
- `storageEngine`: if your target database system supports different storage engines (e.g., MySQL) you can set the one to use here. By default, the standard storage engine of the target system is used
- `numReducers`: the number of reducers to use during the exports. This defines the parallelism of the export. Defaults to the config setting `schedoscope.export.jdbc.numberOfReducers` (i.e., 10)
- `commitSize`: The batch size to use by each reducer for inserting into its temp table. Defaults to the config setting `schedoscope.export.jdbc.insertBatchSize` (i.e., false)
- `isKerberized`: is this a kerberized cluster environment? Defaults to true, if `schedoscope.kerberos.principal` is set in the configuration
- `kerberosPrincipal`: the Kerberos principal to use in a Kerberos cluster enviroment. Defaults to `schedoscope.kerberos.principal` as set in the configuration
- `metastoreUri`: the connection URI to use for the Hive metastore. Defaults to `schedoscope.metastore.metastoreUri` as set in the configuration

 
# Example
    
    package test.export

    case class ClickWithJdbcExport(
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]) extends View
           with DailyParameterization {

      val id = fieldOf[String]
      val url = fieldOf[String]

      val stage= dependsOn(() => Stage(year, month, day))

      transformVia(
        () => HiveTransformation(
          insertInto(this, s"""SELECT id, url FROM ${stage().tableName} WHERE date_id='${dateId}'""")))

       // This will create a table named dev_test_export_click_with_jdbc_export in the MySQL database test_db
       // (assuming an environment named dev).
       // The columns of this table would be:
       // id text, url text, year text, month text, day text, date_id text, used_filter text
       exportTo(() => Jdbc(this, "jdbc:mysql://localhost:3306/test_db", "root", "root"))

    }
