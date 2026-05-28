---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/betweenness-centrality
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/betweenness-centrality
title: Betweenness centrality
description: Learn about the betweenness centrality algorithm. Identify nodes that act as bridges or bottlenecks in your graph, essential for finding influential entities in a network.
data_source: docs.cloud.google.com
---

This algorithm computes the betweenness centralities of all nodes of a [directed graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#directed_graph) with positive [edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) , defined as follows. Let \\(V\\) be the set of all nodes in the graph. For any pair of distinct nodes \\(x\\) and \\(y\\), define the following notation:

  - \\(P(x, y)\\) is the set of all [shortest paths](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#shortest_path) from node \\(x\\) to node \\(y\\).
  - \\(\\delta\_{xy}(z)\\), where \\(z\\) is a node other than \\(x\\) and \\(y\\), is the *pair-dependency* of nodes \\(x\\) and \\(y\\) on \\(z\\). This quantity represents the fraction of shortest paths from \\(x\\) to \\(y\\) that pass through \\(z\\) (or zero if \\(x\\) is not reachable from \\(y\\)):

\\\[ \\delta\_{xy}(z) = \\begin{cases} \\frac{|\\{p \\in P(x, y) : z \\in p\\}|}{|P(x, y)|} & \\text{, if } y \\text{ is reachable from } x ;\\\\ 0 & \\text{, otherwise.}\\\\ \\end{cases} \\\]

The betweenness centrality of node \\(z\\) is the sum of the pair-dependencies of all pairs of nodes (other than \\(z\\) itself) on \\(z\\):

\\\[ \\text{betweenness\_centrality}(z) = \\sum\_{x, y \\in V \\setminus \\{z\\} : x \\neq y} \\delta\_{xy}(z). \\\]

Note that the betweenness centrality of a node is always between \\(0\\) and \\((|V| - 1) \\cdot (|V| - 2)\\), inclusive.

The algorithm also supports an approximate, source-constrained mode, where it computes pair-dependencies only for source nodes \\(x\\) in some set of source nodes \\(S\\) (a subset of \\(V\\)) that the algorithm samples:

\\\[ \\text{betweenness\_centrality}\_S(z) = \\sum\_{x \\in S \\setminus \\{z\\} , y \\in V \\setminus \\{x, z\\}} \\delta\_{xy}(z). \\\]

Note that the approximate betweenness centrality of a node is always between \\(0\\) and \\(|S| \\cdot (|V| - 2)\\), inclusive. Moreover, the exact betweenness centrality is a special case of the approximate betweenness centrality with \\(S\\) equal to \\(V\\).

## Interface specifications

The following list summarizes the expectations on the graph passed as input to this algorithm. These expectations complement the ones listed in [General assumptions on input graphs](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions) .

>   - [Directed graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#directed_graph) (an undirected graph can be converted into a directed graph by replacing each undirected edge \\(xy\\) with two directed edges \\(xy\\) and \\(yx\\), both with the same weight as the original edge).
>   - [Edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) : Must be positive. The output is a list of \\(|V|\\) floating-point numbers, corresponding to the betweenness centralities of all nodes.

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
<td><code dir="ltr" translate="no">num_source_nodes</code></td>
<td>integer (optional)</td>
<td>If set, must be positive, and specifies how many source nodes the algorithm should select for computing approximate betweenness centralities. If not set, or set to a number larger than the number of nodes in the graph, then the algorithm uses all nodes as source nodes, that is, the algorithm computes exact centralities.<br />
<br />
Higher values lead to better approximations, but also to longer running times.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">sampling_seed</code></td>
<td>integer (optional)</td>
<td>The seed for the random number generator that the algorithm uses to select the source nodes (if <code dir="ltr" translate="no">num_source_nodes</code> is set). The algorithm selects the same set of source nodes when given the same input graph, <code dir="ltr" translate="no">num_source_nodes</code> , and <code dir="ltr" translate="no">sampling_seed</code> .</td>
</tr>
</tbody>
</table>

## Algorithm

The algorithm starts by selecting which nodes it uses as source nodes. If `num_source_nodes` is not set, then the algorithm considers all nodes source nodes. If `num_source_nodes` is set, then the algorithm attempts to select this many source nodes, using reservoir sampling (without replacement). At each sampling step, the probability of selecting a given node that the algorithm has not yet selected is proportional to its [out-degree](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#out_degree) . The algorithm never selects nodes with [out-degree](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#out_degree) zero. If the number of nodes with positive out-degree is smaller than `num_source_nodes` (in particular, this happens if `num_source_nodes` is larger than the number of nodes in the graph), then the algorithm will use all nodes with positive out-degree as source nodes (note that this means that the algorithm will compute exact betweenness centralities).

For each source node \\(x\\) (in parallel), the algorithm then runs a parallel single-source shortest path (SSSP) algorithm to compute the shortest-path distances \\(d(x, y)\\) from \\(x\\) to all other nodes \\(y\\). Then it computes the number of shortest paths from \\(x\\) to any other node \\(y\\), by observing that \\(|P(x, x)| = 1\\) and for any node \\(y \\neq x\\) that can be reached from \\(x\\), the following recurrence relation holds:

\\\[ |P(x, y)| = \\sum\_{z \\in V : zy \\in E, d(x, z) + w\_{zy} = d(x, y)} |P(x, z)| , \\\]

where \\(w\_{zy}\\) denotes the weight of edge \\(zy\\).

The algorithm then computes the contribution of the source node \\(x\\) to the betweenness centrality of each other node \\(z\\), namely

\\\[ \\delta\_{x\\odot}(z) := \\sum\_{y \\in V \\setminus \\{x, z\\} } \\delta\_{xy}(z) , \\\]

using the fact that \\(\\delta\_{x\\odot}(x) = 0\\) and for any other node \\(z \\neq x\\) that can be reached from \\(x\\), the following recurrence relation holds:

\\\[ \\delta\_{x\\odot}(z) = \\sum\_{y \\in V : zy \\in E, d(x, z) + w\_{zy} = d(x, y)} \\frac{|P(x,z)|}{|P(x,y)|}( 1+ \\delta\_{x\\odot}(y) ) . \\\]

Finally, the algorithm adds up the contributions from all source nodes to obtain the final (exact or approximate) betweenness centralities.

Because the edges in the input graph have positive weights, the preceding recurrence relations don't introduce cyclic dependencies.

> **Note:** while the sets of paths \\(P(x, y)\\) that the definitions of betweenness centrality use may have exponential size, the algorithm runs in polynomial time, and does not explicitly compute those sets of paths; it computes only their cardinalities, using the recurrence relation explained in the previous section.

## Computational complexity

The algorithm is a parallel algorithm that runs in \\(O(sm\\log n)\\) [work](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#work) and \\(O(m\\log n)\\) [depth](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#depth) , where \\(n\\) and \\(m\\) denote the number of nodes and edges in the input graph, respectively, and \\(s\\) denotes the number of source nodes that the algorithm uses (which is at most `num_source_nodes` if that parameter is set, or \\(n\\) otherwise).

The algorithm uses \\(O(\\text{num\_threads} \\cdot n)\\) memory, where \\(\\text{num\_threads}\\) is the number of threads that the algorithm uses during execution.

## Example

This example illustrates the computation of the exact betweenness centrality for a specific node.

### Input graph

Consider the following input graph:

![A directed graph with seven nodes (a-g) and uniform edge weights.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/input_graph.svg)

For the provided graph, assume that the weight of each edge is 1.0.

### Solution

Betweenness centrality scores for each node x in the graph, denoted \\(BC\_x\\):

  - \\(BC\_a\\) = 1.0, \\(BC\_b\\) = 8.0, \\(BC\_c\\) = 6.5, \\(BC\_d = 9.5\\)
  - \\(BC\_e\\) = 8.0, \\(BC\_f\\) = 5.0, \\(BC\_g\\) = 0.0

### Algorithm details

Recall that betweenness centrality measures a node's centrality by determining how often it lies on the shortest paths between all other pairs of distinct nodes in the graph.

Instead of detailing all the calculations for all the nodes and respective pairs, this section focuses on a specific example to illustrate the algorithm's process. Assume the goal is to calculate the betweenness centrality for node \\(d\\) (colored in yellow), denoted by \\(BC\_d\\).

![The input graph highlighting node d in yellow, for which the algorithm calculates the betweenness centrality.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/node_d.svg)

If all the shortest paths between node \\(x\\) and node \\(y\\) bypass node \\(d\\), it means node \\(d\\) is not needed as an intermediary for that specific connection. Consequently, the pair-dependency of the pair \\((x, y)\\) on node \\(d\\) is 0.

For example, tracing all the shortest paths between pairs of nodes like \\((a, b)\\), \\((a, e)\\), or \\((e, g)\\) reveals that node \\(d\\) is not on any of them. That means those pairs of nodes don't contribute to node \\(d\\)'s betweenness centrality. In this particular example, there are 19 such pairs.

Formally, the pair dependency of a pair \\((x, y)\\) on \\(d\\) is the total number of shortest paths between \\((x, y)\\) passing through \\(d\\), divided by the total number of shortest paths between \\((x, y)\\).

Consider the pairs of nodes for which the pair-dependency on node \\(d\\) is not 0:

*pair (a, f)* : There are two shortest paths, one of which passes through node \\(d\\) (solid dark blue edges) and one of which doesn't (dashed light blue edges):

![The graph showing shortest paths between node a and f, highlighting the one passing through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_a_f.svg)

The pair-dependency is therefore 0.5 (that is, 1/2).

*pair (a, g)* : Again, there are two shortest paths, one of which passes through node \\(d\\):

![The graph showing shortest paths between node a and g, highlighting the one passing through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_a_g.svg)

The pair-dependency is therefore 0.5 (that is, 1/2).

*pair (b, e)* : Here again there are two shortest paths, one of which passes through node \\(d\\):

![The graph showing shortest paths between node b and e, highlighting the one passing through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_b_e.svg)

The pair-dependency is therefore 0.5 (that is, 1/2).

*pair (b, f)* : In this case there is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node b and f, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_b_f.svg)

The pair-dependency is thus 1.0 (that is, 1/1).

*pair (b, g)* : There is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node b and g, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_b_g.svg)

The pair-dependency is thus 1.0 (that is, 1/1).

*pair (e, a)* : There is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node e and a, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_e_a.svg)

The pair-dependency is 1.0 (that is, 1/1).

*pair (e, b)* : There is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node e and node b, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_e_b.svg)

The pair-dependency is 1.0 (that is, 1/1).

*pair (e, c)* : There is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node e and node c, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_e_c.svg)

