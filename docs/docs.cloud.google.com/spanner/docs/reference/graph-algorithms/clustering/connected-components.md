---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/connected-components
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/connected-components
title: Connected components
description: Learn about the connected components algorithm. Find the maximal sets of mutually reachable nodes in undirected or directed graphs.
data_source: docs.cloud.google.com
---

This algorithm computes the [connected components](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#connected_component) of an [undirected graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#undirected_graph) \\(G\\) (or a [directed graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#directed_graph) , which the algorithm treats as an undirected one; see details in the following [Input graph characteristics](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/connected-components#input_graph_characteristics) section). A connected component is a maximal set of nodes such that there is a path between any two nodes in the set. The algorithm returns a [clustering](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#cluster) containing one cluster per connected component.

## Interface specifications

The following list summarizes the expectations on the graph passed as input to this algorithm. These expectations complement the ones listed in [General assumptions on input graphs](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions) .

>   - [Directed graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#directed_graph) or [undirected graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#undirected_graph) : The algorithm ignores edge directions; see details in the following [Algorithm description](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/connected-components#algorithm_description) section.
>   - [Edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) : The algorithm does not use edge weights.

While connected components typically apply to *undirected graphs* only, for performance reasons the implementation also accepts directed graphs. Even when the input is a directed graph, the algorithm still treats edges as though they were undirected. In other words, if the algorithm sees an edge \\(xy\\), then it assumes that an edge \\(yx\\) is implicitly present even if it may not actually be present in the graph.

The output is a non-overlapping clustering, with each cluster representing a connected component.

The algorithm does not accept any configuration parameters.

## Algorithm

The algorithm uses an asynchronous implementation of a [union-find](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) data structure.

Initially, each node belongs to its own (singleton) cluster. Then, for each edge \\(xy\\) (in parallel), the algorithm merges the cluster containing \\(x\\) and the cluster containing \\(y\\) (if they are not the same cluster) into a single cluster. Edge processing happens in parallel with graph loading. The algorithm returns the clustering that results from all those cluster-merging operations.

The algorithm maintains the clustering as a disjoint set forest. While merging two clusters, it uses path compression to reduce the heights of the trees in the forest, so as to speed up future lookup and merge operations.

## Computational complexity

The algorithm is a parallel algorithm that runs in \\(O(n + m \\log n)\\) [work](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#work) and \\(O(n)\\) [depth](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#depth) , where \\(n\\) and \\(m\\) denote the number of nodes and edges in the input graph, respectively. Running the algorithm on a machine that supports high parallelism tends to lead to a smaller running time.

The algorithm uses \\(O(n)\\) memory.

## Example

> **Note:** This example uses a sequential algorithm for illustration purposes, while in the actual implementation the algorithm executes some parts in parallel.

### Input graph

Consider the following input graph:

![An undirected graph with seven nodes and various edges.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/input_graph.svg)

### Solution

The graph has three connected components, containing nodes (a, b, c, d), (f, g), and (h):

![The three connected components discovered in the graph: {a, b, c, d}, {f, g}, and {h}, each enclosed in a dashed outline.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/solution.svg)

### Algorithm details

Initially, the algorithm places each node in its own cluster:

![The initial state of the algorithm where each node is in its own individual cluster.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/starting_clusters.svg)

The algorithm then processes each edge of the graph, merging the clusters of its two endpoints (if they are not already in the same cluster).

Note that the order in which the algorithm processes the edges does not affect the result.

*Processing edge ac* : the algorithm merges clusters (a) and (c).

![Merging of nodes a and c into a single cluster after processing edge ac.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/edge_ac.svg)

*Processing edge bc* : the algorithm merges clusters (b) and (ac).

![Merging of node b into the existing {a, c} cluster after processing edge bc.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/edge_bc.svg)

*Processing edge bd* : the algorithm merges clusters (abc) and (d).

![Merging of node d into the existing {a, b, c} cluster after processing edge bd.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/edge_bd.svg)

*Processing edge cd* : nodes c and d are already in the same cluster, so the clustering remains unchanged.

![Clustering unchanged after processing edge cd, since nodes c and d are already in the same cluster.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/edge_cd.svg)

*Processing edge fg* : the algorithm merges clusters (f) and (g).

![Merging of nodes f and g into a new cluster after processing edge fg.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/connected-components/example1/edge_fg.svg)

The clustering on the right is the final result.

## External references

  - [ConnectIt: A Framework for Static and Incremental Parallel Graph Connectivity Algorithms](https://www.vldb.org/pvldb/vol14/p653-dhulipala.pdf) (Proceedings of the VLDB Endowment, 2020) first published this implementation.
  - [A Randomized Concurrent Algorithm for Disjoint Set Union](https://dl.acm.org/doi/10.1145/2933057.2933116) (Principles of Distributed Computing, 2016) inspired the asynchronous Union-Find implementation.
  - An open-source implementation is available on [GitHub](https://github.com/google/graph-mining/tree/main/in_memory/clustering/connected_components) .
