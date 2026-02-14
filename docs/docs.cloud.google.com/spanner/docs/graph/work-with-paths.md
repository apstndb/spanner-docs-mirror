**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to work with graph paths in Spanner Graph.

In graph databases, the graph path data type represents a sequence of nodes interleaved with edges and shows how these nodes and edges are related. To learn more about the path data type, see [Graph path type](/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type) .

With the [Spanner Graph Language (GQL)](/spanner/docs/reference/standard-sql/graph-intro) , you can construct graph paths and perform queries on them. The examples in this document use the same Spanner Graph schema as found on the [Set up and query Spanner Graph](/spanner/docs/graph/set-up) page.

## Construct a graph path

You can construct a graph path by creating a path variable in a graph pattern or with the [`  PATH  ` function](/spanner/docs/reference/standard-sql/graph-gql-functions#path) .

We recommend constructing a graph path by using the path variable. The format to create a path variable is:

``` text
MATCH p = PATH_PATTERN
```

For more information, see [Graph pattern](/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition) .

#### Example

In the following example, the query finds patterns of money transfers between accounts within `  FinGraph  ` .

``` text
GRAPH FinGraph
MATCH p = (src:Account {id: 16})-[t:Transfers]->{2}(dst:Account {id: 7})
RETURN TO_JSON(p) AS full_path;
```

#### Result

<table>
<thead>
<tr class="header">
<th>full_path</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>[{"identifier": ..., "properties": {"id": 16, ...}, ...}, {"identifier": ..., "properties": {"amount": 300.0, ...}, ...}, ...]</td>
</tr>
</tbody>
</table>

The result indicates that the query found the `  Account -> Transfers -> Account  ` pattern in the database.

## Query a graph path

You can use the following path-specific functions to query a graph path. For more general information about Spanner Graph queries, see [Queries overview](/spanner/docs/graph/queries-overview) .

### `     EDGES    `

The `  EDGES  ` function returns all the edges in a graph path. For detailed semantics, see [`  EDGES  `](/spanner/docs/reference/standard-sql/graph-gql-functions#edges) .

#### Example

This query finds a path between two accounts that pass through a middle account. It returns the amount of the second `  Transfers  ` edge in the path which might be between `  src  ` and `  mid  ` or between `  mid  ` and `  dst  ` .

``` text
GRAPH FinGraph
MATCH p = (src:Account {id: 7})-[t1:Transfers]->{1,3}(mid:Account)-[t2:Transfers]->
  {1,3}(dst:Account {id: 16})
LET second_edge = EDGES(p)[1]
RETURN DISTINCT src.id AS src, dst.id AS dst, second_edge.amount AS second_edge_amount;
```

#### Result

<table>
<thead>
<tr class="header">
<th>src</th>
<th>dst</th>
<th>second_edge_amount</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>16</td>
<td>300</td>
</tr>
</tbody>
</table>

### `     NODES    `

The `  NODES  ` function returns all the nodes in a graph path. For detailed semantics, see [`  NODES  `](/spanner/docs/reference/standard-sql/graph-gql-functions#nodes) .

#### Example

This query finds the graph path of two transfers, and then returns a JSON list representing the path.

``` text
GRAPH FinGraph
MATCH p = (src:Account)-[t:Transfers]->{2}(dst:Account)
RETURN TO_JSON(NODES(p)) AS nodes;
```

#### Result

<table>
<thead>
<tr class="header">
<th>nodes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>[{"identifier": "...", "properties": {"id": 16}, ...}, {"identifier": "...", "properties": {"id": 20, ...}, ...]</td>
</tr>
<tr class="even">
<td>...</td>
</tr>
</tbody>
</table>

### `     PATH_FIRST    `

The `  PATH_FIRST  ` function finds the first node in a graph path. For detailed semantics, see [`  PATH_FIRST  `](/spanner/docs/reference/standard-sql/graph-gql-functions#path_first) .

#### Example

This query finds the first node in a graph path of two transfers. It returns the label of the `  Account  ` node and the account's nickname.

``` text
GRAPH FinGraph
MATCH p = -[:Transfers]->{1,3}(dst:Account{id: 7})
RETURN DISTINCT PATH_FIRST(p).id AS can_reach_target;
```

#### Result

<table>
<thead>
<tr class="header">
<th>can_reach_target</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
</tr>
<tr class="even">
<td>16</td>
</tr>
<tr class="odd">
<td>20</td>
</tr>
</tbody>
</table>

### `     PATH_LAST    `

The `  PATH_LAST  ` function finds the last node in a graph path. For detailed semantics, see [`  PATH_LAST  `](/spanner/docs/reference/standard-sql/graph-gql-functions#path_last) .

#### Example

This query finds the last node in a graph path of two transfers. It returns the label of the `  Account  ` node and the account's nickname.

``` text
GRAPH FinGraph
MATCH p =(start:Account{id: 7})-[:Transfers]->{1,3}
RETURN DISTINCT PATH_LAST(p).id as can_reach_target;
```

#### Result

<table>
<thead>
<tr class="header">
<th>can_reach_target</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
</tr>
<tr class="even">
<td>16</td>
</tr>
<tr class="odd">
<td>20</td>
</tr>
</tbody>
</table>

### `     PATH_LENGTH    `

The `  PATH_LENGTH  ` function finds the number of edges in a graph path. For detailed semantics, see [`  PATH_LENGTH  `](/spanner/docs/reference/standard-sql/graph-gql-functions#path_length) .

#### Example

This query finds the number of edges in a graph path that contains one to three transfers.

``` text
GRAPH FinGraph
MATCH p = (src:Account)-[e:Transfers]->{1,3}(dst:Account)
RETURN PATH_LENGTH(p) AS num_transfers, COUNT(*) AS num_paths;
```

#### Result

<table>
<thead>
<tr class="header">
<th>num_transfers</th>
<th>num_paths</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>5</td>
</tr>
<tr class="even">
<td>2</td>
<td>7</td>
</tr>
<tr class="odd">
<td>3</td>
<td>11</td>
</tr>
</tbody>
</table>

### `     IS_ACYCLIC    `

The `  IS_ACYCLIC  ` function checks if a graph path has repeating nodes. It returns `  TRUE  ` if repetition is found, otherwise it returns `  FALSE  ` . For detailed semantics, see [`  IS_ACYCLIC  `](/spanner/docs/reference/standard-sql/graph-gql-functions#is_acyclic) .

#### Example

This query checks if this graph path has repeating nodes.

``` text
GRAPH FinGraph
MATCH p = (src:Account)-[t:Transfers]->{2}(dst:Account)
RETURN IS_ACYCLIC(p) AS is_acyclic_path,
       ARRAY_TRANSFORM(NODES(p), n->n.id) AS account_ids;
```

#### Result

<table>
<thead>
<tr class="header">
<th>is_acyclic_path</th>
<th>account_ids</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>TRUE</td>
<td>16,20,7</td>
</tr>
<tr class="even">
<td>TRUE</td>
<td>20,7,16</td>
</tr>
<tr class="odd">
<td>TRUE</td>
<td>20,7,16</td>
</tr>
<tr class="even">
<td>FALSE</td>
<td>16,20,16</td>
</tr>
<tr class="odd">
<td>TRUE</td>
<td>7,16,20</td>
</tr>
<tr class="even">
<td>TRUE</td>
<td>7,16,20</td>
</tr>
<tr class="odd">
<td>FALSE</td>
<td>20,16,20</td>
</tr>
</tbody>
</table>

### `     IS_TRAIL    `

The `  IS_TRAIL  ` function checks if a graph path has repeating edges. It returns `  TRUE  ` if repetition is found, otherwise it returns `  FALSE  ` . For detailed semantics, see [`  IS_TRAIL  `](/spanner/docs/reference/standard-sql/graph-gql-functions#is_trail) .

#### Example

This query checks if this graph path has repeating edges.

``` text
GRAPH FinGraph
MATCH p = (src:Account)-[t:Transfers]->{3}(dst:Account)
WHERE src.id < dst.id
RETURN IS_TRAIL(p) AS is_trail_path,
       ARRAY_TRANSFORM(t, t->t.id) AS transfer_ids
```

#### Result

<table>
<thead>
<tr class="header">
<th>is_trail_path</th>
<th>transfer_ids</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>FALSE</td>
<td>16,20,16</td>
</tr>
<tr class="even">
<td>TRUE</td>
<td>7,16,20</td>
</tr>
<tr class="odd">
<td>TRUE</td>
<td>7,16,20</td>
</tr>
</tbody>
</table>

## Path modes

In Spanner Graph, the default behavior is to return all paths, including paths with repeating nodes and edges. You can use the following path modes to include or exclude paths that have repeating nodes and edges. For detailed semantics, see the [Path mode](/spanner/docs/reference/standard-sql/graph-patterns#path_mode) documentation.

### `     WALK    `

The `  WALK  ` path mode returns all paths, including paths with repeating nodes and edges. `  WALK  ` is the default path mode.

#### Example

The following query demonstrates the use of the `  WALK  ` path mode on a [quantified path pattern](/spanner/docs/graph/queries-overview#quantified-path-patterns) . The first path in the results have repeated edges.

``` text
GRAPH FinGraph
MATCH p = WALK (src:Account)-[t:Transfers]->{3}(dst:Account)
WHERE src.id < dst.id
RETURN ARRAY_TRANSFORM(t, t->t.id) AS transfer_ids
```

#### Result

<table>
<thead>
<tr class="header">
<th>transfer_ids</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>16,20,16</td>
</tr>
<tr class="even">
<td>7,16,20</td>
</tr>
<tr class="odd">
<td>7,16,20</td>
</tr>
</tbody>
</table>

### `     ACYCLIC    `

The `  ACYCLIC  ` path mode filters out paths that have repeating nodes.

#### Example

The following query demonstrates the use of the `  ACYCLIC  ` path mode on a [quantified path pattern](/spanner/docs/graph/queries-overview#quantified-path-patterns) . The path with equal `  src  ` and `  dst  ` nodes is filtered out.

``` text
GRAPH FinGraph
MATCH p = ACYCLIC (src:Account)-[t:Transfers]->{2}(dst:Account)
RETURN ARRAY_TRANSFORM(NODES(p), n->n.id) AS account_ids
```

#### Result

<table>
<thead>
<tr class="header">
<th>account_ids</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>16,20,7</td>
</tr>
<tr class="even">
<td>20,7,16</td>
</tr>
<tr class="odd">
<td>20,7,16</td>
</tr>
<tr class="even">
<td>7,16,20</td>
</tr>
<tr class="odd">
<td>7,16,20</td>
</tr>
</tbody>
</table>

### `     TRAIL    `

The `  TRAIL  ` path mode filters out paths that have repeating edges.

#### Example

The following query demonstrates the use of the `  TRAIL  ` path mode on a [quantified path pattern](/spanner/docs/graph/queries-overview#quantified-path-patterns) . Paths with repeated edges are filtered out.

``` text
GRAPH FinGraph
MATCH p = TRAIL (src:Account)-[t:Transfers]->{3}(dst:Account)
WHERE src.id < dst.id
RETURN ARRAY_TRANSFORM(t, t->t.id) AS transfer_ids
```

#### Result

<table>
<thead>
<tr class="header">
<th>transfer_ids</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7,16,20</td>
</tr>
<tr class="even">
<td>7,16,20</td>
</tr>
</tbody>
</table>

## Path search prefix

You can use a path search prefix to restrict a path pattern to return the shortest path from each [data partition](/spanner/docs/reference/standard-sql/graph-patterns#details_5) . For detailed semantics, see [Path search prefix](/spanner/docs/reference/standard-sql/graph-patterns#search_prefix) .

### `     ANY SHORTEST    `

The `  ANY SHORTEST  ` path search prefix returns the shortest path (the path with the least number of edges) that matches the pattern from each data partition. If there are more than one shortest paths per partition, returns any one of them.

#### Example

The following query matches any path between each pair of `  [a, b]  ` .

``` text
GRAPH FinGraph
MATCH p = ANY SHORTEST (a:Account {is_blocked:true})-[t:Transfers]->{1,4}(b:Account)
LET total_amount = SUM(t.amount)
RETURN a.id AS account1_id, total_amount, b.id AS account2_id;
```

#### Result

<table>
<thead>
<tr class="header">
<th>account1_id</th>
<th>total_amount</th>
<th>account2_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>16</td>
<td>500</td>
<td>16</td>
</tr>
<tr class="even">
<td>16</td>
<td>800</td>
<td>7</td>
</tr>
<tr class="odd">
<td>16</td>
<td>300</td>
<td>20</td>
</tr>
</tbody>
</table>

### `     ANY CHEAPEST    `

The `  ANY CHEAPEST  ` path search prefix ensures that for each pair of source and destination accounts, the query returns only one path with the minimum total compute cost. A cheapest path is one with the minimum total compute cost, calculated from `  COST  ` expressions on edges in the path. If multiple cheapest paths exist per partition, the query returns any one of them.

#### Example

The following query matches one of the cheapest paths between each pair of accounts, represented by `  a  ` and `  b  ` , based on transfer amount.

``` text
GRAPH FinGraph
MATCH ANY CHEAPEST (a:Account)-[t:Transfers COST t.amount]->{1,3}(b:Account)
LET total_cost = sum(t.amount)
RETURN a.id AS account1_id, b.id AS account2_id, total_cost
```

#### Result

<table>
<thead>
<tr class="header">
<th>account1_id</th>
<th>account2_id</th>
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

## Conversion rules

For more information, see [GRAPH\_PATH conversion rules](/spanner/docs/reference/standard-sql/conversion_rules) .

## Use case example

In the following use case example, you find all accounts have been routed through one to three accounts, from account ID `  20  ` .

``` text
GRAPH FinGraph
MATCH p = (start:Account {id: 20})-[:Transfers]->{1,3}(dst:Account)
RETURN DISTINCT dst.id AS dst;
```

**Result**

<table>
<thead>
<tr class="header">
<th>dst</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
</tr>
<tr class="even">
<td>16</td>
</tr>
<tr class="odd">
<td>20</td>
</tr>
</tbody>
</table>

However, a query that returns to account ID `  20  ` might be an overly broad query because it starts with account ID `  20  ` . To show more specific results, you can enforce your query to show only acyclic graph paths without any repeating nodes. To do so, you can:

  - Use `  MATCH p = ACYCLIC <path_pattern>  ` ; or
  - Apply an `  IS_ACYCLIC(p)  ` filter in your query

The following query uses `  MATCH p = ACYCLIC PATH_PATTERN  ` :

``` text
GRAPH FinGraph
MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->{1,3}(dst:Account)
RETURN DISTINCT dst.id AS dst;
```

**Result**

<table>
<thead>
<tr class="header">
<th>dst</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
</tr>
<tr class="even">
<td>16</td>
</tr>
</tbody>
</table>

If you want to know the first account that money is transferred through, then you can run the following query:

``` text
GRAPH FinGraph
MATCH p = ACYCLIC (start:Account {id: 20})(-[:Transfers]->
  (nexts:Account)){1,3}(dst:Account)
RETURN dst.id AS dst, ARRAY_AGG(DISTINCT nexts[0].id) AS unique_starts;
```

This query is unconventional because it introduces a new variable inside the quantified path using `  nexts  ` to get the result. With path variables, you can simplify the query:

``` text
GRAPH FinGraph
MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->{1,3}(dst:Account)
RETURN dst.id AS dst, ARRAY_AGG(DISTINCT NODES(p)[OFFSET(1)].id) AS unique_starts;
```

Using `  NODES(p)  ` returns all nodes along the path. Because the first node account is specified as `  start  ` , the next one (at the first offset) is the first account that money is transferred through.

**Result**

<table>
<thead>
<tr class="header">
<th>dst</th>
<th>unique_starts</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>16, 7</td>
</tr>
</tbody>
</table>

Paths are more useful when there are multiple quantified paths. You can add a constraint that the paths found from `  start  ` must pass through account ID `  7  ` :

``` text
GRAPH FinGraph
MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->
  {1,3}(mid:Account {id: 7})-[:Transfers]->{1,3}(dst:Account)
RETURN dst.id AS dst,
  ARRAY_AGG(DISTINCT NODES(p)[OFFSET(1)].id) AS unique_starts;
```

Although the `  MATCH  ` statement changed, the rest of the query doesn't need to change. Without using path variables, there are cases where it's not possible for Spanner to statically know which quantified path to inspect.

Using a path variable, you can get the sum of all transfers:

``` text
GRAPH FinGraph
MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->
  {1,3}(mid:Account {id: 7})-[:Transfers]->{1,3}(dst:Account)
LET all_transfers = EDGES(p)
LET transfer_amounts = SUM(all_transfers.amount)
RETURN dst.id AS dst,
  ARRAY_AGG(DISTINCT NODES(p)[OFFSET(1)].id) AS participating_neighbor_nodes, transfer_amounts;
```

**Result**

<table>
<thead>
<tr class="header">
<th>dst</th>
<th>participating_neighbor_nodes</th>
<th>transfer_amounts</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>16</td>
<td>7</td>
<td>600</td>
</tr>
<tr class="even">
<td>16</td>
<td>7</td>
<td>800</td>
</tr>
</tbody>
</table>

## What's next

  - [Graph path type](/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type)
  - [`  PATH  ` function](/spanner/docs/reference/standard-sql/graph-gql-functions#path)
  - [Path mode](/spanner/docs/reference/standard-sql/graph-patterns#path_mode)
  - [Path search prefix](/spanner/docs/reference/standard-sql/graph-patterns#search_prefix)