The pair-dependency is 1.0 (that is, 1/1).

*pair (f, a)* : There is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node f and node a, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_f_a.svg)

The pair-dependency is 1.0 (that is, 1/1).

*pair (f, b)* : There is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node f and node b, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_f_b.svg)

The pair-dependency is 1.0 (that is, 1/1).

*pair (f, c)* : There is a unique shortest path, and it passes through node \\(d\\):

![The unique shortest path between node f and node c, which passes through node d.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/betweenness-centrality/example1/pair_f_c.svg)

The pair-dependency is 1.0 (that is, 1/1).

Finally, summing the pair-dependencies of the 11 node pairs on node \\(d\\) yields:

\\\[BC\_d = (3 \* 0.5) + (8 \* 1.0) = 9.5\\\]

A similar computation yields the betweenness centralities of all other nodes.

## External references

  - The algorithm is based on Brandes' algorithm, introduced in [A Faster Algorithm for Betweenness Centrality](https://www.tandfonline.com/doi/abs/10.1080/0022250x.2001.9990249) (Journal of Mathematical Sociology, 2001). In particular, that paper introduced the recurrence relations.
  - While Brandes' algorithm is a sequential algorithm, the implementation described in this page leverages parallelism, following the approach from [Ligra: A Lightweight Graph Processing Framework for Shared Memory](https://dl.acm.org/doi/abs/10.1145/2442516.2442530) (Principles and Practice of Parallel Programming, 2013).
  - An earlier version of the algorithm, which handles the single-source case only and requires the graph to be unweighted, was described and benchmarked in [Theoretically Efficient Parallel Graph Algorithms Can Be Fast and Scalable](https://ldhulipala.github.io/papers/gbbs_topc.pdf) (Symposium on Parallelism in Algorithms and Architectures, 2018). An open-source implementation of that earlier version is available on the [Graph Based Benchmark Suite (GBBS)](https://paralg.github.io/gbbs/) .
