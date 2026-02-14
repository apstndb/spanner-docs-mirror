**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document describes best practices for tuning Spanner Graph query performance, which include the following optimizations:

  - Avoid a full scan of the input table for nodes and edges.
  - Reduce the amount of data the query needs to read from storage.
  - Reduce the size of intermediate data.

## Start from lower cardinality nodes

Write the path traversal so that it starts with the lower cardinality nodes. This approach keeps the intermediate result set small, and speeds up query execution.

For example, the following queries have the same semantics:

  - Forward edge traversal:
    
    ``` text
    GRAPH FinGraph
    MATCH (p:Person {name:"Alex"})-[:Owns]->(a:Account {is_blocked: true})
    RETURN p.id AS person_id, a.id AS account_id;
    ```

  - Reverse edge traversal:
    
    ``` text
    GRAPH FinGraph
    MATCH (a:Account {is_blocked:true})<-[:Owns]-(p:Person {name: "Alex"})
    RETURN p.id AS person_id, a.id AS account_id;
    ```

Assuming that there are fewer people with the name `  Alex  ` than there are blocked accounts, we recommend that you write this query in the forward edge traversal.

Starting from lower cardinality nodes is especially important for variable-length path traversal. The following example shows the recommended way to find accounts that are within three transfers of a given account.

``` text
GRAPH FinGraph
MATCH (:Account {id: 7})-[:Transfers]->{1,3}(a:Account)
RETURN a.id;
```

## Specify all labels by default

Spanner Graph infers the qualifying nodes and edge labels if labels are omitted. We recommend that you specify labels for all nodes and edges where possible, because this inference might not always be possible and it might cause more labels than necessary to be scanned.

### Single MATCH statement

The following example finds accounts linked by at most 3 transfers from the given account:

``` text
GRAPH FinGraph
MATCH (src:Account {id: 7})-[:Transfers]->{1,3}(dst:Account)
RETURN dst.id;
```

### Across MATCH statements

Specify labels on nodes and edges when they refer to the same element but are across `  MATCH  ` statements.

The following example shows this recommended approach:

``` text
GRAPH FinGraph
MATCH (acct:Account {id: 7})-[:Transfers]->{1,3}(other_acct:Account)
RETURN acct, COUNT(DISTINCT other_acct) AS related_accts
GROUP BY acct

NEXT

MATCH (acct:Account)<-[:Owns]-(p:Person)
RETURN p.id AS person, acct.id AS acct, related_accts;
```

## Use `     IS_FIRST    ` to optimize queries

You can use the [`  IS_FIRST  `](/spanner/docs/reference/standard-sql/graph-gql-functions#is_first) function to improve query performance by sampling edges and limiting traversals in graphs. This function helps handle high-cardinality nodes and optimize multi-hop queries.

If your specified sample size is too small, the query might return no data. Because of this, you might need to try different sample sizes to find the optimal balance of returned data and improved query performance.

These `  IS_FIRST  ` examples use `  FinGraph  ` , a financial graph with `  Account  ` nodes and `  Transfers  ` edges for money transfers. To create the `  FinGraph  ` and use it to run the sample queries, see [Set up and query Spanner Graph](/spanner/docs/graph/set-up) .

### Limit traversed edges to improve query performance

When you query graphs, some nodes can have a significantly larger number of incoming or outgoing edges compared to other nodes. These high-cardinality nodes are sometimes called super nodes or hub nodes. Super nodes can cause performance issues because traversals through them might involve processing huge amounts of data, which leads to data skew and long execution times.

To optimize a query of a graph with super nodes, use the `  IS_FIRST  ` function within a `  FILTER  ` clause to limit the number of edges the query traverses from a node. Because accounts in `  FinGraph  ` might have significantly higher numbers of transactions than others, you might use `  IS_FIRST  ` to prevent an inefficient query. This technique is particularly useful when you don't need a complete enumeration of all connections from a super node.

The following query finds accounts ( `  a2  ` ) that either directly or indirectly receive transfers from blocked accounts ( `  a1  ` ). The query uses `  IS_FIRST  ` to prevent slow performance when an account has many transfers by limiting the number of `  Transfers  ` edges to consider for each `  Account  ` .

``` text
GRAPH FinGraph
MATCH
(a1:Account {is_blocked: true})
-[e:Transfers WHERE e IN
  {
    MATCH -[selected_e:Transfers]->
    FILTER IS_FIRST(@max_transfers_per_account) OVER (
      PARTITION BY SOURCE_NODE_ID(selected_e)
      ORDER BY selected_e.create_time DESC)
    RETURN selected_e
  }
]->{1,5}
(a2:Account)
RETURN a1.id AS src_id, a2.id AS dst_id;
```

This example uses the following:

  - `  @max_transfers_per_account  ` : A query parameter that specifies the maximum number of `  Transfers  ` edges to consider for each account ( `  a1  ` ).

  - `  PARTITION BY SOURCE_NODE_ID(selected_e)  ` : Ensures that the `  IS_FIRST  ` limit applies independently for each account ( `  a1  ` ).

  - `  ORDER BY selected_e.create_time DESC  ` : Specifies that the most recent transfers are returned.

### Sample intermediate nodes to optimize multi-hop queries

You can also improve query efficiency by using `  IS_FIRST  ` to sample intermediate nodes in multi-hop queries. This technique improves efficiency by limiting the number of paths the query considers for each intermediate node. To do this, break a multi-hop query into multiple `  MATCH  ` statements separated by `  NEXT  ` , and apply `  IS_FIRST  ` at the midpoint where you need to sample:

``` text
GRAPH FinGraph
MATCH (a1:Account {is_blocked: true})-[e1:Transfers]->(a2:Account)
FILTER IS_FIRST(1) OVER (PARTITION BY a2)
RETURN a1, a2

NEXT

MATCH (a2)-[e2:Transfers]->(a3:Account)
RETURN a1.id AS src_id, a2.id AS mid_id, a3.id AS dst_id;
```

To understand how `  IS_FIRST  ` optimizes this query:

  - The clause `  FILTER IS_FIRST(1) OVER (PARTITION BY a2)  ` is applied in the first `  MATCH  ` statement.

  - For each intermediate account node ( `  a2  ` ), `  IS_FIRST  ` considers only the first incoming `  Transfers  ` edge ( `  e1  ` ), reducing the number of paths to explore in the second `  MATCH  ` statement.

  - The overall two-hop query's efficiency is improved because the second `  MATCH  ` doesn't process unnecessary data, especially when `  a2  ` has many incoming transfers.

## What's next

  - Learn how to [query property graphs in Spanner Graph](/spanner/docs/graph/queries-overview) .
  - [Migrate to Spanner Graph](/spanner/docs/graph/migrate) .
