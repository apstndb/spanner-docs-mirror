**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document provides an overview using the graph query language with Spanner Graph, including its syntax for graph pattern matching, and shows you how to run queries against your graph. With Spanner Graph, you can run queries to find patterns, traverse relationships, and gain insights from your property graph data.

The examples in this document use the graph schema that you create in [Set up and query Spanner Graph](/spanner/docs/graph/set-up) . This schema is illustrated in the following diagram:

**Figure 1.** : An example of a Spanner Graph schema.

## Run a Spanner Graph query

You can use the Google Cloud console, Google Cloud CLI, client libraries, the REST API, or the RPC API to run a Spanner Graph query.

### Google Cloud console

The following steps show you how to run a query in the Google Cloud console. These steps assume you have an instance named `  test-instance  ` that contains a database named `  example-db  ` . For information about how to create an instance with a database, see [Set up and query Spanner Graph](/spanner/docs/graph/set-up) .

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the instance named `  test-instance  ` .

3.  Under **Databases** , click the database named `  example-db  ` .

4.  Open the **Spanner Studio** , then click add **New tab** or use the editor tab.

5.  Enter a query into the query editor.

6.  Click **Run** .

### gcloud CLI

To submit queries using the [gcloud CLI command-line](/spanner/docs/gcloud-spanner) tool, do the following:

1.  If it's not already installed, [install the gcloud CLI](/sdk/docs/install-sdk) .

2.  In the gcloud CLI, run the following command:
    
    [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql)

For more information, see [Spanner CLI quickstart](/spanner/docs/spanner-cli) .

### REST API

To submit queries using the REST API, use one of the following commands:

  - [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql)
  - [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql)

