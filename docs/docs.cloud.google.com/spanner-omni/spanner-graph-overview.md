---
name: documents/docs.cloud.google.com/spanner-omni/spanner-graph-overview
uri: https://docs.cloud.google.com/spanner-omni/spanner-graph-overview
title: Spanner Graph overview
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
update_time: "2026-05-08T21:33:01Z"
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Spanner Graph is a feature of Spanner and Spanner Omni that lets you build and query property graphs using a graph database directly within Spanner. Spanner Graph combines the scalability and reliability of Spanner with the power of graph modeling and querying.

Property graphs model data as nodes (entities) and edges (relationships between entities), both of which can have properties (labels and metadata). This is particularly useful for complex, highly connected data like social networks, fraud detection, and recommendation engines.

The topics in this document apply to Spanner Omni in the same way they apply to Spanner.

## Set up and query Spanner Graph

To start using Spanner Graph, you must first create a Spanner Omni deployment and database. After you create the database, you define a property graph schema that maps existing Spanner Omni tables to graph nodes and edges.

After setting up the schema and inserting data into the underlying tables, you can run graph queries using Graph Query Language (GQL), an extension of GoogleSQL that implements the ISO GQL standard. For more information, see [Set up and query Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/set-up) in the Spanner documentation.

## Spanner Graph schema overview

The Spanner Graph schema interprets your relational data as a graph. The schema specifies the node tables and edge tables that make up your graph. Each table maps rows from a Spanner table to graph elements. Elements can have labels to categorize them and properties to store attributes. For more information, see [Schema overview](https://docs.cloud.google.com/spanner/docs/graph/schema-overview) in the Spanner documentation.

### Create and manage a Spanner Graph schema

You use Data Definition Language (DDL) statements to create, update, or drop your property graph definition. The `CREATE PROPERTY GRAPH` statement defines the graph, specifying the underlying tables, keys, and labels. For more information, see [Create and manage a Spanner Graph schema](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema) in the Spanner documentation.

### Best practices for designing a schema

Efficient schema design is crucial for performance. Best practices include:

  - Using interleaving to colocate edges with their source nodes.

  - Using referential constraints (foreign keys) to help ensure graph integrity.

  - Creating secondary indexes on properties that are frequently filtered.

  - Choosing between schematized and schemaless designs based on your query patterns.

For more information, see [Best practices for designing a schema](https://docs.cloud.google.com/spanner/docs/graph/best-practices-designing-schema) in the Spanner documentation.

### Use SQL views to create a property graph

You can use SQL views to define the nodes and edges of your graph. To learn about the differences between using a SQL view and a table to create a graph, see [Benefits of creating graphs with views instead of tables](https://docs.cloud.google.com/spanner/docs/graph/graph-with-views-overview) in the Spanner documentation.

#### Overview of graphs created from SQL views

Using views as an abstraction layer provides row-level access control, flexible data transformations, and a smoother transition from schemaless to formalized data models. For more information, see [Overview of graphs created from SQL views](https://docs.cloud.google.com/spanner/docs/graph/graph-with-views-overview) .

#### Create a graph from SQL views

To create a graph from views, define the views using standard SQL. Then, reference them in the `NODE TABLES` or `EDGE TABLES` clauses of your `CREATE PROPERTY GRAPH` statement. You must explicitly define a `KEY` for each view-based element. The `KEY` clause specifies the columns from the source view that uniquely identify each graph element. For more information, see [Create a graph from SQL views](https://docs.cloud.google.com/spanner/docs/graph/graph-with-views-how-to) in the Spanner documentation.

## Manage Spanner Graph data

You manage data in Spanner Graph by modifying the underlying tables that define the graph. For more information, see [Manage Spanner Graph data](https://docs.cloud.google.com/spanner/docs/graph/insert-update-delete-data) in the Spanner documentation.

| Operation | Description                                                                                                                                        |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Insert    | Add new rows to the node and edge tables using `INSERT` statements or mutation APIs.                                                               |
| Update    | Modify existing properties by updating the corresponding columns in the underlying tables.                                                         |
| Delete    | Remove nodes or edges by deleting the corresponding rows. Use `ON DELETE CASCADE` to automatically remove associated edges when you delete a node. |

## Spanner Graph queries

Spanner Graph supports [Graph Query Language (GQL)](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) for queries to traverse and analyze your graph data.

### Queries overview

Graph queries use the `GRAPH` clause to specify the target graph and the `MATCH` clause to define the patterns you want to find. You can return node and edge properties, or perform aggregations on the results. For more information, see [Queries overview](https://docs.cloud.google.com/spanner/docs/graph/queries-overview) in the Spanner documentation.

### Work with paths

Paths represent sequences of nodes and edges in the graph. You can find all paths between two nodes, find the shortest path, or filter paths based on their properties or length using functions such as `PATH_LENGTH()` , `NODES()` , and `EDGES()` . For more information, see [Work with paths](https://docs.cloud.google.com/spanner/docs/graph/work-with-paths) in the Spanner documentation.

### Best practices for tuning queries

To optimize your graph queries:

  - Start your traversals from lower cardinality nodes.

  - Explicitly specify labels for all node and edge patterns.

  - Use the `IS_FIRST()` function to limit the number of edges traversed from high-cardinality *super nodes* .

For more information, see [Best practices for tuning queries](https://docs.cloud.google.com/spanner/docs/graph/best-practices-tuning-queries) in the Spanner documentation.

### Use full-text search with Spanner Graph

Spanner Graph integrates with Spanner full-text search capabilities, which lets you search for nodes or edges based on unstructured text properties using the `SEARCH()` function. For more information, see [Use full-text search with Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/full-text-search-and-graph) in the Spanner documentation.

### Use vector search with Spanner Graph

You can perform vector similarity searches on your graph data to find K-nearest neighbors (KNN) or approximate nearest neighbors (ANN). This is useful for similarity-based recommendations and AI-driven applications. For more information, see [Use vector search with Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/perform-vector-similarity-search) in the Spanner documentation.

## Manage schemaless data with Spanner Graph

For applications with evolving data models, Spanner Graph supports schemaless data management. You can store all nodes and edges in single, generic tables with `JSON` columns for properties, which lets you add new types and attributes without DDL changes. For more information, see [Manage schemaless data with Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/manage-schemaless-data) in the Spanner documentation.

## Migrate to Spanner Graph

You can migrate your existing graph data from other databases to Spanner Graph. This process typically involves exporting your data as CSV or JSON files and then importing it into the Spanner tables that back your new property graph. For more information, see [Migrate your data to Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/migrate) in the Spanner documentation.

## Spanner Graph reference for openCypher users

If you come from an openCypher background, the GQL implementation in Spanner Graph is familiar, but there are some syntax differences. This reference helps you map openCypher concepts and queries to Spanner Graph `MATCH` and `RETURN` semantics. For more information, see [Spanner Graph reference for openCypher users](https://docs.cloud.google.com/spanner/docs/graph/opencypher-reference) in the Spanner documentation.

## Troubleshoot Spanner Graph

Common issues in Spanner Graph include referential integrity violations (dangling edges) and slow-running queries. Troubleshooting involves inspecting your schema definitions, checking for missing nodes or edges, and using query plans to identify performance bottlenecks. For more information, see [Troubleshoot Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/troubleshoot) in the Spanner documentation.

## What's next

  - [Explore graph data with the Spanner Graph notebook](https://docs.cloud.google.com/spanner-omni/graph-notebook) .
