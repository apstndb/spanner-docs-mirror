---
name: documents/docs.cloud.google.com/spanner/docs/graph/best-practices-tuning-queries
uri: https://docs.cloud.google.com/spanner/docs/graph/best-practices-tuning-queries
title: Best practices for tuning Spanner Graph queries
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This document describes best practices for tuning Spanner Graph query performance, which include the following optimizations:

  - Avoid a full scan of the input table for nodes and edges.
  - Reduce the amount of data the query needs to read from storage.
  - Reduce the size of intermediate data.

## Start from lower cardinality nodes

Write the path traversal so that it starts with the lower cardinality nodes. This approach keeps the intermediate result set small, and speeds up query execution.

For example, the following queries have the same semantics:

  - Forward edge traversal:
    
        GRAPH FinGraph
        MATCH (p:Person {name:"Alex"})-[:Owns]->(a:Account {is_blocked: true})
        RETURN p.id AS person_id, a.id AS account_id;

  - Reverse edge traversal:
    
        GRAPH FinGraph
        MATCH (a:Account {is_blocked:true})<-[:Owns]-(p:Person {name: "Alex"})
        RETURN p.id AS person_id, a.id AS account_id;

Assuming that there are fewer people with the name `Alex` than there are blocked accounts, we recommend that you write this query in the forward edge traversal.

Starting from lower cardinality nodes is especially important for variable-length path traversal. The following example shows the recommended way to find accounts that are within three transfers of a given account.

    GRAPH FinGraph
    MATCH (:Account {id: 7})-[:Transfers]->{1,3}(a:Account)
    RETURN a.id;

## Specify all labels by default

Spanner Graph infers the qualifying nodes and edge labels if labels are omitted. We recommend that you specify labels for all nodes and edges where possible, because this inference might not always be possible and it might cause more labels than necessary to be scanned.

### Single MATCH statement

The following example finds accounts linked by at most 3 transfers from the given account:

    GRAPH FinGraph
    MATCH (src:Account {id: 7})-[:Transfers]->{1,3}(dst:Account)
    RETURN dst.id;

### Across MATCH statements

Specify labels on nodes and edges when they refer to the same element but are across `MATCH` statements.

The following example shows this recommended approach:

    GRAPH FinGraph
    MATCH (acct:Account {id: 7})-[:Transfers]->{1,3}(other_acct:Account)
    RETURN acct, COUNT(DISTINCT other_acct) AS related_accts
    GROUP BY acct
    
    NEXT
    
    MATCH (acct:Account)<-[:Owns]-(p:Person)
    RETURN p.id AS person, acct.id AS acct, related_accts;

## Use `IS_FIRST` to optimize queries

