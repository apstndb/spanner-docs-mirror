GoogleSQL for Spanner supports the following syntax to use GQL within SQL queries.

## Language list

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/reference/standard-sql/graph-sql-queries#graph_table_operator"><code dir="ltr" translate="no">        GRAPH_TABLE       </code> operator</a></td>
<td>Performs an operation on a graph in the <code dir="ltr" translate="no">       FROM      </code> clause of a SQL query and then produces a table with the results.</td>
</tr>
</tbody>
</table>

## `     GRAPH_TABLE    ` operator

``` text
FROM GRAPH_TABLE (
  property_graph_name
  multi_linear_query_statement
) [ [ AS ] alias ]
```

#### Description

Performs an operation on a graph in the `  FROM  ` clause of a SQL query and then produces a table with the results.

With the `  GRAPH_TABLE  ` operator, you can use the [GQL syntax](/spanner/docs/reference/standard-sql/graph-query-statements) to query a property graph. The result of this operation is produced as a table that you can use in the rest of the query.

#### Definitions

  - `  property_graph_name  ` : The name of the property graph to query for patterns.
  - `  multi_linear_query_statement  ` : You can use GQL to query a property graph for patterns. For more information, see [Graph query language](/spanner/docs/reference/standard-sql/graph-query-statements) .
  - `  alias  ` : An optional alias, which you can use to refer to the table produced by the `  GRAPH_TABLE  ` operator elsewhere in the query.

#### Examples

**Note:** The examples in this section reference a property graph called [`  FinGraph  `](/spanner/docs/reference/standard-sql/graph-schema-statements#fin_graph) .

You can use the `  RETURN  ` statement to return specific node and edge properties. For example:

``` text
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
```

You can use the `  RETURN  ` statement to produce output with graph pattern variables. These variables can be referenced outside `  GRAPH_TABLE  ` . For example,

``` text
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
```

The following query produces an error because `  id  ` isn't included in the `  RETURN  ` statement, even though this property exists for element `  n  ` :

``` text
SELECT name, id
FROM GRAPH_TABLE(
  FinGraph
  MATCH (n:Person)
  RETURN n.name
);
```

The following query produces an error because directly outputting the graph element `  n  ` is not supported. Convert `  n  ` to its JSON representation using the `  SAFE_TO_JSON  ` for successful output.

``` text
-- Error
SELECT n
FROM GRAPH_TABLE(
  FinGraph
  MATCH (n:Person)
  RETURN n
);
```

``` text
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
```
