---
name: documents/docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview
uri: https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview
title: Spanner Graph algorithms overview
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Preview — [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview)**
> 
> This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

## Graph Algorithms Overview

Spanner Graph, in collaboration with [Google Research Graph Mining](https://research.google/teams/graph-mining/) , offers a suite of high-performance graph algorithms covering major use cases such as fraud detection, entity resolution and recommendation. The algorithms can scale up to tens of billions of edges with runtimes from minutes to tens of minutes. You run algorithms on Spanner Graph by calling an algorithm function in a Spanner Graph query.

### Fully Managed

Spanner Graph algorithm is a fully managed service that uses Spanner Data Boost and independent on-demand compute resources well suited for large-scale graph analytical workloads. The architecture lets you run computationally intensive graph algorithms with near-zero impact on existing workloads on the provisioned Spanner instance.

### Seamless GQL Integration

Graph algorithms are invoked as built-in function calls in Spanner Graph queries. You can export algorithm output to [Cloud Storage](https://cloud.google.com/storage) or write it back to Spanner to augment the graph. You can use the Google Cloud console, Google Cloud CLI, client libraries, the REST API, or the RPC API to run a Spanner Graph query with an algorithm invocation the same way you run any other [Spanner Graph query](https://docs.cloud.google.com/spanner/docs/graph/queries-overview#run-graph-query) .

The following example shows how to run a connected component analysis in a graph called `FinGraph` to identify clusters of `Accounts` connected by `Transfers` and persist output to a Cloud Storage as `my-bucket-name/my-output.csv` . For details, see [Run algorithms](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms) .

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/my-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL WeaklyConnectedComponents(node_labels => ['Account'], edge_labels => ['Transfers']) YIELD node, cluster
    RETURN node.id, cluster;

### Billing

Spanner Graph algorithms use Spanner Data Boost and independent compute resources for algorithm execution. You pay only for actual Serverless Processing Units (SPU) consumed when the algorithm compute is active. You can view graph algorithm billing information in the Google Cloud console

1.  
2.  In the **Filters** panel, filter **SKUs** to the Spanner Data Boost SKU for each region where graph algorithms were used.

For more information about Spanner pricing, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

### Permission

To invoke graph algorithms, a principal must have the `spanner.databases.runGraphAlgorithms` Identity and Access Management (IAM) permission. You can manage this permission by granting the `roles/spanner.graphIntelligenceUser` IAM, or create a custom role with the `spanner.databases.runGraphAlgorithms` permission. Note that `roles/spanner.graphIntelligenceUser` includes `roles/spanner.databaseReaderWithDataBoost` .

## What's next

  - [Spanner Graph run algorithms](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms)
  - [Spanner Graph algorithms catalog](https://docs.cloud.google.com/spanner/docs/graph/algorithms) .
  - [Spanner Graph algorithm schema requirements and feature compatibility](https://docs.cloud.google.com/spanner/docs/graph/algorithm-schema-requirements-and-feature-compatibility)
  - [Spanner Graph algorithm best practices](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices)