You can use the [`IS_FIRST`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#is_first) function to improve query performance by sampling edges and limiting traversals in graphs. This function helps handle high-cardinality nodes and optimize multi-hop queries.

If your specified sample size is too small, the query might return no data. Because of this, you might need to try different sample sizes to find the optimal balance of returned data and improved query performance.

These `IS_FIRST` examples use `FinGraph` , a financial graph with `Account` nodes and `Transfers` edges for money transfers. To create the `FinGraph` and use it to run the sample queries, see [Set up and query Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/set-up) .

### Limit traversed edges to improve query performance

When you query graphs, some nodes can have a significantly larger number of incoming or outgoing edges compared to other nodes. These high-cardinality nodes are sometimes called super nodes or hub nodes. Super nodes can cause performance issues because traversals through them might involve processing huge amounts of data, which leads to data skew and long execution times.

To optimize a query of a graph with super nodes, use the `IS_FIRST` function within a `FILTER` clause to limit the number of edges the query traverses from a node. Because accounts in `FinGraph` might have significantly higher numbers of transactions than others, you might use `IS_FIRST` to prevent an inefficient query. This technique is particularly useful when you don't need a complete enumeration of all connections from a super node.

The following query finds accounts ( `a2` ) that either directly or indirectly receive transfers from blocked accounts ( `a1` ). The query uses `IS_FIRST` to prevent slow performance when an account has many transfers by limiting the number of `Transfers` edges to consider for each `Account` .

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

This example uses the following:

  - `@max_transfers_per_account` : A query parameter that specifies the maximum number of `Transfers` edges to consider for each account ( `a1` ).

  - `PARTITION BY SOURCE_NODE_ID(selected_e)` : Ensures that the `IS_FIRST` limit applies independently for each account ( `a1` ).

  - `ORDER BY selected_e.create_time DESC` : Specifies that the most recent transfers are returned.

### Sample intermediate nodes to optimize multi-hop queries

You can also improve query efficiency by using `IS_FIRST` to sample intermediate nodes in multi-hop queries. This technique improves efficiency by limiting the number of paths the query considers for each intermediate node. To do this, break a multi-hop query into multiple `MATCH` statements separated by `NEXT` , and apply `IS_FIRST` at the midpoint where you need to sample:

    GRAPH FinGraph
    MATCH (a1:Account {is_blocked: true})-[e1:Transfers]->(a2:Account)
    FILTER IS_FIRST(1) OVER (PARTITION BY a2)
    RETURN a1, a2
    
    NEXT
    
    MATCH (a2)-[e2:Transfers]->(a3:Account)
    RETURN a1.id AS src_id, a2.id AS mid_id, a3.id AS dst_id;

To understand how `IS_FIRST` optimizes this query:

  - The clause `FILTER IS_FIRST(1) OVER (PARTITION BY a2)` is applied in the first `MATCH` statement.

  - For each intermediate account node ( `a2` ), `IS_FIRST` considers only the first incoming `Transfers` edge ( `e1` ), reducing the number of paths to explore in the second `MATCH` statement.

  - The overall two-hop query's efficiency is improved because the second `MATCH` doesn't process unnecessary data, especially when `a2` has many incoming transfers.

## Use factorized execution to optimize queries

Optimize Spanner Graph query performance by factorizing a query that traverses a [graph pattern](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition) and creates duplicate intermediate results.

A graph query can execute in factorized mode, which deduplicates high-cardinality path prefixes before traversing the rest of the path. This optimization improves query performance by reducing repeated key comparisons or hash table probes during graph query execution. To factorize a query, add the [`@{factorized_mode}` hint](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#factorized_mode) to the individual pattern traversal or at the query level.

Set `factorized_mode` to one of the following:

  - `factorize_left` : Optimizes graph traversals where many paths converge on a few nodes. Spanner Graph deduplicates paths by destination node ID and then fetches node properties to reduce redundant work.

  - `factorize_both` : Optimizes non-linear queries with multiple sub-paths that share nodes. Spanner Graph avoids generating intermediate results for each path independently. Instead, it computes each sub-path once and then combines them, delaying the cross-product of results until the end. Use this for patterns like triangles or branches.

For more information about traversal hints, see [Graph hints](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#graph_hints) .

The following examples show how to use factorized mode hints to improve graph query performance. To run the sample code, create the `FinGraph` schema in [Set up and query Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/set-up) .

### Reduce intermediate results in a linear query

Fraudulent accounts often receive a high volume of transactions. Queries designed to identify these accounts can generate large intermediate results, many of which are duplicates because they converge on a small set of destination accounts. Retrieving fraudulent node properties from these redundant intermediate results requires unnecessary computation, which can significantly slow down query performance.

The `@{factorized_mode}` hint addresses this issue by optimizing query execution. The following example demonstrates how to use this hint to retrieve account and transaction details. It traces transfers originating from a known blocked account to potentially fraudulent accounts that are one to three hops away. The unoptimized version of this query is available in the second tab.

### Factorized query

    GRAPH FinGraph
    MATCH (a1:Account {is_blocked: true})-[e1:Transfers]->{1,3}@{factorized_mode=factorize_left}(a2:Account)
    LET total_amount = SUM(e1.amount)
    RETURN a2.id, a2.create_time, total_amount;

### Unoptimized query

    GRAPH FinGraph
    MATCH (a1:Account {is_blocked: true})-[e1:Transfers]->{1,3}(a2:Account)
    LET total_amount = SUM(e1.amount)
    RETURN a2.id, a2.create_time, total_amount;

When the number of distinct destination nodes `a2` is much smaller than the number of paths leading to `a2` , this technique prevents redundant work. This is common in fraud networks, where intermediaries transfer money to a few recipients. The factorized hint performs the following actions:

  - Finds all paths that include one to three edges starting from a blocked node, and the sum of transfers along those paths.

  - Determines the number of distinct destination nodes `a2` .

  - Retrieves the `create_time` for each distinct destination node `a2` only once.

  - Combines the `create_time` for each node `a2` with the list of paths leading to `a2` to form the query result.

### Defer intermediate results generation in complex non-linear queries

Graph queries often have a complex structure, consisting of several linear subpath patterns. The following example shows a query that analyzes transactions originating from an account with `id` equal to 1. This query categorizes all destination accounts by their `create_time` into three distinct buckets:

  - Within the past 30 days
  - 30 to 90 days ago
  - Older than 90 days

The query returns all triples and the total transaction amount for these transactions.

The `@{factorized_mode=factorize_both}` hint optimizes query execution by avoiding early intermediate result materialization. It uses the fact that all subpaths in the `MATCH` clause are conditionally independent, given the value of the start node `a` . `factorize_both` evaluates the first two subpaths independently and doesn't repeat evaluation of the last subpath.

### Factorized query

    GRAPH FinGraph
    MATCH (a:Account {id: 1})-[e1:Transfers]->(a1:Account),
      @{factorized_mode=factorize_both}(a:Account)-[e2:Transfers]->(a2:Account),
      (a:Account)-[e3:Transfers]->(a3:Account)
    WHERE e1.create_time < TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
      AND e2.create_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
      AND e2.create_time < TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
      AND e3.create_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
    LET total_amount = e1.amount + e2.amount + e3.amount
    RETURN a1.id as a1_id, a2.id as a2_id, a3.id as a3_id, total_amount;

### Unoptimized query

    GRAPH FinGraph
    MATCH (a:Account {id: 1})-[e1:Transfers]->(a1:Account),
      (a:Account)-[e2:Transfers]->(a2:Account),
      (a:Account)-[e3:Transfers]->(a3:Account)
    WHERE e1.create_time < TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
      AND e2.create_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
      AND e2.create_time < TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
      AND e3.create_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
    LET total_amount = e1.amount + e2.amount + e3.amount
    RETURN a1.id as a1_id, a2.id as a2_id, a3.id as a3_id, total_amount;

For a given starting account with `id` equal to 1, each outgoing `Transfers` edge leads to multiple independent destination nodes. Consider the following example:

  - `M` is the number of accounts ( `a1` ) that have transfers in `(-inf, 90 DAYS)` range

  - `N` is the number of accounts ( `a2` ) that have transfers in `[90 DAYS, 30 DAYS)` range

  - `K` is the number of accounts ( `a3` ) that have transfers in `[30 DAYS, +inf)` range

The unoptimized query generates intermediate results early, computing the second subpath `M` times and the third subpath `M * N` times. In contrast, a factorized version optimizes query execution by doing the following:

  - Obtaining `M` destination nodes ( `a1` ) for the first subpath, and temporarily storing a set of `a1` nodes.

  - Obtaining `N` destination nodes ( `a2` ) for the second subpath only once and temporarily storing a set of `a2` nodes.

  - Obtaining `K` destination nodes ( `a3` ) for the last subpath only once.

  - Performing a cross product between temporary intermediate results to create the product of `M` x `N` x `K` records.

## What's next

  - Learn how to [query property graphs in Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/queries-overview) .
  - [Migrate to Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/migrate) .
