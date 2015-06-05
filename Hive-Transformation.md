# Summary
Hive transformations facilitate the computation of views using HiveQL queries. For this purpose, they also support the deployment and registration of user defined functions (UDF).

# Syntax

    case class HiveTransformation(sql: String, udfs: List[Function] = List())