For more information, see [Query data using the REST API](/spanner/docs/getting-started/rest#query_data_using_sql) and [Get started with Spanner using REST](/spanner/docs/getting-started/rest) .

### RPC API

To submit queries using the RPC API, use one of the following commands:

  - [`  ExecuteSql  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql)
  - [`  ExecuteStreamingSql  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql)

### Client libraries

To learn more about how to run a query with a Spanner client library, see the following:

  - [Query using the Spanner C++ client library](/spanner/docs/getting-started/cpp#execute-query-client-library-C++)
  - [Query using the Spanner C\# client library](/spanner/docs/getting-started/csharp)
  - [Query using the Spanner Go client library](/spanner/docs/getting-started/go#execute-query-client-library-Go)
  - [Query using the Spanner Java client library](/spanner/docs/getting-started/csharp)
  - [Query using the Spanner Node.js client library](/static/spanner/docs/getting-started/nodejs#execute-query-client-library-Node.js)
  - [Query using the Spanner PHP client library](/spanner/docs/getting-started/php#execute-query-client-library-PHP)
  - [Query using the Spanner Python client library](/spanner/docs/getting-started/python#execute-query-client-library-Python)
  - [Query using the Spanner Ruby client library](/spanner/docs/getting-started/ruby#execute-query-client-library-Ruby)

For more information about the Spanner client libraries, see the [Spanner client libraries overview](/spanner/docs/reference/libraries) .

## Visualize Spanner Graph query results

You can view a visual representation of your Spanner Graph query results in Spanner Studio in the Google Cloud console. A query visualization lets you see how the returned elements (nodes and edges) are connected. This can reveal patterns, dependencies, and anomalies that are difficult to see when you view the results in a table. To view a visualization of a query, the query must return full nodes in JSON format. Otherwise, you can see the query results in only tabular format. For more information, see [Use Spanner Graph query visualizations](/spanner/docs/graph/work-with-visualizations) .

## Spanner Graph query structure

A Spanner Graph query consists of several components, such as the property graph name, node and edge patterns, and quantifiers. You use these components to create a query that finds specific patterns in your graph. Each component is described in the [Graph pattern matching](#graph-pattern-matching) section of this document.

The query in Figure 2 demonstrates the basic structure of a Spanner Graph query. The query starts by specifying the target graph, `  FinGraph  ` , using the `  GRAPH  ` clause. The `  MATCH  ` clause then defines the pattern to search for. In this case, it's a `  Person  ` node connected to an `  Account  ` node through an `  Owns  ` edge. The `  RETURN  ` clause specifies which properties of the matched nodes to return.

**Figure 2.** : An example of the structure of a Spanner Graph query.

## Graph pattern matching

Graph pattern matching finds specific patterns within your graph. The most basic patterns are element patterns, such as node patterns that match nodes and edge patterns that match edges.

### Node patterns

A node pattern matches nodes in your graph. This pattern contains matching parentheses, which might optionally include a graph pattern variable, a label expression, and property filters.

#### Find all nodes

The following query returns all nodes in the graph. The variable `  n  ` , a graph pattern variable, binds to the matching nodes. In this case, the node pattern matches all nodes in the graph.

``` text
GRAPH FinGraph
MATCH (n)
RETURN LABELS(n) AS label, n.id;
```

This query returns `  label  ` and `  id  ` :

<table>
<thead>
<tr class="header">
<th>label</th>
<th>id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Account</td>
<td>7</td>
</tr>
<tr class="even">
<td>Account</td>
<td>16</td>
</tr>
<tr class="odd">
<td>Account</td>
<td>20</td>
</tr>
<tr class="even">
<td>Person</td>
<td>1</td>
</tr>
<tr class="odd">
<td>Person</td>
<td>2</td>
</tr>
<tr class="even">
<td>Person</td>
<td>3</td>
</tr>
</tbody>
</table>

#### Find all nodes with a specific label

The following query matches all nodes in the graph that have the `  Person  ` [label](/spanner/docs/reference/standard-sql/graph-patterns#label_expression_definition) . The query returns the `  label  ` and the `  id  ` , `  name  ` properties of the matched nodes.

``` text
GRAPH FinGraph
MATCH (p:Person)
RETURN LABELS(p) AS label, p.id, p.name;
```

This query returns the following properties of the matched nodes:

<table>
<thead>
<tr class="header">
<th>label</th>
<th>id</th>
<th>name</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Person</td>
<td>1</td>
<td>Alex</td>
</tr>
<tr class="even">
<td>Person</td>
<td>2</td>
<td>Dana</td>
</tr>
<tr class="odd">
<td>Person</td>
<td>3</td>
<td>Lee</td>
</tr>
</tbody>
</table>

#### Find all nodes matching a label expression

You can create a [label expression](/spanner/docs/reference/standard-sql/graph-patterns#label_expression_definition) with one or more logical operators. For example, the following query matches all nodes in the graph that have either the `  Person  ` or `  Account  ` label. The graph pattern variable `  n  ` exposes all [properties](/spanner/docs/reference/standard-sql/graph-schema-statements#element_table_property_definition) from nodes with the `  Person  ` or `  Account  ` label.

``` text
GRAPH FinGraph
MATCH (n:Person|Account)
RETURN LABELS(n) AS label, n.id, n.birthday, n.create_time;
```

In the following results of this query:

  - All nodes have the `  id  ` property.
  - Nodes matching the `  Account  ` label have the `  create_time  ` property, but don't have the `  birthday  ` property. The `  birthday  ` property is `  NULL  ` for these nodes.
  - Nodes matching the `  Person  ` label have the `  birthday  ` property, but don't have the `  create_time  ` property. The `  create_time  ` property is `  NULL  ` for these nodes.

<table>
<thead>
<tr class="header">
<th>label</th>
<th>id</th>
<th>birthday</th>
<th>create_time</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Account</td>
<td>7</td>
<td>NULL</td>
<td>2020-01-10T14:22:20.222Z</td>
</tr>
<tr class="even">
<td>Account</td>
<td>16</td>
<td>NULL</td>
<td>2020-01-28T01:55:09.206Z</td>
</tr>
<tr class="odd">
<td>Account</td>
<td>20</td>
<td>NULL</td>
<td>2020-02-18T13:44:20.655Z</td>
</tr>
<tr class="even">
<td>Person</td>
<td>1</td>
<td>1991-12-21T08:00:00Z</td>
<td>NULL</td>
</tr>
<tr class="odd">
<td>Person</td>
<td>2</td>
<td>1980-10-31T08:00:00Z</td>
<td>NULL</td>
</tr>
<tr class="even">
<td>Person</td>
<td>3</td>
<td>1986-12-07T08:00:00Z</td>
<td>NULL</td>
</tr>
</tbody>
</table>

**Note:** `  NULL  ` doesn't necessarily indicate the absence of a property, because an element can expose a property with a `  NULL  ` value. To check the existence of a property, use the [`  PROPERTY_EXISTS  ` predicate](/spanner/docs/reference/standard-sql/operators#property_exists_predicate) .

#### Find all nodes matching the label expression and property filter

This query matches all nodes in the graph that have the `  Person  ` label and where the property `  id  ` is equal to `  1  ` .

``` text
GRAPH FinGraph
MATCH (p:Person {id: 1})
RETURN LABELS(p) AS label, p.id, p.name, p.birthday;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>label</th>
<th>id</th>
<th>name</th>
<th>birthday</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Person</td>
<td>1</td>
<td>Alex</td>
<td>1991-12-21T08:00:00Z</td>
</tr>
</tbody>
</table>

You can use the `  WHERE  ` clause to form more complex filtering conditions on labels and properties.

The following query uses the `  WHERE  ` clause to form a more complex filtering condition on properties. It matches all nodes in the graph that have the `  Person  ` label, and the property `  birthday  ` is before `  1990-01-10  ` .

``` text
GRAPH FinGraph
MATCH (p:Person WHERE p.birthday < '1990-01-10')
RETURN LABELS(p) AS label, p.name, p.birthday;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>label</th>
<th>name</th>
<th>birthday</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Person</td>
<td>Dana</td>
<td>1980-10-31T08:00:00Z</td>
</tr>
<tr class="even">
<td>Person</td>
<td>Lee</td>
<td>1986-12-07T08:00:00Z</td>
</tr>
</tbody>
</table>

### Edge patterns

An edge pattern matches edges or relationships between nodes. Edge patterns are enclosed in square brackets ( `  []  ` ) and include symbols such as `  -  ` , `  ->  ` , or `  <-  ` to indicate directions. An edge pattern might optionally include a graph pattern variable to bind to matching edges.

#### Find all edges with matching labels

This query returns all edges in the graph with the `  Transfers  ` label. The query binds the graph pattern variable `  e  ` to the matching edges.

``` text
GRAPH FinGraph
MATCH -[e:Transfers]->
RETURN e.Id as src_account, e.order_number
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account</th>
<th>order_number</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>304330008004315</td>
</tr>
<tr class="even">
<td>7</td>
<td>304120005529714</td>
</tr>
<tr class="odd">
<td>16</td>
<td>103650009791820</td>
</tr>
<tr class="even">
<td>20</td>
<td>304120005529714</td>
</tr>
<tr class="odd">
<td>20</td>
<td>302290001255747</td>
</tr>
</tbody>
</table>

#### Find all edges matching the label expression and property filter

This query's edge pattern uses a label expression and a property filter to find all edges labeled with `  Transfers  ` that match a specified `  order_number  ` .

``` text
GRAPH FinGraph
MATCH -[e:Transfers {order_number: "304120005529714"}]->
RETURN e.Id AS src_account, e.order_number
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account</th>
<th>order_number</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>304120005529714</td>
</tr>
<tr class="even">
<td>20</td>
<td>304120005529714</td>
</tr>
</tbody>
</table>

#### Find all edges using any direction edge pattern

You can use the `  any direction  ` edge pattern ( `  -[]-  ` ) in a query to match edges in either direction. The following query finds all transfers with a blocked account.

``` text
GRAPH FinGraph
MATCH (account:Account)-[transfer:Transfers]-(:Account {is_blocked:true})
RETURN transfer.order_number, transfer.amount;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>order_number</th>
<th>amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>304330008004315</td>
<td>300</td>
</tr>
<tr class="even">
<td>304120005529714</td>
<td>100</td>
</tr>
<tr class="odd">
<td>103650009791820</td>
<td>300</td>
</tr>
<tr class="even">
<td>302290001255747</td>
<td>200</td>
</tr>
</tbody>
</table>

### Path patterns

A path pattern is built from alternating node and edge patterns.

#### Find all paths from a specific node using a path pattern

The following query finds all transfers to an account initiated from an account owned by `  Person  ` with `  id  ` equal to `  2  ` .

Each matched result represents a path from `  Person  ` `  {id: 2}  ` through a connected `  Account  ` using the `  Owns  ` edge, into another `  Account  ` using the `  Transfers  ` edge.

``` text
GRAPH FinGraph
MATCH
  (p:Person {id: 2})-[:Owns]->(account:Account)-[t:Transfers]->
  (to_account:Account)
RETURN
  p.id AS sender_id, account.id AS from_id, to_account.id AS to_id;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>sender_id</th>
<th>from_id</th>
<th>to_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>2</td>
<td>20</td>
<td>7</td>
</tr>
<tr class="even">
<td>2</td>
<td>20</td>
<td>16</td>
</tr>
</tbody>
</table>

### Quantified path patterns

A quantified pattern repeats a pattern within a specified range.

#### Match a quantified edge pattern

To find paths of a variable length, you can apply a quantifier to an edge pattern. The following query demonstrates this by finding destination accounts that are one to three transfers away from a source `  Account  ` with an `  id  ` of `  7  ` .

The query applies the quantifier `  {1, 3}  ` to the edge pattern `  -[e:Transfers]->  ` . This instructs the query to match paths that repeat the `  Transfers  ` edge pattern one, two, or three times. The `  WHERE  ` clause is used to exclude the source account from the results. The `  ARRAY_LENGTH  ` function is used to access the [`  group variable  `](#group-variables) `  e  ` . For more information, see [access group variable](#access-group-variable) .

``` text
GRAPH FinGraph
MATCH (src:Account {id: 7})-[e:Transfers]->{1, 3}(dst:Account)
WHERE src != dst
RETURN src.id AS src_account_id, ARRAY_LENGTH(e) AS path_length, dst.id AS dst_account_id;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account_id</th>
<th>path_length</th>
<th>dst_account_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>1</td>
<td>16</td>
</tr>
<tr class="even">
<td>7</td>
<td>1</td>
<td>16</td>
</tr>
<tr class="odd">
<td>7</td>
<td>3</td>
<td>16</td>
</tr>
<tr class="even">
<td>7</td>
<td>3</td>
<td>16</td>
</tr>
<tr class="odd">
<td>7</td>
<td>2</td>
<td>20</td>
</tr>
<tr class="even">
<td>7</td>
<td>2</td>
<td>20</td>
</tr>
</tbody>
</table>

Some rows in the results are repeated. This is because multiple paths that match the pattern can exist between the same source and destination nodes, and the query returns all of them.

#### Match a quantified path pattern

The following query finds paths between `  Account  ` nodes with one to two `  Transfers  ` edges through intermediate accounts that are blocked.

The parenthesized path pattern is quantified, and its `  WHERE  ` clause specifies conditions for the repeated pattern.

``` text
GRAPH FinGraph
MATCH
  (src:Account)
  ((a:Account)-[:Transfers]->(b:Account {is_blocked:true}) WHERE a != b){1,2}
    -[:Transfers]->(dst:Account)
RETURN src.id AS src_account_id, dst.id AS dst_account_id;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account_id</th>
<th>dst_account_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>20</td>
</tr>
<tr class="even">
<td>7</td>
<td>20</td>
</tr>
<tr class="odd">
<td>20</td>
<td>20</td>
</tr>
</tbody>
</table>

### Group variables

A graph pattern variable declared in a quantified pattern becomes a *group variable* when accessed outside that pattern. It then binds to an array of matched graph elements.

You can access a group variable as an array. Its graph elements are preserved in the order of their appearance along the matched paths. You can aggregate a group variable using [horizontal aggregation](/spanner/docs/reference/standard-sql/graph-gql-functions#gql-horiz-agg-func-calls) .

#### Access group variable

In the following example, the variable `  e  ` is accessed as follows:

  - A graph pattern variable bound to a single edge in the `  WHERE  ` clause `  e.amount > 100  ` when it's within the quantified pattern.
  - A group variable bound to an array of edge elements in `  ARRAY_LENGTH(e)  ` in the `  RETURN  ` statement when it's outside the quantified pattern.
  - A group variable bound to an array of edge elements, which is aggregated by `  SUM(e.amount)  ` outside the quantified pattern. This is an example of [horizontal aggregation](/spanner/docs/reference/standard-sql/graph-gql-functions#gql-horiz-agg-func-calls) .

<!-- end list -->

``` text
GRAPH FinGraph
MATCH
  (src:Account {id: 7})-[e:Transfers WHERE e.amount > 100]->{0,2}
  (dst:Account)
WHERE src.id != dst.id
LET total_amount = SUM(e.amount)
RETURN
  src.id AS src_account_id, ARRAY_LENGTH(e) AS path_length,
  total_amount, dst.id AS dst_account_id;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account_id</th>
<th>path_length</th>
<th>total_amount</th>
<th>dst_account_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>1</td>
<td>300</td>
<td>16</td>
</tr>
<tr class="even">
<td>7</td>
<td>2</td>
<td>600</td>
<td>20</td>
</tr>
</tbody>
</table>

### Path search prefixes

To limit matched paths within groups that share source and destination nodes, you can use the `  ANY  ` , `  ANY SHORTEST  ` , or `  ANY CHEAPEST  ` path [search prefix](/spanner/docs/reference/standard-sql/graph-patterns#search_prefix) . You can only apply these prefixes before an entire path pattern, and you can't apply them inside parentheses.

#### Match using `     ANY    `

The following query finds all reachable unique accounts that are one or two `  Transfers  ` away from a given `  Account  ` node.

The `  ANY  ` path search prefix ensures that the query returns only one path between a unique pair of `  src  ` and `  dst  ` `  Account  ` nodes. In the following example, although you can reach the `  Account  ` node with `  {id: 16}  ` in two different paths from the source `  Account  ` node, the query returns only one path.

``` text
GRAPH FinGraph
MATCH ANY (src:Account {id: 7})-[e:Transfers]->{1,2}(dst:Account)
LET ids_in_path = ARRAY_CONCAT(ARRAY_AGG(e.Id), [dst.Id])
RETURN src.id AS src_account_id, dst.id AS dst_account_id, ids_in_path;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account_id</th>
<th>dst_account_id</th>
<th>ids_in_path</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>16</td>
<td>7,16</td>
</tr>
<tr class="even">
<td>7</td>
<td>20</td>
<td>7,16,20</td>
</tr>
</tbody>
</table>

#### Match using `     ANY SHORTEST    `

The `  ANY SHORTEST  ` path search prefix returns a single path for each pair of source and destination nodes, selected from those with the minimum number of edges.

For example, the following query finds one of the shortest paths between an `  Account  ` node with `  id  ` `  7  ` and an `  Account  ` node with an `  id  ` of `  20  ` . The query considers paths with one to three `  Transfers  ` edges.

``` text
GRAPH FinGraph
MATCH ANY SHORTEST (src:Account {id: 7})-[e:Transfers]->{1, 3}(dst:Account {id: 20})
RETURN src.id AS src_account_id, dst.id AS dst_account_id, ARRAY_LENGTH(e) AS path_length;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account_id</th>
<th>dst_account_id</th>
<th>path_length</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>20</td>
<td>2</td>
</tr>
</tbody>
</table>

#### Match using `     ANY CHEAPEST    `

The `  ANY CHEAPEST  ` path search prefix ensures that for each pair of source and destination accounts, the query returns only one path with the minimum total compute cost.

The following query finds a path with the minimum total compute cost between `  Account  ` nodes. This cost is based on the sum of the `  amount  ` property of the `  Transfers  ` edges. The search considers paths with one to three `  Transfers  ` edges.

``` text
GRAPH FinGraph
MATCH ANY CHEAPEST (src:Account)-[e:Transfers COST e.amount]->{1,3}(dst:Account)
LET total_cost = sum(e.amount)
RETURN src.id AS src_account_id, dst.id AS dst_account_id, total_cost
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account_id</th>
<th>dst_account_id</th>
<th>total_cost</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>7</td>
<td>900</td>
</tr>
<tr class="even">
<td>7</td>
<td>16</td>
<td>100</td>
</tr>
<tr class="odd">
<td>7</td>
<td>20</td>
<td>400</td>
</tr>
<tr class="even">
<td>16</td>
<td>7</td>
<td>800</td>
</tr>
<tr class="odd">
<td>16</td>
<td>16</td>
<td>500</td>
</tr>
<tr class="even">
<td>16</td>
<td>20</td>
<td>300</td>
</tr>
<tr class="odd">
<td>20</td>
<td>7</td>
<td>500</td>
</tr>
<tr class="even">
<td>20</td>
<td>16</td>
<td>200</td>
</tr>
<tr class="odd">
<td>20</td>
<td>20</td>
<td>500</td>
</tr>
</tbody>
</table>

### Graph patterns

A graph pattern consists of one or more path patterns, separated by a comma ( `  ,  ` ). Graph patterns can contain a `  WHERE  ` clause, which lets you access all the graph pattern variables in the path patterns to form filtering conditions. Each path pattern produces a collection of paths.

#### Match using a graph pattern

The following query identifies intermediary accounts and their owners involved in transactions amounts exceeding 200, through which funds are transferred from a source account to a blocked account.

The following path patterns form the graph pattern:

  - The first pattern finds paths where the transfer occurs from one account to a blocked account using an intermediate account.
  - The second pattern finds paths from an account to its owning person.

The variable `  interm  ` acts as a common link between the two path patterns, which requires `  interm  ` to reference the same element node in both path patterns. This creates an equi-join operation based on the `  interm  ` variable.

**Note:** If there is no shared variable among the path patterns, a [cross join](/spanner/docs/reference/standard-sql/query-syntax#cross_join) is performed between the collection of matches for each path pattern.

``` text
GRAPH FinGraph
MATCH
  (src:Account)-[t1:Transfers]->(interm:Account)-[t2:Transfers]->(dst:Account),
  (interm)<-[:Owns]-(p:Person)
WHERE dst.is_blocked = TRUE AND t1.amount > 200 AND t2.amount > 200
RETURN
  src.id AS src_account_id, dst.id AS dst_account_id,
  interm.id AS interm_account_id, p.id AS owner_id;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>src_account_id</th>
<th>dst_account_id</th>
<th>interm_account_id</th>
<th>owner_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>20</td>
<td>16</td>
<td>7</td>
<td>1</td>
</tr>
</tbody>
</table>

### Linear query statements

You can chain multiple graph statements together to form a linear query statement. The statements are executed in the same order as they appear in the query.

  - Each statement takes the output from the previous statement as input. The input is empty for the first statement.

  - The output of the last statement is the final result.

For example, you can use linear query statements to find the maximum transfer to a blocked account. The following query finds the account and its owner with the largest outgoing transfer to a blocked account.

``` text
GRAPH FinGraph
MATCH (src_account:Account)-[transfer:Transfers]->(dst_account:Account {is_blocked:true})
ORDER BY transfer.amount DESC
LIMIT 1
MATCH (src_account:Account)<-[owns:Owns]-(owner:Person)
RETURN src_account.id AS account_id, owner.name AS owner_name;
```

The following table illustrates this process by showing the intermediate results passed between each statement. For brevity, only some properties are shown.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Statement</th>
<th>Intermediate result (abbreviated)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>MATCH
  (src_account:Account)
    -[transfer:Transfers]-&gt;
  (dst_account:Account {is_blocked:true})</code></pre></td>
<td><table>
<thead>
<tr class="header">
<th><strong>src_account</strong></th>
<th><strong>transfer</strong></th>
<th><strong>dst_account</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">           {id: 7}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 300.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">           {id: 7}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 100.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">           {id: 20}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 200.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="even">
<td><pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>ORDER BY transfer.amount DESC</code></pre></td>
<td><table>
<thead>
<tr class="header">
<th><strong>src_account</strong></th>
<th><strong>transfer</strong></th>
<th><strong>dst_account</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">           {id: 7}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 300.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">           {id: 20}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 200.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">           {id: 7}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 100.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>LIMIT 1</code></pre></td>
<td><table>
<thead>
<tr class="header">
<th><strong>src_account</strong></th>
<th><strong>transfer</strong></th>
<th><strong>dst_account</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">           {id: 7}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 300.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="even">
<td><pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>MATCH
  (src_account:Account)
    &lt;-[owns:Owns]-
  (owner:Person)</code></pre></td>
<td><table>
<thead>
<tr class="header">
<th><strong>src_account</strong></th>
<th><strong>transfer</strong></th>
<th><strong>dst_account</strong></th>
<th><strong>owns</strong></th>
<th><strong>owner</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">           {id: 7}          </code></td>
<td><code dir="ltr" translate="no">           {amount: 300.0}          </code></td>
<td><code dir="ltr" translate="no">           {id: 16, is_blocked: true}          </code></td>
<td><code dir="ltr" translate="no">           {person_id: 1, account_id: 7}          </code></td>
<td><code dir="ltr" translate="no">           {id: 1, name: Alex}          </code></td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>RETURN
  src_account.id AS account_id,
  owner.name AS owner_name
        </code></pre></td>
<td><table>
<thead>
<tr class="header">
<th><strong>account_id</strong></th>
<th><strong>owner_name</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">           7          </code></td>
<td><code dir="ltr" translate="no">           Alex          </code></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

Here are the query results:

<table>
<thead>
<tr class="header">
<th>account_id</th>
<th>owner_name</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       7      </code></td>
<td><code dir="ltr" translate="no">       Alex      </code></td>
</tr>
</tbody>
</table>

### Return statement

The [`  RETURN  ` statement](/spanner/docs/reference/standard-sql/graph-query-statements#gql_return) specifies what to return from the matched patterns. It can access graph pattern variables and include expressions and other clauses, such as `  ORDER BY  ` and `  GROUP BY  ` .

Spanner Graph doesn't support returning graph elements as query results. To return the entire graph element, use the [`  TO_JSON  ` function](/spanner/docs/reference/standard-sql/json_functions#to_json) or [`  SAFE_TO_JSON  ` function](/spanner/docs/reference/standard-sql/json_functions#safe_to_json) . Of these two functions, we recommend that you use `  SAFE_TO_JSON  ` .

#### Return graph elements as JSON

``` text
GRAPH FinGraph
MATCH (n:Account {id: 7})
-- Returning a graph element in the final results is NOT allowed. Instead, use
-- the TO_JSON function or explicitly return the graph element's properties.
RETURN TO_JSON(n) AS n;
```

``` text
GRAPH FinGraph
MATCH (n:Account {id: 7})
-- Certain fields in the graph elements, such as TOKENLIST, can't be returned
-- in the TO_JSON function. In those cases, use the SAFE_TO_JSON function instead.
RETURN SAFE_TO_JSON(n) AS n;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>n</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       {"identifier":"mUZpbkdyYXBoLkFjY291bnQAeJEO","kind":"node","labels":["Account"],"properties":{"create_time":"2020-01-10T14:22:20.222Z","id":7,"is_blocked":false,"nick_name":"Vacation Fund"}}      </code></td>
</tr>
</tbody>
</table>

## Composing larger queries with NEXT keyword

You can chain multiple graph linear query statements using the `  NEXT  ` keyword. The first statement receives an empty input, and the output of each subsequent statement becomes the input for the next.

The following example finds the owner of the account with the most incoming transfers by chaining multiple graph linear statements. You can use the same variable, for example, `  account  ` , to refer to the same graph element across multiple linear statements.

``` text
GRAPH FinGraph
MATCH (:Account)-[:Transfers]->(account:Account)
RETURN account, COUNT(*) AS num_incoming_transfers
GROUP BY account
ORDER BY num_incoming_transfers DESC
LIMIT 1

NEXT

MATCH (account:Account)<-[:Owns]-(owner:Person)
RETURN account.id AS account_id, owner.name AS owner_name, num_incoming_transfers;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>account_id</th>
<th>owner_name</th>
<th>num_incoming_transfers</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       16      </code></td>
<td><code dir="ltr" translate="no">       Lee      </code></td>
<td><code dir="ltr" translate="no">       3      </code></td>
</tr>
</tbody>
</table>

## Functions and expressions

You can use all GoogleSQL [functions](/spanner/docs/reference/standard-sql/functions-all) (both aggregate and scalar functions), [operators](/spanner/docs/reference/standard-sql/operators) , and [conditional expressions](/spanner/docs/reference/standard-sql/conditional_expressions) in Spanner Graph queries. Spanner Graph also supports graph-specific functions and operators.

### Built-in functions and operators

The following [functions](/spanner/docs/reference/standard-sql/graph-gql-functions) and [operators](/spanner/docs/reference/standard-sql/graph-operators) are used in GQL:

  - `  PROPERTY_EXISTS(n, birthday)  ` : Returns whether `  n  ` has the `  birthday  ` property.
  - `  LABELS(n)  ` : Returns the labels of `  n  ` as defined in the graph schema.
  - `  PROPERTY_NAMES(n)  ` : Returns the property names of `  n  ` .
  - `  TO_JSON(n)  ` : Returns `  n  ` in JSON format. For more information, see the [`  TO_JSON  ` function](/spanner/docs/reference/standard-sql/json_functions#to_json) .

the `  PROPERTY_EXISTS  ` predicate, `  LABELS  ` function, and `  TO_JSON  ` function, as well as other built-in functions like `  ARRAY_AGG  ` and `  CONCAT  ` .

``` text
GRAPH FinGraph
MATCH (person:Person)-[:Owns]->(account:Account)
RETURN person, ARRAY_AGG(account.nick_name) AS accounts
GROUP BY person

NEXT

RETURN
  LABELS(person) AS labels,
  TO_JSON(person) AS person,
  accounts,
  CONCAT(person.city, ", ", person.country) AS location,
  PROPERTY_EXISTS(person, is_blocked) AS is_blocked_property_exists,
  PROPERTY_EXISTS(person, name) AS name_property_exists
LIMIT 1;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>is_blocked_property_exists</th>
<th>name_property_exists</th>
<th>labels</th>
<th>accounts</th>
<th>location</th>
<th>person</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       false      </code></td>
<td><code dir="ltr" translate="no">       true      </code></td>
<td><code dir="ltr" translate="no">       Person      </code></td>
<td><code dir="ltr" translate="no">       ["Vacation Fund"]      </code></td>
<td><code dir="ltr" translate="no">       Adelaide, Australia      </code></td>
<td><code dir="ltr" translate="no">       {"identifier":"mUZpbkdyYXBoLlBlcnNvbgB4kQI=","kind":"node","labels":["Person"],"properties":{"birthday":"1991-12-21T08:00:00Z","city":"Adelaide","country":"Australia","id":1,"name":"Alex"}}      </code></td>
</tr>
</tbody>
</table>

## Subqueries

A *subquery* is a query nested in another query. The following lists Spanner Graph subquery rules:

  - A subquery is enclosed within a pair of braces `  {}  ` .
  - A subquery might start with the leading `  GRAPH  ` clause to specify the graph in scope. The specified graph doesn't need to be the same as the one used in the outer query.
  - When the `  GRAPH  ` clause is omitted in the subquery, the following occurs:
      - The graph in scope is inferred from the closest outer query context.
      - The subquery must start from a graph pattern matching statement with `  MATCH  ` .
  - A graph pattern variable declared outside the subquery scope can't be declared again inside the subquery, but it can be referred to in expressions or functions inside the subquery.

#### Use a subquery to find the total number of transfers from each account

The following query illustrates the use of the `  VALUE  ` subquery. The subquery is enclosed in braces `  {}  ` prefixed by the `  VALUE  ` keyword. The query returns the total number of transfers initiated from an account.

``` text
GRAPH FinGraph
MATCH (p:Person)-[:Owns]->(account:Account)
RETURN p.name, account.id AS account_id, VALUE {
  MATCH (a:Account)-[transfer:Transfers]->(:Account)
  WHERE a = account
  RETURN COUNT(transfer) AS num_transfers
} AS num_transfers;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>name</th>
<th>account_id</th>
<th>num_transfers</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       Alex      </code></td>
<td><code dir="ltr" translate="no">       7      </code></td>
<td><code dir="ltr" translate="no">       2      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Dana      </code></td>
<td><code dir="ltr" translate="no">       20      </code></td>
<td><code dir="ltr" translate="no">       2      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Lee      </code></td>
<td><code dir="ltr" translate="no">       16      </code></td>
<td><code dir="ltr" translate="no">       1      </code></td>
</tr>
</tbody>
</table>

For a list of supported subquery expressions, see [Spanner Graph subqueries](/spanner/docs/reference/standard-sql/graph-subqueries) .

#### Use a subquery to find accounts owned by each person

The following query uses the `  CALL  ` statement with an inline subquery. The `  MATCH (p:Person)  ` statement creates a table with a single column named `  p  ` . Each row in this table contains a `  Person  ` node. The `  CALL (p)  ` statement executes the enclosed subquery for each row in this working table. The subquery finds accounts owned by each matched person `  p  ` . Multiple accounts for the same person are ordered by account ID.

The example declares the outer-scoped node variable `  p  ` from the `  MATCH (p:Person)  ` clause. The `  CALL (p)  ` statement references this variable. This declaration lets you redeclare or *multiply-declare* the node variable in a path pattern of the subquery. This ensures that the inner and outer `  p  ` node variables bind to the same `  Person  ` node in the graph. If the `  CALL  ` statement doesn't declare the node variable `  p  ` , the subquery treats the redeclared variable `  p  ` as a new variable. This new variable is independent of the outer-scoped variable, and the subquery doesn't *multiply-declare* it because it returns different results. For more information, see [`  CALL  `](/spanner/docs/reference/standard-sql/graph-query-statements#gql_call) statement.

``` text
GRAPH FinGraph
MATCH (p:Person)
CALL (p) {
  MATCH (p)-[:Owns]->(a:Account)
  RETURN a.Id AS account_Id
  ORDER BY account_Id
}
RETURN p.name AS person_name, account_Id
ORDER BY person_name, account_Id;
```

**Result**

<table>
<thead>
<tr class="header">
<th>person_name</th>
<th>account_Id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Alex</td>
<td>7</td>
</tr>
<tr class="even">
<td>Dana</td>
<td>20</td>
</tr>
<tr class="odd">
<td>Lee</td>
<td>16</td>
</tr>
</tbody>
</table>

## Query parameters

You can query Spanner Graph with parameters. For more information, see the [syntax](/spanner/docs/reference/standard-sql/lexical#query_parameters) and learn how to [query data with parameters](/spanner/docs/samples/spanner-query-with-parameter) in the Spanner client libraries.

The following query illustrates the use of query parameters.

``` text
GRAPH FinGraph
MATCH (person:Person {id: @id})
RETURN person.name;
```

## Query graphs and tables together

You can use Graph queries in conjunction with SQL to access information from your Graphs and Tables together in a single statement.

The `  GRAPH_TABLE  ` operator takes a linear graph query and returns its result in a tabular form that can be integrated into a SQL query. This interoperability lets you enrich graph query results with non-graph content and the other way around.

For example, you can create a `  CreditReports  ` table and insert a few credit reports, as shown in the following example:

``` text
CREATE TABLE CreditReports (
  person_id     INT64 NOT NULL,
  create_time   TIMESTAMP NOT NULL,
  score         INT64 NOT NULL,
) PRIMARY KEY (person_id, create_time);
```

``` text
INSERT INTO CreditReports (person_id, create_time, score)
VALUES
  (1,"2020-01-10 06:22:20.222", 700),
  (2,"2020-02-10 06:22:20.222", 800),
  (3,"2020-03-10 06:22:20.222", 750);
```

Next, you can identify specific persons through graph pattern matching in `  GRAPH_TABLE  ` and join the graph query results with the `  CreditReports  ` table to retrieve credit scores.

``` text
SELECT
  gt.person.id,
  credit.score AS latest_credit_score
FROM GRAPH_TABLE(
  FinGraph
  MATCH (person:Person)-[:Owns]->(:Account)-[:Transfers]->(account:Account {is_blocked:true})
  RETURN DISTINCT person
) AS gt
JOIN CreditReports AS credit
  ON gt.person.id = credit.person_id
ORDER BY credit.create_time;
```

Here are the query results:

<table>
<thead>
<tr class="header">
<th>person_id</th>
<th>latest_credit_score</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       1      </code></td>
<td><code dir="ltr" translate="no">       700      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2      </code></td>
<td><code dir="ltr" translate="no">       800      </code></td>
</tr>
</tbody>
</table>

## What's next

Learn [best practices for tuning queries](/spanner/docs/graph/best-practices-tuning-queries) .
