---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/correlation-clustering
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/correlation-clustering
title: Correlation clustering
description: Learn about the correlation clustering algorithm. Partition nodes into non-overlapping clusters by maximizing the LambdaCC objective on graphs with positive, zero, or negative edge weights.
data_source: docs.cloud.google.com
---

This algorithm computes a non-overlapping clustering of an [undirected graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#undirected_graph) with [edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) that may be negative, zero, or positive, heuristically trying to maximize the LambdaCC objective (which is a generalization of the correlation clustering objective), defined as follows. Let

  - \\(k\_x\\) be a non-negative weight that the algorithm assigns to node \\(x\\). Note that the implementation sets \\(k\_x = 1\\) for all nodes.
  - \\(w\_{xy}\\) be the weight of the edge \\(xy\\) in the graph, or \\(0\\) if there is no such edge. The graph must be undirected, so \\(w\_{xy} = w\_{yx}\\). Edge weights may be negative, zero, or positive.
  - \\(r\\) be a non-negative resolution parameter, which controls the granularity of the clustering. Smaller resolution values tend to lead to larger clusters.
  - \\(C\\) be the set of all clusters in the clustering. Each cluster is a set of nodes, and exactly one cluster contains each node.

Then, the following formula defines the LambdaCC objective:

\\\[ \\sum\_{c \\in C} \\sum\_{x, y \\in c} w\_{xy} - r \\cdot k\_x \\cdot k\_y . \\\]

The second sum goes over all unordered pairs of nodes in a cluster exactly once. In particular, the algorithm considers each undirected edge, including [self-loops](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#self_loop) , exactly once.

### Relation to other clustering objectives

The LambdaCC objective described in the preceding section generalizes the following popular clustering objectives:

  - If all node weights are equal to 1, the objective becomes a (weighted) [correlation clustering](https://en.wikipedia.org/wiki/Correlation_clustering) objective that the literature commonly studies. Since the implementation sets all node weights to 1 (which is the most commonly studied case), this documentation refers to the algorithm as the *correlation clustering algorithm* (and not as the LambdaCC algorithm).
  - When the node weight of each node is its weighted degree (that is, the sum of the weights of its incident edges) divided by the number of edges in the graph, the objective becomes equivalent to the [modularity](https://en.wikipedia.org/wiki/Modularity_\(networks\)) objective. By equivalent, this means that the two objectives are monotone in each other (and for such equivalent formulas, the algorithm produces exactly the same result). The provided [modularity clustering algorithm](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/modularity-clustering) is thus a thin wrapper around correlation clustering, which sets the node weights appropriately.

## Interface specifications

The following list summarizes the expectations on the graph passed as input to this algorithm. These expectations complement the ones listed in [General assumptions on input graphs](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions) .

>   - [Undirected graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#undirected_graph)
>   - [Edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) : May be negative, zero, or positive.

The output is a non-overlapping clustering, with the best LambdaCC objective that the algorithm achieves for the given resolution parameter \\(r\\).

The following parameters configure the algorithm:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">resolution</code></td>
<td>floating-point number</td>
<td>Controls the granularity of the clustering. Must be finite and non-negative.<br />
<br />
Smaller values tend to lead to larger clusters. When the resolution is zero and all edge weights are positive, the algorithm (with a sufficient number of iterations) finds the connected components of the graph.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">num_iterations</code></td>
<td>integer</td>
<td>Maximum number of outer iterations of the algorithm. Must be finite and positive.<br />
<br />
Larger values tend to lead to higher-quality clusterings (w.r.t. the objective function), but also tend to increase the running time.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">num_inner_iterations</code></td>
<td>integer</td>
<td>Maximum number of inner iterations of the algorithm. Must be finite and positive.<br />
<br />
Larger values tend to lead to higher-quality clusterings (w.r.t. the objective function), but also tend to increase the running time.</td>
</tr>
</tbody>
</table>

## Algorithm

The underlying algorithm is a parallel, asynchronous implementation of the Louvain method.

The algorithm runs a sequence of *outer* iterations (or just *iterations* , for short). In the first outer iteration, the algorithm starts by assigning each node to a cluster of its own. Then, in each *inner* iteration of the algorithm, the algorithm considers each node \\(x\\), in parallel. If moving \\(x\\) to a different cluster increases the LambdaCC objective, the algorithm moves it to a cluster that maximizes the objective increase; otherwise, the algorithm does not move \\(x\\). The algorithm repeats the inner iterations until no nodes move or it reaches the maximum number of inner iterations.

Once the algorithm completes the inner iterations, it *contracts* each cluster to a single node and starts a new outer iteration on the contracted graph, as the preceding section describes—that is, the algorithm assigns each node in the contracted graph to its own cluster, then performs a sequence of inner iterations. The following rules define the contracted graph:

  - The weight of each node in the contracted graph is the total weight of the nodes that formed it.

  - An edge \\(xy\\) exists in the contracted graph if and only if *any* edge existed between a node in the cluster corresponding to \\(x\\) in the original graph and a node in the cluster corresponding to \\(y\\) in the original graph.

  - The weight of an edge \\(xy\\) in the contracted graph is the sum of the weights of all edges that the algorithm contracted into it.

For a node \\(v\\) in the contracted graph corresponding to a cluster \\(c\\) in the original graph, moving \\(v\\) to a new cluster in the contracted graph is equivalent to moving all nodes in \\(c\\) to a new cluster in the original graph.

The outer iterations continue until convergence to a *local* optimum (that is, until the algorithm cannot move any node from its current cluster to a different one at the start of an outer iteration while increasing the objective) or until the algorithm reaches the maximum number of outer iterations.

> **Note:** The algorithm is non-deterministic for the following reasons:
> 
>   - The order in which the algorithm processes nodes within each inner iteration may affect the final clustering.
>   - The algorithm processes nodes in parallel, leading to a non-deterministic ordering.

## Computational complexity

The algorithm is a parallel algorithm that runs in \\(O(\\text{num\_iterations} \\cdot \\text{num\_inner\_iterations}\\cdot m)\\) [work](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#work) and \\(O(\\text{num\_iterations} \\cdot \\text{num\_inner\_iterations}\\cdot \\log n)\\) [depth](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#depth) , where \\(n\\) and \\(m\\) denote the number of nodes and edges in the input graph, respectively, and \\(\\text{num\_iterations}\\) and \\(\\text{num\_inner\_iterations}\\) denote the number of (outer) iterations and inner iterations, respectively. Running the algorithm on a machine that supports high amounts of parallelism tends to lead to a smaller running time.

The algorithm uses \\(O(m + n)\\) memory.

## Examples

> **Note:** These examples use a sequential algorithm for illustration purposes, while in the actual implementation the algorithm executes some parts in parallel.

### Example 1

#### Input graph

Consider the following input graph:

![Undirected weighted graph with ten nodes and weighted edges.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/input_graph.svg)

This example assumes that \\(k\_x = 1\\) (the weight of each node \\(x\\)) for the given graph. The following description assumes that the resolution parameter \\(r\\) is 1.

#### Solution

![The optimal clustering solution showing four clusters: (a), (bcde), (fgh), and (ij).](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/solution.svg)

This is a clustering that optimizes the objective, but in general the algorithm may find a suboptimal solution.

The LambdaCC objective is the sum of LambdaCC increases obtained in the outer iterations:

\\\[\\text{LambdaCC} = \\Delta\\text{LambdaCC}\_1 + \\Delta\\text{LambdaCC}\_2 = 3.5 + 0.1 = 3.6\\\]

#### Algorithm details

Initially, the algorithm assigns each node to a cluster of its own (represented in the diagram with orange dashed lines).

![The initial state of the algorithm where each node is in its own individual cluster.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/initial_state.svg)

\\\[\\text{LambdaCC} = 0\\\]

*Outer iteration \#1 - inner iteration \#1*

The algorithm iterates over all nodes in some order. For each node \\(x\\), the algorithm moves \\(x\\) to a cluster (either the cluster of a neighbor of \\(x\\), or a brand new cluster containing just \\(x\\) itself) chosen such that the increase in the LambdaCC objective is maximized, if there exists such a move that increases the objective.

Recall that the LambdaCC objective is the sum of edge weights within the clusters minus the contribution of the resolution parameter.

> **Note:** The resulting cluster formations depend on the order of best move consideration. This example assumes that the algorithm processes nodes in alphabetical order. In the implementation, this order is non-deterministic.

The following are the possible moves for each node (\\(x\\)) in the left component of the graph and their effect on the LambdaCC objective—where a red arrow indicates no increase, an orange arrow indicates an increase but not the highest one for that node, and a green arrow indicates that node's best move, if there is a move that increases the LambdaCC objective.

Let \\(\\Delta\\text{LambdaCC}\_1\\) denote the overall objective increase achieved in the outer iteration \#1. Initially, \\(\\Delta\\text{LambdaCC}\_1 = 0\\), and the algorithm updates it each time it executes a move.

> *Node (a)* :
> 
> Node (a)'s possible moves, and their effect on the LambdaCC objective:
> 
>   - **(a) \\(\\rightarrow\\) (c):** -0.1 - 0.1 *1* 1 = -0.2
>   - **(a) \\(\\rightarrow\\) (e):** -0.9 - 0.1 *1* 1 = -1.0
> 
> ![Node a's possible moves, merging with either c or e.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/node_a.svg)
> 
> No move increases the LambdaCC objective, so the algorithm commits no move.

> *Node (b)* :
> 
> Node (b)'s possible moves, and their effect on the LambdaCC objective:
> 
>   - **(b) \\(\\rightarrow\\) (c):** 0.9 - 0.1 *1* 1 = 0.8
>   - **(b) \\(\\rightarrow\\) (d):** 0.7 - 0.1 *1* 1 = 0.6
> 
> ![Node b's possible moves, merging with either c or d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/node_b.svg)
> 
> The move to (c) maximizes the increase of the LambdaCC objective.
> 
> \\\[\\Delta\\text{LambdaCC}\_1 = 0 + 0.8 = 0.8\\\]

> *Node (c)* :
> 
> Node (c)'s possible moves, and their effect on the LambdaCC objective:
> 
>   - **split from (b):** -0.9 + 0.1 *1* 1 = -0.8
>   - **(c) \\(\\rightarrow\\) (a):** -0.8 - 0.1 - 0.1 *1* 1 = -1.0
>   - **(c) \\(\\rightarrow\\) (d):** -0.8 - 0.2 - 0.1 *1* 1 = -1.1
> 
> ![Node c's possible moves, splitting from b or merging with either a or d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/node_c.svg)
> 
> No move increases the LambdaCC objective, so the algorithm commits no move.

> *Node (d)* :
> 
> Node (d)'s possible moves and their effect on the LambdaCC objective:
> 
>   - **(d) \\(\\rightarrow\\) (bc):** (0.7-0.2) - 2 *0.1* 1\*1 = 0.3
>   - **(d) \\(\\rightarrow\\) (e):** 0.8 - 0.1 *1* 1 = 0.7
> 
> ![Node d's possible moves, merging with either bc or e.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/node_d.svg)
> 
> The move to (e) maximizes the increase of the LambdaCC objective.
> 
> \\\[\\Delta\\text{LambdaCC}\_1 = 0.8 + 0.7 = 1.5\\\]

> *Node (e)* :
> 
> Node (e)'s possible moves and their effect on the LambdaCC objective:
> 
>   - **split from (d):** - 0.8 + 0.1 *1* 1 = -0.7
>   - **(e) \\(\\rightarrow\\) (a):** - 0.7 - 0.9 - 0.1 *1* 1 = -1.7
> 
> ![Node e's possible moves, either splitting from d or merging with a.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/node_e.svg)
> 
> No move increases the LambdaCC objective, so the algorithm commits no move.

Running similar steps on the right component of the graph, the algorithm executes the moves (f) \\(\\rightarrow\\) (g), (h) \\(\\rightarrow\\) (fg), and (i) \\(\\rightarrow\\) (j), producing clusters (fgh) and (ij). The algorithm updates \\(\\Delta\\text{LambdaCC}\_1\\) to account for those three additional moves:

\\\[\\Delta\\text{LambdaCC}\_1 = 1.5 + (0.6 - 0.1\*1\*1) + (0.8 - 2\*0.1\*1\*1) + (1.0 - 0.1\*1\*1) = 3.5\\\]

Recall that the algorithm repeats the inner iterations until no nodes move or it reaches the maximum number of inner iterations (by default, 10). Hence, a second inner iteration occurs. In this specific case no single-node move increases the objective, and so the algorithm commits no additional move. Since no moves occur in an inner iteration, the algorithm contracts the graph:

![The contracted graph after the first outer iteration, which contains nodes a, bc, de, fgh, and ij.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/contracted_graph_1.svg)

In the contracted graph, the weight of each node is the total weight of the nodes that formed it, and the value next to the node indicates its weight. The letters in each node of the contracted graph correspond to the nodes of the original graph that formed this node in the contracted graph.

The weight of an edge \\(xy\\) in the contracted graph is the sum of the weights of all edges that the algorithm contracted into it.

*Outer iteration \#2 - inner iteration \#1*

Let \\(\\Delta\\text{LambdaCC}\_2\\) denote the overall objective increase achieved in the outer iteration \#2. Initially, \\(\\Delta\\text{LambdaCC}\_2 = 0\\), and the algorithm updates it each time it executes a move.

The algorithm considers each cluster in the contracted graph.

> *Cluster (a)* :
> 
> Cluster (a)'s possible moves, and their effect on the LambdaCC objective:
> 
>   - **(a) \\(\\rightarrow\\) (bc):** -0.1 - 0.1 *1* 2 = -0.3
>   - **(a) \\(\\rightarrow\\) (de):** -0.9 - 0.1 *1* 2 = -1.1
> 
> ![Cluster a's possible moves, merging with either bc or de.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/cluster_a.svg)
> 
> No move increases the LambdaCC objective, so the algorithm commits no move.

> *Cluster (bc)* :
> 
> Cluster (bc)'s possible moves, and their effect on the LambdaCC objective:
> 
>   - **(bc) \\(\\rightarrow\\) (a):** -0.1 - 0.1 *2* 1 = -0.3
>   - **(bc) \\(\\rightarrow\\) (de):** 0.5 - 0.1 *2* 2 = 0.1
> 
> ![Cluster bc's possible moves, merging with either a or de.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/cluster_bc.svg)
> 
> The move to cluster (de) maximizes the increase of the LambdaCC objective.
> 
> \\\[\\Delta\\text{LambdaCC}\_2 = 0 + 0.1 = 0.1\\\]

> *Cluster (de)* :
> 
> Cluster (de)'s possible moves, and their effect on the LambdaCC objective:
> 
>   - **split from (bc):** -0.5 + 0.1 *2* 2 = -0.1
>   - **(de) \\(\\rightarrow\\) (a):** -0.1 - 0.9 - 0.1 *2* 1 = -1.2
> 
> ![Cluster de's possible moves, either splitting from bc or merging with a.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/cluster_de.svg)
> 
> No move increases the LambdaCC objective, so the algorithm commits no move.

Running similar steps on the right component of the graph, which contains nodes (fgh) and (ij), the algorithm does not find any move that improves the LambdaCC objective. Therefore the algorithm commits no additional move.

The algorithm again performs a second inner iteration where no single-node move increases the objective, and so the algorithm commits no move. Since no moves occur in an inner iteration, the algorithm contracts the graph. The total objective increase in outer iteration \#2 was:

\\\[\\Delta\\text{LambdaCC}\_2 = 0.1\\\]

Here's the result of contracting the graph after outer iteration \#2:

![The contracted graph after the second outer iteration, which contains nodes a, bcde, fgh, and ij.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example1/contracted_graph_2.svg)

At this point there is no single node move that would increase the objective, so the algorithm terminates, producing the clusters (a), (bcde), (fgh), and (ij).

### Example 2

This example demonstrates the sensitivity of the clustering outcome to the sequence in which node-level decisions are evaluated.

For the given graph, assume \\(k\_x\\) to be 1 (the weight of node x) and \\(r\\) to be 0.9 (the resolution parameter).

**Initial graph**

![An undirected graph with three nodes and weighted edges.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example2/initial_graph.svg)

**Algorithm start**

![Each node starts in its own cluster, denoted by dashed orange rectangles around each individual node.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example2/algorithm_start.svg)

Initially, the algorithm assigns each node to a cluster of its own (represented in the diagram with orange dashed lines). *LambdaCC = 0*

**Inner iteration \#1**

The algorithm iterates over all nodes in some order. For each node \\(x\\), the algorithm moves \\(x\\) to a cluster chosen such that the increase in the LambdaCC objective is maximized, if such a move exists.

In this example, assume the algorithm first considers node (a). The best move for (a) is to join the cluster of (b). Since this improves the objective, the algorithm performs it.

![Node a joins node b's cluster, while node c remains in its own cluster.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example2/iter1_a.svg)

Once (a) is moved to (b), the LambdaCC score becomes: \*LambdaCC = 1.0 - 0.9 \* 1 \* 1 = 0.1\*

At this point, moving (c) to the same cluster would change the LambdaCC objective by a negative amount (1.0 - 2 *0.9* 1\*1 = -0.8), so the algorithm does not perform the move.

**Alternative outcome**

If the algorithm had chosen to move node (c) first, it would have obtained a different (but equally optimal) clustering:

![Node c joins node b's cluster, while node a remains in its own cluster.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/clustering/images/correlation-clustering/example2/iter1_b.svg)

## External references

  - The paper [A Correlation Clustering Framework for Community Detection](https://dl.acm.org/doi/10.1145/3178876.3186110) introduced the LambdaCC objective.
  - The paper [Scalable Community Detection via Parallel Correlation Clustering](https://arxiv.org/abs/2108.01731) introduced the correlation clustering algorithm.
  - The paper [The ParClusterers Benchmark Suite (PCBS): A Fine-Grained Analysis of Scalable Graph Clustering](https://arxiv.org/abs/2411.10290) benchmarked the algorithm.
  - You can find an open-source implementation on [GitHub](https://github.com/google/graph-mining/tree/main/in_memory/clustering/correlation) .
