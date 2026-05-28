---
name: documents/docs.cloud.google.com/spanner/docs/graph/algorithms
uri: https://docs.cloud.google.com/spanner/docs/graph/algorithms
title: Spanner Graph algorithms catalog
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Preview — [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview)**
> 
> This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

Spanner Graph, in close collaboration with [Google Research Graph Mining](https://research.google/teams/graph-mining/) , offers a suite of algorithms covering graph analysis needs for the following use cases:

  - [Centrality](https://docs.cloud.google.com/spanner/docs/graph/algorithms#centrality)
  - [Clustering](https://docs.cloud.google.com/spanner/docs/graph/algorithms#clustering)
  - [Similarity](https://docs.cloud.google.com/spanner/docs/graph/algorithms#similarity)
  - [Path Finding](https://docs.cloud.google.com/spanner/docs/graph/algorithms#path-finding)

## Centrality

Centrality algorithms rank nodes by their structural importance within a graph. It can help you identify fraudulent accounts in fintech, influencers in social graphs, critical routers in telecom networks and more.

### PageRank

`PageRank` scores nodes by importance. A node is considered more important if many other important nodes link to it. The algorithm simulates a random walk through the graph. Nodes that are visited more often in this walk are considered more important. See [page rank](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/pagerank) for algorithm details.

#### Function signature

`PageRank(input_parameters) YIELD node, score`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name              | Type                 | Required | Default | Description                                                                                                                                                               |
| ----------------- | -------------------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| source\_nodes     | Array of graph nodes | No       | (none)  | If present, source for personalized PageRank.                                                                                                                             |
| damping\_factor   | double               | No       | 0.85    | The probability that, at any given iteration, the algorithm chooses to traverse one of the outgoing edges of the current node. Must be in the range of \[0, 1).           |
| max\_iterations   | int                  | No       | 10      | The maximum number of iterations of the algorithm. Must be positive. A higher number of iterations tends to produce more precise results at the cost of a longer runtime. |
| approx\_precision | double               | No       | 1e-2    | Approximation precision threshold for the algorithm. Must be non-negative. Smaller values tends to produce more precise results at the cost of longer runtime.            |

#### Output

Function yields:

| Name  | Type                                                                                                                           | Description                     |
| ----- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------- |
| node  | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.            |
| score | FLOAT64                                                                                                                        | The PageRank score of the node. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/my-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL PageRank(
        node_labels => ['Account'], edge_labels => ['Transfers'],
        source_nodes => ARRAY {
                          MATCH (n:Account {id:7})
                          RETURN n
                        }
      ) YIELD node, score
    RETURN node.id, score;

### BetweennessCentrality

`BetweennessCentrality` measures how often a node lies on shortest paths between other pairs of nodes. Nodes with high betweenness centrality often act as critical bridges connecting different parts of the graph. See [betweenness centrality](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/betweenness-centrality) for algorithm details.

#### Function signature

`BetweennessCentrality(input_parameters) YIELD node, centrality`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name               | Type  | Required | Default | Description                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------ | ----- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| num\_source\_nodes | INT64 | No       | (none)  | If set, must be positive, and specifies how many source nodes the algorithm should select for computing approximate betweenness centralities. If not set, or set to a number larger than the number of nodes in the graph, then all nodes are used as source nodes, that is, the algorithm computes exact betweenness centralities. Higher values lead to better approximations, but also to larger running times. |

#### Output

Function yields:

| Name       | Type                                                                                                                           | Description                       |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------ | --------------------------------- |
| node       | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.              |
| centrality | FLOAT64                                                                                                                        | The centrality score of the node. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/my-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL BetweennessCentrality(
        node_labels => ['Account'], edge_labels => ['Transfers'], num_source_nodes => 5
      ) YIELD node, centrality
    RETURN node.id, centrality;

### ClosenessCentrality

`ClosenessCentrality` measures how close a node is to all other nodes in the graph. Nodes with high closeness centrality can reach other nodes through shorter paths. See [closeness centrality](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/closeness-centrality) for algorithm details.

#### Function signature

`ClosenessCentrality(input_parameters) YIELD node, centrality`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name                  | Type    | Required | Default | Description                                                                                                                                                                                                                                                                   |
| --------------------- | ------- | -------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| mode                  | STRING  | No       | EXACT   | The algorithm mode. Supported values are `EXACT` and `HYBRID` .                                                                                                                                                                                                               |
| use\_wasserman\_faust | BOOL    | No       | FALSE   | If \`false\`, compute standard closeness centralities. If \`true\`, compute Wasserman-Faust closeness centralities.                                                                                                                                                           |
| epsilon               | FLOAT64 | No       | 0.1     | Used only for the \`HYBRID\` algorithm. Must be in the range (0.0, 1.0). Smaller values generally means higher precision. Larger values generally allows the algorithm to execute faster.                                                                                     |
| sample\_size          | INT64   | No       | 0       | Used only for the \`HYBRID\` algorithm. The number of pivot nodes to use for approximately computing the centrality values. Must be non-negative. If not set or zero, a default value of min(100 \* ln( `N` ), `N` ) is used, where `N` is the node count of the input graph. |

#### Output

Function yields:

| Name       | Type                                                                                                                           | Description                                 |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| node       | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.                        |
| centrality | FLOAT64                                                                                                                        | The closeness centrality score of the node. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/my-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL ClosenessCentrality(
        node_labels => ['Account'], edge_labels => ['Transfers'],
        mode => 'EXACT'
      ) YIELD node, centrality
    RETURN node.id, centrality;

## Clustering

Clustering algorithms group nodes into clusters (also known as communities) such that nodes within each cluster are more densely connected to each other than to nodes in other clusters. This is useful to detect potential fraud rings in fintech, identify customer segmentations in retail and more.

### WeaklyConnectedComponents

`WeaklyConnectedComponents` finds disjoint sets of nodes such that every node in a set is reachable from any other node in the same set, but not from nodes in other sets. See [connected components](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/connected-components) for algorithm details.

#### Function signature

`WeaklyConnectedComponents(input_parameters) YIELD node, cluster`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) .

#### Output

Function yields:

| Name    | Type                                                                                                                           | Description                                                                                                                               |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| node    | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.                                                                                                                      |
| cluster | INT64                                                                                                                          | The ID of the connected component/cluster the node belongs to. In the range of \[0, `N` ), where `N` is the number of nodes in the graph. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/wcc-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL WeaklyConnectedComponents(
        node_labels => ['Account'], edge_labels => ['Transfers']
      ) YIELD node, cluster
    RETURN node.id, cluster;

### ModularityClustering

`ModularityClustering` partitions the graph into clusters by optimizing for modularity, a quality measure of the density of connections within clusters compared to what would be expected in a random network. See [modularity clustering](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/modularity-clustering) for algorithm details.

#### Function signature

`ModularityClustering(input_parameters) YIELD node, cluster`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name                   | Type   | Required | Default | Description                                                                                                                                                                                                                                                                                                                                                |
| ---------------------- | ------ | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| resolution             | double | No       | 1.0     | Controls the granularity of the clustering. Must be finite and non-negative. Smaller resolution values tend to lead to larger clusters. Typical values are in the range \[0.5, 5\]. When the resolution is zero, the algorithm (with a sufficient number of iterations) finds the connected components of the graph (ignoring edges whose weight is zero). |
| max\_iterations        | int    | No       | 10      | Maximum number of outer iterations of the algorithm. Must be positive. Larger values tend to lead to higher-quality clusterings at the cost of longer runtime.                                                                                                                                                                                             |
| max\_inner\_iterations | int    | No       | 10      | Maximum number of inner iterations of the algorithm. Must be positive. Larger values tend to lead to higher-quality clusterings at the cost of longer runtime.                                                                                                                                                                                             |

#### Output

Function yields:

| Name    | Type                                                                                                                           | Description                                                                                                                     |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| node    | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.                                                                                                            |
| cluster | INT64                                                                                                                          | The ID of the community/cluster the node belongs to. In the range of \[0, `N` ), where `N` is the number of nodes in the graph. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/modularity-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL ModularityClustering(
        node_labels => ['Account'], edge_labels => ['Transfers'],
        resolution => 1.0, max_iterations => 10
      ) YIELD node, cluster
    RETURN node.id, cluster;

### CorrelationClustering

`CorrelationClustering` partitions nodes based on their pairwise similarity or dissimilarity, aiming to group similar nodes together while separating dissimilar ones. See [correlation clustering](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/correlation-clustering) for algorithm details.

#### Function signature

`CorrelationClustering(input_parameters) YIELD node, cluster`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name                   | Type   | Required | Default | Description                                                                                                                                                                                                                                                                                                                           |
| ---------------------- | ------ | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| resolution             | double | Yes      | (none)  | Controls the granularity of the clustering. Must be explicitly defined by client. Must be finite and non-negative. Smaller values tend to lead to larger clusters. When the resolution is zero and all edge weights are positive, the algorithm (with a sufficient number of iterations) finds the connected components of the graph. |
| max\_iterations        | int    | No       | 10      | Maximum number of outer iterations of the algorithm. Must be positive. Larger values tend to lead to higher-quality clusterings at the cost of longer runtime.                                                                                                                                                                        |
| max\_inner\_iterations | int    | No       | 10      | Maximum number of inner iterations of the algorithm. Must be positive. Larger values tend to lead to higher-quality clusterings at the cost of longer runtime.                                                                                                                                                                        |

#### Output

Function yields:

| Name    | Type                                                                                                                           | Description                                                                                                                     |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| node    | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.                                                                                                            |
| cluster | INT64                                                                                                                          | The ID of the community/cluster the node belongs to. In the range of \[0, `N` ), where `N` is the number of nodes in the graph. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/correlation-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL CorrelationClustering(
        node_labels => ['Account'], edge_labels => ['Transfers'],
        resolution => 0.5
      ) YIELD node, cluster
    RETURN node.id, cluster;

### LabelPropagation

`LabelPropagation` assigns nodes to clusters using a label propagation technique, starting from an initial labeling which may use seed labels provided in the input. As nodes iteratively adopt the label shared by the majority of their neighbors, densely connected groups converge to a consensus label, forming communities. See [label propagation](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/label-propagation) for algorithm details.

#### Function signature

`LabelPropagation(input_parameters) YIELD node, cluster`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name                  | Type   | Required | Default | Description                                                                                                     |
| --------------------- | ------ | -------- | ------- | --------------------------------------------------------------------------------------------------------------- |
| seed\_label\_property | string | No       | (none)  | The node property name for the seed label.                                                                      |
| max\_iterations       | int    | No       | 10      | The maximum number of iterations of label propagation that the algorithm performs. Must be finite and positive. |

#### Output

Function yields:

| Name    | Type                                                                                                                           | Description                                  |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------- |
| node    | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.                         |
| cluster | INT64                                                                                                                          | The propagated cluster/label ID of the node. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/lp-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL LabelPropagation(
        node_labels => ['Account'], edge_labels => ['Transfers'],
        max_iterations => 10
      ) YIELD node, cluster
    RETURN node.id, cluster;

### CliqueFinding

`CliqueFinding` identifies potentially overlapping dense communities that meet a minimum density threshold, in such a way that every clique (a group of nodes where each node is connected to every other node) is contained in at least one cluster. See [clique aggregator](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/clique-aggregator) for algorithm details.

#### Function signature

`CliqueFinding(input_parameters) YIELD node, clique`

**Note:** Unlike most algorithms, there can be multiple rows for the same node since a node may be part of many cliques.

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name         | Type   | Required | Default | Description                                                              |
| ------------ | ------ | -------- | ------- | ------------------------------------------------------------------------ |
| min\_density | double | No       | 0.9     | The minimum density of the clusters to be returned. Must be in \[0, 1\]. |

#### Output

Function yields:

| Name   | Type                                                                                                                           | Description                               |
| ------ | ------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| node   | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | A node in the graph.                      |
| clique | INT64                                                                                                                          | The ID of the clique the node belongs to. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/clique-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL CliqueFinding(
        node_labels => ['Account'], edge_labels => ['Transfers'],
        min_density => 0.9
      ) YIELD node, clique
    RETURN node.id, clique;

## Similarity

Similarity algorithms quantify how alike pairs of nodes are based on the structure of their neighborhoods. This is useful for inferring missing edges between nodes for entity resolution, recommendation and more.

Spanner Graph supports the following pairwise node similarities where similarity scores are computed based on neighborhoods of the nodes.

  - `JaccardSimilarity` : Based on ratio of common neighbors to total neighbors
  - `CosineSimilarity` : Based on edge weights of common neighbors
  - `CommonNeighborsSimilarity` : Based on number of shared neighbors
  - `TotalNeighborsSimilarity` : Based on number of neighbors of at least one of the two nodes

See [pairwise node similarity](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-similarity/pairwise-node-similarity) for algorithm details.

These four algorithms share the same arguments and output structure.

#### Function signature

`[JaccardSimilarity|CosineSimilarity|CommonNeighborsSimilarity|TotalNeighborsSimilarity](input_parameters) YIELD source_node, target_node, similarity`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name          | Type           | Required | Default | Description |
| ------------- | -------------- | -------- | ------- | ----------- |
| source\_nodes | Array of nodes | Yes      | N/A     |             |
| target\_nodes | Array of nodes | Yes      | N/A     |             |

#### Output

Function yields:

| Name         | Type                                                                                                                           | Description                                                     |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------- |
| source\_node | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | The source node used in the similarity calculation.             |
| target\_node | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | The target node used in the similarity calculation.             |
| similarity   | FLOAT64                                                                                                                        | The calculated similarity score between source and target node. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/jaccard-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL JaccardSimilarity(
        source_nodes => ARRAY {
                          MATCH (n:Account {id: 7})
                          RETURN n
                        },
        target_nodes => ARRAY {
                          MATCH (n:Account)
                          WHERE n.id != 7
                          RETURN n
                        }
      ) YIELD source_node, target_node, similarity
    RETURN source_node.id AS source_id, target_node.id AS target_id, similarity;

## Path Finding

Path finding algorithms compute optimal routes between nodes. This is useful for identifying cheapest route in supply chains, assessing vulnerability in cybersecurity and more.

### ShortestPath

`ShortestPath` computes shortest paths between a given set of source nodes and a given set of target nodes. See [many-to-many shortest paths](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/traversal/shortest-paths) for algorithm details.

#### Function signature

`ShortestPath(input_parameters) YIELD source_node, target_node, path, cost`

#### Input parameters

All [common input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) and:

| Name          | Type           | Required | Default | Description |
| ------------- | -------------- | -------- | ------- | ----------- |
| source\_nodes | Array of nodes | Yes      | N/A     |             |
| target\_nodes | Array of nodes | Yes      | N/A     |             |

#### Output

Function yields:

| Name         | Type                                                                                                                           | Description                    |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------ |
| source\_node | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | The source node of the path.   |
| target\_node | [GRAPH\_ELEMENT](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) (node) | The target node of the path.   |
| path         | GRAPH\_PATH                                                                                                                    | The shortest path found.       |
| cost         | FLOAT64                                                                                                                        | The cost of the shortest path. |

#### Example

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/shortest-path-output.csv",
      format = "csv"
    ) AS
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
    RETURN source_node.id AS source_id, target_node.id AS target_id, PATH_LENGTH(path) AS length, cost;

## What's next

  - [Spanner Graph run algorithms](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms) .
  - [Spanner Graph algorithm schema requirements and feature compatibility](https://docs.cloud.google.com/spanner/docs/graph/algorithm-schema-requirements-and-feature-compatibility) .
  - [Spanner Graph algorithm best practices](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices) .
