---
name: documents/docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices
uri: https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices
title: Best practices for running algorithms on Spanner Graph
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Preview — [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview)**
> 
> This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This document describes best practices for running algorithms on Spanner Graph. It covers the following topics:

  - [Graph schema best practices for algorithms](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices#graph-schema-algorithms-bp)
  - [Handle algorithm output](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices#handle-output)
  - [Specifying `machine_category` for large workloads](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices#specify-machine-category)
  - [Reusing compute for fast iteration](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices#reuse-compute)
  - [Differentiating element types in output](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices#diff-element-types)

## Graph schema best practices for algorithms

If you use customized [labels and properties](https://docs.cloud.google.com/spanner/docs/graph/schema-overview#customize-labels-properties) in Spanner Graph, we recommend that you always expose [element keys](https://docs.cloud.google.com/spanner/docs/graph/schema-overview#element-key) as properties. This way, you can access those key properties in the algorithm query's `RETURN` clause, and use them to identify the graph element for which the algorithm generates output.

## Handle algorithm output

This section shares recommendations on what to return from algorithm output and where to persist the returned results.

### What to return

When the algorithm's output signature contains graph elements, Spanner Graph lets you return any property of those elements. To reduce processing cost, we recommend that you minimize the list of properties to return. For example, only return properties that are keys of an element because keys are necessary to identify the element.

### Where to persist

Spanner Graph supports persisting algorithm query result to Cloud Storage or back to the same Spanner Graph instance where you initiate the query.

**Persisting back to Spanner** makes the algorithm output accessible to all Spanner operations natively. For example, you can run `WeaklyConnectedComponent` , persist `cluster_id` back to the graph. Subsequently, run `PageRank` for an input graph with specific `cluster_id` . Spanner features like [Change Stream](https://docs.cloud.google.com/spanner/docs/change-streams) propagate new algorithm outputs to downstream systems. If you want to use algorithm output in Spanner, we recommend persisting to Spanner. While Spanner Graph uses efficient ways to batch and persist algorithm output to Spanner, all writes must go through the same [leader](https://docs.cloud.google.com/spanner/docs/replication#overview) replica. If you already have critical workloads taking up leader capacity, you might want to stagger algorithm runs to avoid competing with critical workloads.

Consider **persisting to Cloud Storage** if you don't plan to use algorithm output in Spanner or want to evaluate the algorithm output before ingesting it to your primary data source. See supported file [formats](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#persisting-to-gcs) .

### Data modeling considerations for augmenting the graph

This section discusses some data modeling considerations when persisting algorithm output to Spanner.

#### Properties versus edges

Algorithms can produce a variety of insights, including the following:

  - Node-level metrics (for example, centrality score). These can be persisted as new properties on the node.
  - Pairwise insights (for example, similarity between two nodes). These can be persisted as new edges between the two nodes with the output value as edge property.
  - Community detection algorithms identify logical communities in the graph and can assign one or more communities to a given node. You can model community membership as a new property on the node by tagging each member node with the ID of its assigned community. You can also model community membership as new subgraphs and store the logical communities as a new type of nodes in the graph with edges connecting them to member nodes. Community property might be sufficient if you want to know the community a node belongs to when you access a node. Community subgraph might be more appropriate when you want to navigate by community (for example, find nodes in the same community as a starting node). Depending on how you plan to use community information, you can choose either or both.

#### Predefined schema versus flexible schema

Before you persist algorithm output back to Spanner, define the destination column or table in the schema in one of the following ways:

  - If your algorithm use cases are stable and known, you can add additional columns or tables beforehand.
  - If you are iterating and experimenting with different ways to extract insight (for example, trying out different algorithms, parameters, input subgraphs), you might want a flexible way to persist and differentiate output from multiple experimentations without updating the Spanner schema for each run. In this case, you can consider storing new properties and edges produced by algorithms in generic child tables.

To store new properties and edges produced by algorithms in generic child tables, follow these steps:

Step 1: Create generic child tables

    -- Create `AccountAlgoProperty` as a child table of `Account`, to store all
    -- properties produced by algorithms for `Account`.
    -- `algo_run_id`: a unique ID for a given algorithm run.
    -- `int_val`: column to store integer algorithm output.
    -- `float_val`: column to store float algorithm output.
    CREATE TABLE AccountAlgoProperty (
     id INT64 NOT NULL,
     algo_run_id STRING(200) NOT NULL,
     int_val INT64,
     float_val FLOAT64
    ) PRIMARY KEY(id, algo_run_id),
     INTERLEAVE IN PARENT Account;
    
    -- Create `AccountAlgoEdge` as a child table of `Account`, to store all
    -- outgoing edges produced by algorithms for `Account`.
    -- `algo_run_id`: a unique ID for a given algorithm run.
    -- `to_id`: destination `Account` id.
    -- `int_val`: column to store integer algorithm output.
    -- `float_val`: column to store float algorithm output.
    CREATE TABLE AccountAlgoEdge (
     id INT64 NOT NULL,
     algo_run_id STRING(200) NOT NULL,
     to_id INT64 NOT NULL,
     int_val INT64,
     float_val FLOAT64,
     CONSTRAINT FK_AccountId FOREIGN KEY (to_id) REFERENCES Account (id) NOT ENFORCED,
    ) PRIMARY KEY(id, algo_run_id, to_id),
     INTERLEAVE IN PARENT Account;

Step 2: Store properties and edges produced by algorithms as child table rows, along with the unique `algo_run_id` that produced them.

    -- Store PageRank score as property in child table.
    EXPORT DATA
     OPTIONS (format = "CLOUD_SPANNER",
              table = "AccountAlgoProperty",
              write_mode = 'upsert_ignore_all' ) AS
    GRAPH FinGraph
    CALL PageRank(node_labels => ['Account'], edge_labels => ['Transfers'])
    RETURN node.id, "page_rank_123" AS algo_run_id, score As float_val;
    
    -- Store ShortestPath output as edge in child table.
    EXPORT DATA
     OPTIONS (format = "CLOUD_SPANNER",
              table = "AccountAlgoEdge",
              write_mode = 'upsert_ignore_all' ) AS
    GRAPH FinGraph
    CALL ShortestPath(
        source_nodes => ARRAY {
                          MATCH (n:Account {id: 7})
                          RETURN n
                        },
        target_nodes => ARRAY {
                          MATCH (n:Account {id: 16})
                          RETURN n
                        }
      ) YIELD source_node, target_node, path, cost
    RETURN source_node.id AS id, "shortest_path_456" AS algo_run_id,
      target_node.id AS to_id, PATH_LENGTH(path) AS int_val, cost AS float_val;

Step 3: Access algorithm output using [`GRAPH_TABLE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-sql-queries#graph_table_operator)

    SELECT acct.id, prop.float_val AS page_rank_score
    FROM GRAPH_TABLE(
      FinGraph
      MATCH (n:Account)
      RETURN n.id
    ) acct JOIN AccountAlgoProperty prop ON acct.id = prop.id
      AND prop.algo_run_id = 'page_rank_123';
    
    SELECT acct.id, edge.to_id, edge.int_val AS path_len, edge.float_val AS cost
    FROM GRAPH_TABLE(
      FinGraph
      MATCH (n:Account)
      RETURN n.id
    ) acct JOIN AccountAlgoEdge edge ON acct.id = edge.id
      AND edge.algo_run_id = 'shortest_path_456';

Step 4: Optionally, create a logical graph augmented with algorithm generated properties and edges to make accessing algorithm output easier.

    -- Create a view that aggregates non-null algorithm properties per node into
    -- name-value pairs in JSON.
    CREATE OR REPLACE VIEW AccountWithAlgoProperties SQL SECURITY INVOKER AS
    SELECT
      n.id, ANY_VALUE(n.create_time) AS create_time,
      ANY_VALUE(n.is_blocked) AS is_blocked, ANY_VALUE(n.nick_name) AS nick_name,
      JSON_OBJECT(
        IF(COUNT(c.algo_run_id) = 0, [], ARRAY_AGG(c.algo_run_id)),
        IF(COUNT(c.algo_run_id) = 0, [], ARRAY_AGG(
          COALESCE(
            IF (c.int_val IS NULL, NULL, TO_JSON(c.int_val)),
            IF (c.float_val IS NULL, NULL, TO_JSON(c.float_val))
          )))) AS algo_props
    FROM Account AS n LEFT JOIN AccountAlgoProperty AS c ON n.id = c.id
    GROUP BY n.id;
    
    -- Create FinGraphAugmented with algorithm generated properties and edges.
    CREATE OR REPLACE PROPERTY GRAPH FinGraphAugmented
      NODE TABLES (
        AccountWithAlgoProperties AS Account
          KEY(id)
          LABEL Account PROPERTIES(
            create_time,
            id,
            is_blocked,
            nick_name,
            algo_props),
        Person
      )
      EDGE TABLES (
        PersonOwnAccount
          SOURCE KEY (id) REFERENCES Person (id)
          DESTINATION KEY (account_id) REFERENCES Account (id)
          LABEL Owns,
        AccountTransferAccount
          SOURCE KEY (id) REFERENCES Account (id)
          DESTINATION KEY (to_id) REFERENCES Account (id)
          LABEL Transfers,
        AccountAlgoEdge
          SOURCE KEY (id) REFERENCES Account (id)
          DESTINATION KEY (to_id) REFERENCES Account (id)
      );

You can then directly access algorithm properties and navigate through algorithm generated edges in graph query:

    -- Retrieve PageRank score property in graph query.
    GRAPH FinGraphAugmented
    MATCH (a:Account)
    RETURN a.id, a.algo_props.page_rank_123 AS page_rank
    ORDER BY page_rank DESC
    LIMIT 5;
    
    -- Navigate through ShortestPath edge in graph query.
    GRAPH FinGraphAugmented
    MATCH (s:Account {id: 7})-[e:AccountAlgoEdge {algo_run_id : 'shortest_path_456'}]->(d:Account)
    RETURN s.id AS src, d.id AS dst, e.int_val AS path_len, e.float_val AS cost

## Specify `machine_category` for large workloads

For most workloads, the `default` machine\_category is appropriate. If your input graph has \> 1 billion nodes and \> 10 billion edges, we recommend that you explicitly choose `large` for [`machine_category`](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) .

## Reuse compute for fast iteration

If you are experimenting with different algorithms or different input parameters on small datasets for fast iteration, we recommend that you use [`max_idle_time`](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) . A larger `max_idle_time` Spanner Graph lets you reuse compute resources it already provisioned for more algorithm workloads. This reduces the overhead of cold starting compute on demand and mitigates the risk of being unable to provision compute due to temporary lack of capacity. Note that algorithm compute is billable when it is idle.

## Differentiate element types in output

If the algorithm output contains multiple element types, you might want to include `ELEMENT_DEFINITION_NAME` in your output to differentiate them. For example:

    EXPORT DATA OPTIONS (
      uri = "gs://bucket-name/page_rank.csv",
      format = "csv",
      overwrite = TRUE) AS
    GRAPH FinGraph
    CALL PageRank()
    YIELD node, score
    RETURN ELEMENT_DEFINITION_NAME(node) AS node_type, node.id, score;

## What's next

  - [Spanner Graph run algorithms](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms) .
  - [Spanner Graph algorithms catalog](https://docs.cloud.google.com/spanner/docs/graph/algorithms) .
  - [Spanner Graph algorithm schema requirements and feature compatibility](https://docs.cloud.google.com/spanner/docs/graph/algorithm-schema-requirements-and-feature-compatibility) .
