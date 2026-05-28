---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/clique-aggregator
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/clique-aggregator
title: Clique aggregator clustering
description: Learn about the clique aggregator clustering algorithm. Find overlapping clusters with density guarantees, ensuring every clique is fully contained in at least one cluster.
data_source: docs.cloud.google.com
---

This algorithm computes a collection of potentially overlapping clusters of an [undirected graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#undirected_graph) , satisfying the following properties:

  - Every [clique](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#clique) of size \\(\\geq 2\\) in the graph is fully contained in at least one cluster.
  - The *density* of each cluster is at least some chosen threshold `min_density` .
  - No cluster is a subset of another cluster.

Here, the density of a cluster is defined as the number of edges in the cluster divided by the maximum possible number of edges in a cluster on \\(n\\) nodes, that is, (\\(n\\) choose \\(2\\)).

[Self-loops](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#self_loop) don't contribute to the density computation. A cluster with a single node has density \\(1\\) by convention.

## Interface specifications

The following list summarizes the expectations on the graph passed as input to this algorithm. These expectations complement the ones listed in [General assumptions on input graphs](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions) .

> **Input graph characteristics**
> 
>   - [Undirected graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#undirected_graph) .
>   - [Edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) : the algorithm does not use edge weights.

The output is a collection of potentially overlapping clusters satisfying the properties described in the preceding section. The algorithm does not guarantee that nodes that are not part of any clique of size \\(\\geq 2\\) (that is, isolated nodes) appear in any cluster.

The following parameter configures the algorithm:

| Name          | Type                  | Description                                                                                          |
| ------------- | --------------------- | ---------------------------------------------------------------------------------------------------- |
| `min_density` | floating-point number | The minimum density of the clusters that the algorithm returns. Must be in the range \\(\[0, 1\]\\). |

## Algorithm

The algorithm recursively grows clusters. Each recursive call receives a clique \\(C\\) (initially empty) and a set of candidate nodes \\(H\\) (initially all nodes in the graph), where every node in \\(C\\) is connected to every node in \\(H\\). The goal of each recursive call is to find clusters that cover all cliques in \\(G\[C \\cup H\]\\), where \\(G\[S\]\\) denotes the subgraph of \\(G\\) induced by \\(S\\), that is, the subgraph containing exactly the nodes in \\(S\\) and all edges of \\(G\\) between them.

1.  **Base case.** If \\(G\[C \\cup H\]\\) has density \\(\\geq\\) `min_density` , output \\(C \\cup H\\) as a single cluster and return.
2.  **Branch on a minimum-degree node.** Otherwise, pick a minimum-degree node \\(v \\in H\\) in \\(G\[C \\cup H\]\\). Recurse with \\(C' = C \\cup \\{v\\}\\) and \\(H'\\) set to the neighbors of \\(v\\) within \\(H\\). This covers all cliques that contain \\(v\\).
3.  **Recurse on the rest.** Remove \\(v\\) from \\(H\\) and go back to step 1 to cover all cliques that don't contain \\(v\\).

Steps 1–3 execute as a loop: the algorithm repeatedly picks the minimum-degree node, branches on its neighborhood (step 2), and removes it (step 3), until the remaining subgraph is dense enough (step 1). One can show that the minimum-degree node has at most \\(d\\) neighbors, where \\(d\\) is the [degeneracy](https://en.wikipedia.org/wiki/Degeneracy_\(graph_theory\)) of the graph, so each recursive subproblem receives at most \\(d\\) candidate nodes. Since \\(d\\) is typically much smaller than the maximum degree in real-world graphs, the subproblems remain small.

To produce an inclusion-maximal aggregator (ensuring no output cluster is a subset of another), the algorithm employs a pruning strategy from the [Bron–Kerbosch algorithm](https://en.wikipedia.org/wiki/Bron%E2%80%93Kerbosch_algorithm) for maximal clique enumeration. It maintains a set \\(X\\) of nodes whose cliques have already been covered by previous recursive calls. If some node in \\(X\\) is connected to every node in \\(H\\), then previous calls have already covered all cliques in \\(G\[C \\cup H\]\\), and the algorithm prunes the recursive call.

## Computational complexity

For a fixed value `min_density` \\(\< 1\\), the algorithm runs in \\(n \\cdot d^{O(\\log d)}\\) time, where \\(n\\) is the number of nodes and \\(d\\) is the [degeneracy](https://en.wikipedia.org/wiki/Degeneracy_\(graph_theory\)) of the graph. The degeneracy is at most the maximum degree and is typically much smaller in real-world graphs. For `min_density` \\(= 1\\), the problem reduces to maximal clique enumeration, which can produce exponentially many cliques in the worst case.

The algorithm uses \\(O(n \\cdot d)\\) memory.

## Example

Consider the following undirected graph with seven nodes:

![An undirected graph with seven nodes and various edges.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/clique-aggregator/example/input_graph.svg)

The graph has three maximal cliques: \\(\\{a, b, c\\}\\), \\(\\{c, d, e\\}\\), and \\(\\{d, e, f, g\\}\\).

With `min_density` \\(= 0.8\\), the output of the algorithm is two overlapping clusters:

  - \\(\\{a, b, c\\}\\) with density \\(3/3 = 1\\).
  - \\(\\{c, d, e, f, g\\}\\) with density \\(8/10 = 0.8\\).

![The resulting two overlapping clusters, {a, b, c} and {c, d, e, f, g}, covering all maximal cliques in the graph.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/clique-aggregator/example/output_clusters.svg)

Every maximal clique is fully contained in at least one cluster, and no cluster is a subset of another. Note that node \\(c\\) belongs to both clusters.

## External references

  - The algorithm was introduced in [Aggregating Maximal Cliques in Real-World Graphs](https://arxiv.org/abs/2512.03960) (arXiv, 2025).
  - An open-source implementation is available on [GitHub](https://github.com/google/graph-mining/tree/main/in_memory/clustering/clique_aggregator) .
