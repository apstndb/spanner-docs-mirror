**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This page describes how to work with graph paths in Spanner Graph.

In graph databases, the graph path data type represents a sequence of nodes interleaved with edges and shows how these nodes and edges are related. To learn more about the path data type, see [Graph path type](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type) .

With the [Spanner Graph Language (GQL)](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) , you can construct graph paths and perform queries on them. The examples in this document use the same Spanner Graph schema as found on the [Set up and query Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/set-up) page.

## Construct a graph path

You can construct a graph path by creating a path variable in a graph pattern or with the [`PATH` function](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#path) .

We recommend constructing a graph path by using the path variable. The format to create a path variable is:

    MATCH p = PATH_PATTERN

For more information, see [Graph pattern](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition) .

#### Example

In the following example, the query finds patterns of money transfers between accounts within `FinGraph` .

    GRAPH FinGraph
    MATCH p = (src:Account {id: 16})-[t:Transfers]->{2}(dst:Account {id: 7})
    RETURN TO_JSON(p) AS full_path;

#### Result

| full\_path                                                                                                                       |
| -------------------------------------------------------------------------------------------------------------------------------- |
| \[{"identifier": ..., "properties": {"id": 16, ...}, ...}, {"identifier": ..., "properties": {"amount": 300.0, ...}, ...}, ...\] |

The result indicates that the query found the `Account -> Transfers -> Account` pattern in the database.

## Query a graph path

You can use the following path-specific functions to query a graph path. For more general information about Spanner Graph queries, see [Queries overview](https://docs.cloud.google.com/spanner/docs/graph/queries-overview) .

### `EDGES`

The `EDGES` function returns all the edges in a graph path. For detailed semantics, see [`EDGES`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#edges) .

#### Example

This query finds a path between two accounts that pass through a middle account. It returns the amount of the second `Transfers` edge in the path which might be between `src` and `mid` or between `mid` and `dst` .

    GRAPH FinGraph
    MATCH p = (src:Account {id: 7})-[t1:Transfers]->{1,3}(mid:Account)-[t2:Transfers]->
      {1,3}(dst:Account {id: 16})
    LET second_edge = EDGES(p)[1]
    RETURN DISTINCT src.id AS src, dst.id AS dst, second_edge.amount AS second_edge_amount;

#### Result

| src | dst | second\_edge\_amount |
| --- | --- | -------------------- |
| 7   | 16  | 300                  |

### `NODES`

The `NODES` function returns all the nodes in a graph path. For detailed semantics, see [`NODES`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#nodes) .

#### Example

This query finds the graph path of two transfers, and then returns a JSON list representing the path.

    GRAPH FinGraph
    MATCH p = (src:Account)-[t:Transfers]->{2}(dst:Account)
    RETURN TO_JSON(NODES(p)) AS nodes;

#### Result

| nodes                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------ |
| \[{"identifier": "...", "properties": {"id": 16}, ...}, {"identifier": "...", "properties": {"id": 20, ...}, ...\] |
| ...                                                                                                                |

### `PATH_FIRST`

The `PATH_FIRST` function finds the first node in a graph path. For detailed semantics, see [`PATH_FIRST`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#path_first) .

#### Example

This query finds the first node in a graph path of two transfers. It returns the label of the `Account` node and the account's nickname.

    GRAPH FinGraph
    MATCH p = -[:Transfers]->{1,3}(dst:Account{id: 7})
    RETURN DISTINCT PATH_FIRST(p).id AS can_reach_target;

#### Result

| can\_reach\_target |
| ------------------ |
| 7                  |
| 16                 |
| 20                 |

### `PATH_LAST`

The `PATH_LAST` function finds the last node in a graph path. For detailed semantics, see [`PATH_LAST`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#path_last) .

#### Example

This query finds the last node in a graph path of two transfers. It returns the label of the `Account` node and the account's nickname.

    GRAPH FinGraph
    MATCH p =(start:Account{id: 7})-[:Transfers]->{1,3}
    RETURN DISTINCT PATH_LAST(p).id as can_reach_target;

#### Result

| can\_reach\_target |
| ------------------ |
| 7                  |
| 16                 |
| 20                 |

### `PATH_LENGTH`

The `PATH_LENGTH` function finds the number of edges in a graph path. For detailed semantics, see [`PATH_LENGTH`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#path_length) .

#### Example

This query finds the number of edges in a graph path that contains one to three transfers.

    GRAPH FinGraph
    MATCH p = (src:Account)-[e:Transfers]->{1,3}(dst:Account)
    RETURN PATH_LENGTH(p) AS num_transfers, COUNT(*) AS num_paths;

#### Result

| num\_transfers | num\_paths |
| -------------- | ---------- |
| 1              | 5          |
| 2              | 7          |
| 3              | 11         |

### `IS_ACYCLIC`

The `IS_ACYCLIC` function checks if a graph path has repeating nodes. It returns `TRUE` if repetition is found, otherwise it returns `FALSE` . For detailed semantics, see [`IS_ACYCLIC`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#is_acyclic) .

#### Example

This query checks if this graph path has repeating nodes.

    GRAPH FinGraph
    MATCH p = (src:Account)-[t:Transfers]->{2}(dst:Account)
    RETURN IS_ACYCLIC(p) AS is_acyclic_path,
           ARRAY_TRANSFORM(NODES(p), n->n.id) AS account_ids;

#### Result

| is\_acyclic\_path | account\_ids |
| ----------------- | ------------ |
| TRUE              | 16,20,7      |
| TRUE              | 20,7,16      |
| TRUE              | 20,7,16      |
| FALSE             | 16,20,16     |
| TRUE              | 7,16,20      |
| TRUE              | 7,16,20      |
| FALSE             | 20,16,20     |

### `IS_TRAIL`

The `IS_TRAIL` function checks if a graph path has repeating edges. It returns `TRUE` if repetition is found, otherwise it returns `FALSE` . For detailed semantics, see [`IS_TRAIL`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#is_trail) .

#### Example

This query checks if this graph path has repeating edges.

    GRAPH FinGraph
    MATCH p = (src:Account)-[t:Transfers]->{3}(dst:Account)
    WHERE src.id < dst.id
    RETURN IS_TRAIL(p) AS is_trail_path,
           ARRAY_TRANSFORM(t, t->t.id) AS transfer_ids

#### Result

| is\_trail\_path | transfer\_ids |
| --------------- | ------------- |
| FALSE           | 16,20,16      |
| TRUE            | 7,16,20       |
| TRUE            | 7,16,20       |

## Path modes

In Spanner Graph, the default behavior is to return all paths, including paths with repeating nodes and edges. You can use the following path modes to include or exclude paths that have repeating nodes and edges. For detailed semantics, see the [Path mode](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#path_mode) documentation.

### `WALK`

The `WALK` path mode returns all paths, including paths with repeating nodes and edges. `WALK` is the default path mode.

#### Example

The following query demonstrates the use of the `WALK` path mode on a [quantified path pattern](https://docs.cloud.google.com/spanner/docs/graph/queries-overview#quantified-path-patterns) . The first path in the results have repeated edges.

    GRAPH FinGraph
    MATCH p = WALK (src:Account)-[t:Transfers]->{3}(dst:Account)
    WHERE src.id < dst.id
    RETURN ARRAY_TRANSFORM(t, t->t.id) AS transfer_ids

#### Result

| transfer\_ids |
| ------------- |
| 16,20,16      |
| 7,16,20       |
| 7,16,20       |

### `ACYCLIC`

The `ACYCLIC` path mode filters out paths that have repeating nodes.

#### Example

The following query demonstrates the use of the `ACYCLIC` path mode on a [quantified path pattern](https://docs.cloud.google.com/spanner/docs/graph/queries-overview#quantified-path-patterns) . The path with equal `src` and `dst` nodes is filtered out.

    GRAPH FinGraph
    MATCH p = ACYCLIC (src:Account)-[t:Transfers]->{2}(dst:Account)
    RETURN ARRAY_TRANSFORM(NODES(p), n->n.id) AS account_ids

#### Result

| account\_ids |
| ------------ |
| 16,20,7      |
| 20,7,16      |
| 20,7,16      |
| 7,16,20      |
| 7,16,20      |

### `TRAIL`

The `TRAIL` path mode filters out paths that have repeating edges.

#### Example

The following query demonstrates the use of the `TRAIL` path mode on a [quantified path pattern](https://docs.cloud.google.com/spanner/docs/graph/queries-overview#quantified-path-patterns) . Paths with repeated edges are filtered out.

    GRAPH FinGraph
    MATCH p = TRAIL (src:Account)-[t:Transfers]->{3}(dst:Account)
    WHERE src.id < dst.id
    RETURN ARRAY_TRANSFORM(t, t->t.id) AS transfer_ids

#### Result

| transfer\_ids |
| ------------- |
| 7,16,20       |
| 7,16,20       |

## Path search prefix

You can use a path search prefix to restrict a path pattern to return the shortest path from each [data partition](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#details_5) . For detailed semantics, see [Path search prefix](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#search_prefix) .

### `ANY SHORTEST`

The `ANY SHORTEST` path search prefix returns the shortest path (the path with the least number of edges) that matches the pattern from each data partition. If there are more than one shortest paths per partition, returns any one of them.

#### Example

The following query matches any path between each pair of `[a, b]` .

    GRAPH FinGraph
    MATCH p = ANY SHORTEST (a:Account {is_blocked:true})-[t:Transfers]->{1,4}(b:Account)
    LET total_amount = SUM(t.amount)
    RETURN a.id AS account1_id, total_amount, b.id AS account2_id;

#### Result

| account1\_id | total\_amount | account2\_id |
| ------------ | ------------- | ------------ |
| 16           | 500           | 16           |
| 16           | 800           | 7            |
| 16           | 300           | 20           |

### `ANY CHEAPEST`

The `ANY CHEAPEST` path search prefix ensures that for each pair of source and destination accounts, the query returns only one path with the minimum total compute cost. A cheapest path is one with the minimum total compute cost, calculated from `COST` expressions on edges in the path. If multiple cheapest paths exist per partition, the query returns any one of them.

#### Example

The following query matches one of the cheapest paths between each pair of accounts, represented by `a` and `b` , based on transfer amount.

    GRAPH FinGraph
    MATCH ANY CHEAPEST (a:Account)-[t:Transfers COST t.amount]->{1,3}(b:Account)
    LET total_cost = sum(t.amount)
    RETURN a.id AS account1_id, b.id AS account2_id, total_cost

#### Result

| account1\_id | account2\_id | total\_cost |
| ------------ | ------------ | ----------- |
| 7            | 7            | 900         |
| 7            | 16           | 100         |
| 7            | 20           | 400         |
| 16           | 7            | 800         |
| 16           | 16           | 500         |
| 16           | 20           | 300         |
| 20           | 7            | 500         |
| 20           | 16           | 200         |
| 20           | 20           | 500         |

## Conversion rules

For more information, see [GRAPH\_PATH conversion rules](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_rules) .

## Use case example

In the following use case example, you find all accounts have been routed through one to three accounts, from account ID `20` .

    GRAPH FinGraph
    MATCH p = (start:Account {id: 20})-[:Transfers]->{1,3}(dst:Account)
    RETURN DISTINCT dst.id AS dst;

**Result**

| dst |
| --- |
| 7   |
| 16  |
| 20  |

However, a query that returns to account ID `20` might be an overly broad query because it starts with account ID `20` . To show more specific results, you can enforce your query to show only acyclic graph paths without any repeating nodes. To do so, you can:

  - Use `MATCH p = ACYCLIC <path_pattern>` ; or
  - Apply an `IS_ACYCLIC(p)` filter in your query

The following query uses ` MATCH p = ACYCLIC PATH_PATTERN  ` :

    GRAPH FinGraph
    MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->{1,3}(dst:Account)
    RETURN DISTINCT dst.id AS dst;

**Result**

| dst |
| --- |
| 7   |
| 16  |

If you want to know the first account that money is transferred through, then you can run the following query:

    GRAPH FinGraph
    MATCH p = ACYCLIC (start:Account {id: 20})(-[:Transfers]->
      (nexts:Account)){1,3}(dst:Account)
    RETURN dst.id AS dst, ARRAY_AGG(DISTINCT nexts[0].id) AS unique_starts;

This query is unconventional because it introduces a new variable inside the quantified path using `nexts` to get the result. With path variables, you can simplify the query:

    GRAPH FinGraph
    MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->{1,3}(dst:Account)
    RETURN dst.id AS dst, ARRAY_AGG(DISTINCT NODES(p)[OFFSET(1)].id) AS unique_starts;

Using `NODES(p)` returns all nodes along the path. Because the first node account is specified as `start` , the next one (at the first offset) is the first account that money is transferred through.

**Result**

| dst | unique\_starts |
| --- | -------------- |
| 7   | 16, 7          |

Paths are more useful when there are multiple quantified paths. You can add a constraint that the paths found from `start` must pass through account ID `7` :

    GRAPH FinGraph
    MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->
      {1,3}(mid:Account {id: 7})-[:Transfers]->{1,3}(dst:Account)
    RETURN dst.id AS dst,
      ARRAY_AGG(DISTINCT NODES(p)[OFFSET(1)].id) AS unique_starts;

Although the `MATCH` statement changed, the rest of the query doesn't need to change. Without using path variables, there are cases where it's not possible for Spanner to statically know which quantified path to inspect.

Using a path variable, you can get the sum of all transfers:

    GRAPH FinGraph
    MATCH p = ACYCLIC (start:Account {id: 20})-[:Transfers]->
      {1,3}(mid:Account {id: 7})-[:Transfers]->{1,3}(dst:Account)
    LET all_transfers = EDGES(p)
    LET transfer_amounts = SUM(all_transfers.amount)
    RETURN dst.id AS dst,
      ARRAY_AGG(DISTINCT NODES(p)[OFFSET(1)].id) AS participating_neighbor_nodes, transfer_amounts;

**Result**

| dst | participating\_neighbor\_nodes | transfer\_amounts |
| --- | ------------------------------ | ----------------- |
| 16  | 7                              | 600               |
| 16  | 7                              | 800               |

## What's next

  - [Graph path type](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type)
  - [`PATH` function](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#path)
  - [Path mode](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#path_mode)
  - [Path search prefix](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#search_prefix)
