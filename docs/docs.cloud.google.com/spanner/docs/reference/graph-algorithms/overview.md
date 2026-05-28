---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/overview
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/overview
title: Graph algorithms overview
description: Overview of the available graph algorithms. Includes node centrality, graph clustering, node similarity, and path finding algorithms.
data_source: docs.cloud.google.com
---

Many datasets contain valuable insights hidden within relationships between data points. Graphs represent such relationships—structures that model entities (nodes) and the connections between them (edges).

The graph algorithms in this documentation enable you to analyze such relationships at scale. By analyzing graphs, you can uncover hidden patterns, identify key entities, and gain a deeper understanding of complex datasets. Prominent applications of graph algorithms include personalization, recommendation, identity resolution, and abuse detection.

For definitions of graph terms used in this documentation, see the [Graph algorithms glossary](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary) . You should also review the [General assumptions on input graphs](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions) for shared constraints, and the [Computational model](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/computational-model) to understand the *work* and *depth* concepts used to characterize algorithm complexity.

The available algorithms fall into four categories:

  - Centrality
  - Clustering
  - Similarity
  - Path finding

For a simplified overview of each algorithm, see the corresponding section on this page. For in-depth explanations, configuration options, and examples, refer to the dedicated documentation page linked for each algorithm.

## Centrality

Node centrality algorithms rank nodes by their structural importance within a graph. Different algorithms capture different notions of importance—for example, how well connected a node is, or how much information flows through it. This is useful for detecting influential accounts in social networks, identifying critical nodes in infrastructure networks, and spotting bottlenecks in communication flows. These centrality algorithms are available.

  - [Betweenness centrality](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/betweenness-centrality) : Measures how often a node lies on shortest paths between other pairs of nodes. Nodes with high betweenness centrality often act as critical bridges connecting different parts of the graph.
  - [Closeness centrality](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/closeness-centrality) : Measures how close a node is to all other nodes in the graph. Nodes with high closeness centrality can reach other nodes through shorter paths.
  - [PageRank](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/pagerank) : Scores nodes by importance, based on the idea that a node is more important if many other important nodes link to it. The algorithm simulates a random walk through the graph; the walk visits important nodes more often.

## Clustering

Graph clustering algorithms group nodes into *clusters* (also known as *communities* ) such that nodes within each cluster connect more densely to each other than to nodes in other clusters. This is useful for community detection, entity resolution, customer segmentation, and discovering groups of related items. Clustering algorithms typically produce disjoint clusters, but there are exceptions; among the following algorithms, clique aggregator clustering is the only one that can produce overlapping clusters. Each of these algorithms requires an undirected graph as input, with the exception of connected components, which also accepts directed graphs (treating them as undirected).

  - [Connected components](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/connected-components) : Finds disjoint sets of nodes such that paths connect every node in a set to any other node in the same set, but no paths connect nodes in different sets.
  - [Correlation clustering](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/correlation-clustering) : Partitions nodes based on their pairwise similarity or dissimilarity, aiming to group similar nodes together while separating dissimilar ones.
  - [Majority vote label propagation](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/label-propagation) : Assigns nodes to clusters using a label propagation technique, starting from an initial labeling which may use seed labels in the input. As nodes iteratively adopt the label shared by the majority of their neighbors, densely connected groups converge to a consensus label, forming communities.
  - [Modularity clustering](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/modularity-clustering) : Partitions the graph into clusters by optimizing for *modularity* , a quality measure of the density of connections within clusters compared to what a random network would produce.
  - [Clique aggregator clustering](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/clique-aggregator) : Identifies potentially overlapping dense communities that meet a minimum density threshold. At least one cluster contains every clique (a group of nodes where each node connects to every other node).

## Similarity

Node similarity algorithms quantify how alike pairs of nodes are based on the structure of their neighborhoods. This is useful for link prediction (inferring missing edges), deduplication (finding nodes that likely represent the same entity), and recommendation (suggesting items based on similar connection patterns).

  - [Pairwise node similarity](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-similarity/pairwise-node-similarity) : Computes similarity scores between pairs of nodes based on their neighborhoods. Supported metrics include Jaccard similarity (ratio of common neighbors to total neighbors), cosine similarity (based on edge weights of common neighbors), common neighbors (number of shared neighbors), and total neighbors (number of neighbors of at least one of the two nodes).

## Path finding

Path-finding algorithms compute optimal routes between nodes. This is useful for analyzing reachability and proximity between entities, tracing information flow or supply chains, and identifying the cheapest connections in a network.

  - [Many-to-many shortest paths](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/traversal/shortest-paths) : Computes shortest paths between a given set of source nodes and a given set of target nodes.
