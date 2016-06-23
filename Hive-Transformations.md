# Summary
Hive transformations facilitate the computation of views using HiveQL queries. For this purpose, they also support the deployment and registration of user-defined functions (UDF).

> Please note that Hive queries are no longer submitted via Hive-Server but directly using the Hive-Exec library.
> In order to ease heap consumption within Schedoscope by avoiding local map join processing, all queries are issued 
> by default with the Hive setting `hive.auto.convert.join` set to `false`. If a query needs map joins for performance
> reasons, set `hive.auto.convert.join` to `true` by either a adding `SET` statement in front of your query or 
> passing `hive.auto.convert.join -> "true"` within the `settings` map of `insertInto()`.

# Syntax
    case class HiveTransformation(sql: String, udfs: List[Function] = List())

# Description
Hive Transformations have the following parameters:

* `sql`: the HiveQL query string.  
         It supports `${parameter}` style placeholders, which are replaced at query execution time by the values passed using the `.configureWith()` clause (see below).

* `udfs`: a list of `org.apache.hadoop.hive.metastore.api.Function` function descriptors, defaulting to an empty 
  list.  
  If not yet existing for the database / package the current view resides in, these functions are created before
  executing the hive query.  
  Please note that within the query functions have to be fully qualified by the database name when called.  
  The `withFunctions()` helper functions below simplifies the creation of Hive function descriptors.

# Helpers

The following helper functions make life with Hive transformations easier:

### insertInto

`insertInto()` generates an `INSERT INTO` statement prelude appropriate for a view along with a static `PARTITION` clause in case the view is partitioned. The benefit of using `insertInto()` is that one only needs to focus on the `SELECT` statement producing the view data and that setting of partition values is handled correctly.

    insertInto(view: View, selectStatement: String, settings: Map[String, String] = Map())

* `view`: the view for which to generate the `INSERT INTO` statement prelude
* `selectStatement`: the select statement producing the columns of the view
* `settings`: A map with settings - value pairs. These are translated to `SET` clauses executed before `INSERT`

### insertDynamicallyInto

`insertDynamicallyInto()` generates an `INSERT INTO` statement prelude appropriate for a view along with a dynamic `PARTITION` clause. As with `insertInto()`, one only needs to focus on the `SELECT` statement producing the view data. As it is common with Hive and dynamic partitioning, the `SELECT` statement needs to produce the partition values as the last columns in the order of the view parameters.

    insertDynamicallyInto(view: View, selectStatement: String, settings: Map[String, String] = Map())

* `view`: the view for which to generate the `INSERT INTO` statement prelude
* `selectStatement`: the select statement producing the columns of the view
* `settings`: A map with settings - value pairs. These are translated to `SET` clauses executed before `INSERT`

### withFunctions

`withFunctions()` produces a list of `org.apache.hadoop.hive.metastore.api.Function` Hive function descriptors as required when registering and calling custom UDFs from a Hive transformation. It receives a map mapping the UDF names to be registered to the classes implementing those UDFs. It traverses the classpath looking for the jar files where the UDF class is defined and fills in the appropriate function descriptors such that proper `CREATE FUNCTION`  statements can be produced.

    def withFunctions(view: View, functions: Map[String, Class[_]] = Map())

* `view`: the view for which to register the function
* `functions`: a map of user-defined function names to classes implementing the functions.

## queryFrom

`queryFrom()` reads a Hive query from a given file path.

    def queryFrom(filePath: String): String

Parameters:
* `filePath`: the local file path from which to read the script.

## queryFromResource

`queryFromResource()` reads a Hive query from a resource on the classpath.

    def queryFromResource(resourcePath: String): String

Parameters:
* `resourcePath`: the resource path from which to read the script.

# Examples

An example of a minimal Hive transformation receiving a query directly as a string:

    transformVia(() =>
      HiveTransformation(
        """
        INSERT INTO example.example_view
        SELECT id, name FROM example.source_view
        """
      ))

