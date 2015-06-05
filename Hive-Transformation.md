# Summary
Hive transformations facilitate the computation of views using HiveQL queries. For this purpose, they also support the deployment and registration of user-defined functions (UDF).

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

