GoogleSQL for Spanner supports the following syntax to use graphs within SQL queries.

## Language list

| Name                                                                                                                               | Summary                                                                                                          |
| ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| [`GRAPH_TABLE` operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-sql-queries#graph_table_operator) | Performs an operation on a graph in the `FROM` clause of a SQL query and then produces a table with the results. |

## `GRAPH_TABLE` operator

    FROM GRAPH_TABLE (
      property_graph_name
      multi_linear_query_statement
    ) [ [ AS ] alias ]

#### Description

Performs an operation on a graph in the `FROM` clause of a SQL query and then produces a table with the results.

With the `GRAPH_TABLE` operator, you can use the [GQL syntax](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements) to query a property graph. The result of this operation is produced as a table that you can use in the rest of the query.

#### Definitions

  - `property_graph_name` : The name of the property graph to query for patterns.
  - `multi_linear_query_statement` : You can use GQL to query a property graph for patterns. For more information, see [Graph query language](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements) .
  - `alias` : An optional alias, which you can use to refer to the table produced by the `GRAPH_TABLE` operator elsewhere in the query.

#### Examples

> **Note:** The examples in this section reference a property graph called [`FinGraph`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-schema-statements#fin_graph) .

You can use the `RETURN` statement to return specific node and edge properties. For example:

    SELECT name, id
    FROM GRAPH_TABLE(
      FinGraph
      MATCH (n:Person)
      RETURN n.name AS name, n.id AS id
    );
    
    /*-----------+
     | name | id |
     +-----------+
     | Alex | 1  |
     | Dana | 2  |
     | Lee  | 3  |
     +-----------*/

You can use the `RETURN` statement to produce output with graph pattern variables. These variables can be referenced outside `GRAPH_TABLE` . For example,

    SELECT n.name, n.id
    FROM GRAPH_TABLE(
      FinGraph
      MATCH (n:Person)
      RETURN n
    );
    
    /*-----------+
     | name | id |
     +-----------+
     | Alex | 1  |
     | Dana | 2  |
     | Lee  | 3  |
     +-----------*/

The following query produces an error because `id` isn't included in the `RETURN` statement, even though this property exists for element `n` :

    SELECT name, id
    FROM GRAPH_TABLE(
      FinGraph
      MATCH (n:Person)
      RETURN n.name
    );

The following query produces an error because directly outputting the graph element `n` is not supported. Convert `n` to its JSON representation using the `SAFE_TO_JSON` function for successful output.

    -- Error
    SELECT n
    FROM GRAPH_TABLE(
      FinGraph
      MATCH (n:Person)
      RETURN n
    );

    SELECT SAFE_TO_JSON(n) as json_node
    FROM GRAPH_TABLE(
      FinGraph
      MATCH (n:Person)
      RETURN n
    );
    
    /*---------------------------+
     | json_node                 |
     +---------------------------+
     | {"identifier":"mUZpbk...} |
     | {"identifier":"mUZpbk...} |
     | {"identifier":"mUZpbk...} |
     +--------------------------*/