A more meaningful example with a parameterized query:

    transformVia(() =>
      HiveTransformation(
        """
        INSERT INTO ${orderTable}
        PARTITION (year = '${year}', month = '${month}')
        SELECT id, order_number, order_amount 
        FROM ${clickstreamTable}
        WHERE year = '${year}' and month = '${month}'
        AND eventType = 'order'
        """
      ).configureWith(
        Map(
          "orderTable" -> this.tableName,
          "clickstreamTable" -> this.clickstream.tableName,
          "year" -> this.year.v.get,
          "month" -> this.month.v.get,
    )))

The previous example making use of the `insertInto()` helper to avoid the partition clause specification:

    transformVia(() =>
      HiveTransformation(
        insertInto(this,
          """
          SELECT id, order_number, order_amount 
          FROM ${clickstreamTable}
          WHERE year = '${year}' and month = '${month}'
          AND eventType = 'order'
          """
      )).configureWith(
        Map(
          "clickstreamTable" -> this.clickstream.tableName,
          "year" -> this.year.v.get,
          "month" -> this.month.v.get,
    )))

The example enhanced by the declaration and call of a UDF using withFunctions:

    transformVia(() =>
      HiveTransformation(
        insertInto(this,
          """
          SELECT id, order_number, order_amount, ${orderDb}.calc_tax(order_amount, order_country) 
          FROM ${clickstreamTable}
          WHERE year = '${year}' and month = '${month}'
          AND eventType = 'order'
          """
       ),
       withFunctions(this, Map("calc_tax" -> classOf[CalcTaxUDF]))  
     ).configureWith(
        Map(
          "orderDb" -> this.dbName,
          "clickstreamTable" -> this.clickstream.tableName,
          "year" -> this.year.v.get,
          "month" -> this.month.v.get,
    )))

The same example with the query moved into a resource file and accessing it using the queryFromResource helper:

    transformVia(() =>
      HiveTransformation(
        insertInto(this, queryFromResource("hiveql/order/insert_order.sql")),
        withFunctions(this, Map("calc_tax" -> classOf[CalcTaxUDF]))  
     ).configureWith(
        Map(
          "orderDb" -> this.dbName,
          "clickstreamTable" -> this.clickstream.tableName,
          "year" -> this.year.v.get,
          "month" -> this.month.v.get,
    )))

Finally, a "real" example taken from the Nodes view of schedoscope tutorial. It makes use of the `settings` parameter of the `insertInto` clause to enable GZIP compression for Parquet. Furthermore, the setting of standard `${parameter}` query parameters has been refactored into a separate, project-specific function:

    transformVia(() =>
      HiveTransformation(
        insertInto(this,
          queryFromResource("hiveql/processed/insert_nodes.sql"),
          settings = Map("parquet.compression" -> "GZIP")),
        withFunctions(this, Map("collect" -> classOf[CollectUDAF]))
      ).configureWith(defaultHiveQlParameters(this)))

# Packaging and Deployment

When using custom UDFs within Hive transformations, the code implementing the UDF must be properly packaged such that Schedoscope can automatically deploy it on the cluster and `withFunctions` can find it to create correct `CREATE FUNCTION` statements.

Generally, there are two ways of deploying custom UDFs:

## In-Project

The UDF can be part of the same codebase as the Schedoscope views. In this case, the UDF and the classes it depends on should be bundled into an additional project jar. The filename of this jar should end with `-hive.jar` and be on the classpath when launching Schedoscope. 

During startup, Schedoscope discovers `-hive.jar` files on the classpath and uploads them to HDFS into the folder configured by the property `schedoscope.transformations.hive.location` suffixed by the environment config property `schedoscope.app.enviroment`. Thus, the default folder is `/tmp/schedoscope/hive/dev/`.

