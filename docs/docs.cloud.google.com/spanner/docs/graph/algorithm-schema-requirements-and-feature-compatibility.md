---
name: documents/docs.cloud.google.com/spanner/docs/graph/algorithm-schema-requirements-and-feature-compatibility
uri: https://docs.cloud.google.com/spanner/docs/graph/algorithm-schema-requirements-and-feature-compatibility
title: Spanner Graph algorithm schema requirements and feature compatibility
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Preview — [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview)**
> 
> This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

## Schema requirements

Spanner Graph algorithms can run on graphs with schemas that meet the following requirements:

  - An edge input table must be interleaved into its source node input table and have a [Foreign Key](https://docs.cloud.google.com/spanner/docs/foreign-keys/overview#enforced-foreign-keys) or [Informational Foreign Key](https://docs.cloud.google.com/spanner/docs/foreign-keys/overview#informational-foreign-keys) to its destination node input table. See [Best practices for designing a Spanner Graph schema](https://docs.cloud.google.com/spanner/docs/graph/best-practices-designing-schema#enforced-foreign-key) .
  - Each input table to the graph algorithm must have a unique identifying label.

The following is an example of an unsupported schema:

    -- Unsupported: Both `Person` and `Account` can only be referred to via label `Entity`.
    CREATE PROPERTY GRAPH FinGraph
      NODE TABLES (
        Person KEY (id)
          LABEL Entity PROPERTIES (...),
        Account KEY (id)
          LABEL Entity PROPERTIES (...)
      );

The following is an example of a supported schema:

    -- Supported: Even though label `Entity` refers to both `Person` and `Account`,
    -- label `Customer` uniquely identifies `Person`, label `Account` uniquely identifies `Account`.
    CREATE PROPERTY GRAPH FinGraph
      NODE TABLES (
        Person KEY (id)
          LABEL Customer PROPERTIES (...)
          LABEL Entity PROPERTIES (...),
        Account KEY (id)
          LABEL Account PROPERTIES (...)
          LABEL Entity PROPERTIES (...)
      );

## Feature compatibility

This section outlines algorithm query compatibility with general graph features.

### Supported MATCH clause

When you use the `MATCH` clause in [algorithm query structure](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#algo-query-structure) to define the input graph for algorithms, you must consider the following:

1.  Each `MATCH` must contain only one element pattern (node or edge) with one named variable.
2.  When an edge pattern includes explicit patterns for end nodes, don't use a node label or variable in the node patterns.
3.  You can use a `WHERE` clause within the element pattern if it does not refer to variables defined elsewhere.
4.  The label expression in the `MATCH` clause must uniquely identify one element table.

The following examples demonstrate supported `MATCH` clauses:

    -- Supported: Assuming only 1 element table has `Account` label:
    MATCH (a:Account)
    
    MATCH (a:Account WHERE a.id > 400)
    
    MATCH (a:Account WHERE a.id IN [101, 102, 105])
    
    MATCH -[e:Transfers WHERE e.amount < 500]->

The following examples demonstrate unsupported `MATCH` clauses:

    -- Unsupported: End nodes for edge patterns cannot have labels or named variables.
    MATCH (a:Account)-[]->
    
    -- Unsupported: Each MATCH can only name one variable.
    MATCH (a:Account)-[e:Transfers]->
    
    -- Unsupported: If there are multiple node element tables in the graph, each MATCH
    -- must uniquely identify one element table.
    MATCH (a)
    
    -- Unsupported: The `Entity` label maps to both `Account` and `Person` node element tables.
    MATCH (a:Account | Entity)

### Use RETURN clause

The `RETURN` clause in [algorithm query structure](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#algo-query-structure) defines the data returned by the algorithm query. You can refer to columns from the algorithm's output signature in the `RETURN` clause, with some constraints:

  - If an algorithm outputs a [Node](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) , you can return any property of the node. You can also return [ELEMENT\_DEFINITION\_NAME](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) of the node.
  - If an algorithm outputs a [Path](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type) , you can return `PATH_LENGTH(path).`
  - You can return any scalar output from the algorithm.
  - You can return any literal Constant.

The following examples demonstrate supported `RETURN` clauses:

    -- Supported.
    CALL PageRank(...) YIELD node, score
    RETURN ELEMENT_DEFINITION_NAME(node) AS node_type, node.id, score
    
    -- Supported: You can return constants.
    CALL PageRank(...) YIELD node, score
    RETURN node.id, score, "pagerank-run1" AS algo_run_id
    
    -- Supported.
    CALL ShortestPath(...) YIELD source_node, target_node, path, cost
    RETURN source_node.id AS source_id, target_node.id AS target_id,
      PATH_LENGTH(path) AS length, cost

The following examples demonstrate unsupported `RETURN` clauses:

    -- Unsupported: You cannot return the graph element `node` directly.
    RETURN node, score
    
    -- Unsupported: You can only return properties of `node`. Applying a function to a node property
    -- is not supported.
    RETURN node.id + 1, score
    
    -- Unsupported: You can only return the scalar `score` directly from the algorithm output. Applying
    -- a function to scalar output is not supported.
    RETURN node.id, score + 1
    
    -- Unsupported: General function calls are not allowed.
    RETURN JSON_OBJECT(node.id, score) as json_obj

### Parameters are not supported

Algorithm queries don't support [query parameters](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/lexical#query_parameters) .

### Support for graphs created from a view

When you create your graph on a [view](https://docs.cloud.google.com/spanner/docs/graph/graph-with-views-how-to#create-graph-views) , algorithm queries are only supported if the nodes and edges (including the end nodes of an edge) used to define the algorithm input are not based on a view.

## What's next

  - [Spanner Graph run algorithms](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms) .
  - [Spanner Graph algorithms catalog](https://docs.cloud.google.com/spanner/docs/graph/algorithms) .
  - [Spanner Graph algorithm best practices](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices) .