With Maven, a `-hive.jar` jar can be packaged using the Proguard plugin, for example:

    <plugin>
        <groupId>com.github.wvengen</groupId>
        <artifactId>proguard-maven-plugin</artifactId>
        <version>2.0.8</version>
        <executions>
            <execution>
                <id>package-hive-resources</id>
                <phase>package</phase>
                <goals>
                    <goal>proguard</goal>
                </goals>
                <configuration>
                    <obfuscate>false</obfuscate>
                    <injar>classes</injar>
                    <libs>
                        <lib>${java.home}/lib/rt.jar</lib>
                    </libs>
                    <options>
                        <option>-keep public class example.functions.** { *; }</option>
                        <option>-dontnote **</option>
                        <option>-dontwarn</option>
                        <option>-dontshrink</option>
                        <option>-dontoptimize</option>
                        <option>-dontskipnonpubliclibraryclassmembers</option>
                    </options>
                    <outjar>${project.build.finalName}-hive.jar</outjar>
                    <inFilter>**.class</inFilter>
                    <assembly>
                        <inclusions>
                            <inclusion>
                                <groupId>joda-time</groupId>
                                <artifactId>joda-time</artifactId>
                            </inclusion>
                        </inclusions>
                    </assembly>
                </configuration>
            </execution>
        </executions>
    </plugin>

Here, all UDF and dependend classes in the package `example.functions` are bundled, along with the dependend classes from the Joda time library. The resulting jar ends up in the Maven target directory.

## Ex-Project

Sometimes one would like to use UDF libraries from external sources. For example, UDFs from the [Klout brickhouse library](https://github.com/klout/brickhouse). In this case, the deployment process should copy the library jar into a separate library folder. That folder should be configured in the property `schedoscope.transformations.hive.libDirectory` and put into the classpath. 

Upon launch, Schedoscope uploads all jars in this folder into the folder configured by the property `schedoscope.transformations.hive.location` suffixed by the environment config property `schedoscope.app.enviroment`. Again, the default folder is `/tmp/schedoscope/hive/dev/`.

With Maven, deployment of library jars can be achieved using the assembly plugin. For example:

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>2.4</version>
        <executions>
            <execution>
                <id>hive</id>
                <phase>package</phase>
                <goals>
                    <goal>single</goal>
                </goals>
                <configuration>
                    <outputDirectory>deployment</outputDirectory>
                    <descriptor>src/main/assemble/hive.xml</descriptor>
                </configuration>
            </execution>
        </executions>
    </plugin>

along with the assembly descriptor

    <assembly>
        <id>package</id>
        <includeBaseDirectory>false</includeBaseDirectory>
        <formats>
            <format>dir</format>
        </formats>
        
        <fileSets>
            <fileSet>
                <directory>${basedir}/target/dependencies/</directory>
                <outputDirectory>/udfs/</outputDirectory>
                <includes>
                    <include>brickhouse*.jar</include>
                </includes>
            </fileSet>
        </fileSets>
    </assembly>

would copy the brickhouse libary jar to the folder `${baseDirectory}/deployment/deployment-package/udfs`, which then would need to be configured into `schedoscope.transformations.hive.libDirectory`.

# Change detection

Schedoscope tries to automatically detect changes to Hive transformation-based views and to initiate rematerialization of views if the tranformation logic has potentially changed. 

This is achieved by protocoling transformation version checksums with each materialized view in the Hive metastore. If the transformation version checksum differs between the materialized version and the current one, the view is rematerialized.

The following properties of Hive transformations contribute to the transformation version checksum:
- the query string in the `sql` parameter. It is normalized to avoid checksums changing because of  whitespaces, formatting, indentation, and comments as well as `SET`tings
- the names of the deployed jar files. I.e., those specified by the property `schedoscope.transformations.hive.libDirectory` or those ending with `-hive.jar`

Hence, if you change the SQL query string a view gets rematerialized. If you change an UDF's implementation and want to trigger rematerialization, you need to change the name of the jar the UDF resides in, e.g., by incrementing a version number.