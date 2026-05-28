---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/pagerank
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/node-centrality/pagerank
title: PageRank
description: Learn about the PageRank algorithm. Measure the importance of each node in a directed graph based on the quantity and quality of links to it.
data_source: docs.cloud.google.com
---

This algorithm computes the approximate PageRank centralities of all nodes of a weighted [directed graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#directed_graph) with non-negative [edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) . Those centralities provide a ranking of the nodes based on the underlying link structure of the graph.

Originally, the PageRank algorithm was used to rank Web pages for Web search indexing. The computation was based on the idea of a "random surfer" who randomly clicks on links. The PageRank score of a page is proportional to the probability that the random surfer will land on that page.

Formally, the PageRank centralities of nodes in a graph are defined as follows. Let \\(S\\) be a nonempty set of source nodes chosen by the user; by default, \\(S\\) consists of all nodes of the graph. Consider a random walk which starts at a node of \\(S\\) selected uniformly at random, and at each step does one of these actions:

  - If the current node has at least one outgoing edge with positive weight, do one of two actions:
      - With probability \\(\\gamma\\), select one of those outgoing edges and traverse it (that is, move to the other endpoint of the selected edge). The outgoing edge to be traversed is selected at random; for each of the candidate outgoing edges, its probability of being selected is proportional to its weight.
      - With probability \\(1 - \\gamma\\), move to one of the nodes in \\(S\\), which the algorithm chooses uniformly at random.
  - Otherwise (that is, if all outgoing edges of the current node have weight zero): move to one of the nodes in \\(S\\), which the algorithm chooses uniformly at random.

Here, \\(\\gamma \\in \[0, 1)\\) is a parameter of the algorithm, called the *damping factor* . The (exact) PageRank centrality of a node is defined as the limit as \\(s\\) goes to infinity of the probability that such a random walk with \\(s\\) steps ends at that node. Note that the PageRank centrality is defined (and computed by the algorithm) for all nodes in the graph, not just those in the set \\(S\\) of source nodes.

## Interface specifications

The following list summarizes the expectations on the graph passed as input to this algorithm. These expectations complement the ones listed in [General assumptions on input graphs](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions) .

>   - [Directed graph](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#directed_graph) (an undirected graph can be converted into a directed graph by replacing each undirected edge \\(xy\\) with two directed edges \\(xy\\) and \\(yx\\), both with the same weight as the original edge).
>   - [Edge weights](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#edge_weights) : Must be non-negative.

The output is a list of \\(n\\) floating-point numbers (where \\(n\\) denotes the number of nodes in the input graph), corresponding to the approximate PageRank centralities of all nodes (not just those in \\(S\\)).

These parameters configure the algorithm:

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
<td><code dir="ltr" translate="no">damping_factor</code></td>
<td>floating-point number</td>
<td>The probability that, at any given iteration, the algorithm chooses to traverse one of the outgoing edges of the current node (if the node has at least one outgoing edge with positive weight). Must be in the range [0, 1).<br />
<br />
This parameter is usually set to 0.85 in research papers related to PageRank.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">num_iterations</code></td>
<td>integer</td>
<td>The maximum number of iterations of the algorithm. Must be positive.<br />
<br />
The running time of the algorithm scales linearly with the number of iterations executed; a higher number of iterations tends to produce more precise results.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">approx_precision</code></td>
<td>floating-point number</td>
<td>Approximation precision threshold for the algorithm. Must be non-negative. The algorithm stops if the L1-distance between the approximate centrality vector that the algorithm obtains in two consecutive iterations is smaller than <code dir="ltr" translate="no">approx_precision</code> times the number of nodes in the graph.<br />
<br />
Smaller values of <code dir="ltr" translate="no">approx_precision</code> tend to lead to more precise results, but also tend to increase the running time.</td>
</tr>
</tbody>
</table>

## Algorithm

The algorithm executes at most `num_iterations` (possibly fewer than that, if the algorithm achieves convergence earlier—see the documentation of the `approx_precision` parameter). The \\(i\\)-th iteration computes the probability distribution for the location of the random walk after \\(i\\) steps.

The algorithm uses the following equations to compute those distributions. Let \\(P\_i(v)\\) (where \\(i = 0, 1, 2...\\)) denote the probability that the random walk reaches node \\(v\\) after \\(i\\) steps.

For \\(i = 0\\), since the algorithm chooses the starting point of the random walk as a source node uniformly at random:

\\\[ P\_0(y) = \\begin{cases} \\frac{1}{|S|} & \\text{, if } y \\in S ;\\\\ 0 & \\text{, otherwise.} \\\\ \\end{cases} \\\]

For \\(i \\ge 1\\), the algorithm derives the \\(P\_i\\) values from the \\(P\_{i-1}\\) values using a recurrence relation.

Let:

  - \\(V\\) denote the set of nodes of the graph;
  - \\(\\deg^+(x)\\) denote the weighted out-degree of a node \\(x\\) (that is, the sum of the weights of \\(x\\)'s outgoing edges);
  - \\(V\_{\\textrm{sink}}\\) denote the set of nodes of the graph whose weighted out-degree is \\(0\\);
  - \\(E\\) denote the set of the (directed) edges in the graph;
  - \\(w\_{xy}\\) denote the weight of an edge \\(xy\\);
  - \\(N^-(y)\\) denote the set of nodes \\(x\\) such that there is an edge \\(xy\\) with positive weight.

The algorithm computes the \\(P\_i\\) values by this relation:

\\\[ P\_i(y) = \\begin{cases} \\gamma \\sum\_{x \\in N^-(y)} \\frac{w\_{xy}P\_{i-1}(x)}{\\deg^+(x)} + \\frac{(1 - \\gamma) + \\gamma\\sum\_{x \\in V\_{\\textrm{sink}}}P\_{i-1}(x)}{|S|} & \\text{, if } y \\in S ;\\\\ \\gamma \\sum\_{x \\in N^-(y)} \\frac{w\_{xy}P\_{i-1}(x)}{\\deg^+(x)} & \\text{, otherwise.} \\\\ \\end{cases}\\\]

## Computational complexity

The algorithm is a parallel algorithm that runs in \\(O(\\text{num\_iterations} \\cdot (m + n))\\) [work](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#work) and \\(O(\\text{num\_iterations} \\cdot \\log n)\\) [depth](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary#depth) , where \\(n\\) and \\(m\\) denote the number of nodes and edges in the input graph, respectively, and \\(\\text{num\_iterations}\\) denotes the number of iterations. Running the algorithm on a machine that supports high amounts of parallelism tends to lead to a smaller running time.

The algorithm uses \\(O(n)\\) memory.

## Examples

### Example 1

#### Input graph

Consider this input graph:

![A directed weighted graph with four nodes (a-d) and weighted edges.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example1/input_graph.svg)

For the provided directed weighted graph, assume that:

  - A number over an edge indicates its weight.
  - The damping factor \\(\\gamma\\) is 0.85.
  - The number of iterations is 3.
  - The approximate precision is 0.01.
  - No source nodes are provided in the input (that is, the algorithm treats all nodes as source nodes).

#### Solution

PageRank scores that the algorithm obtains after three iterations for each node \\(x\\) in the graph, denoted \\(P\_3(x)\\):

  - \\(P\_3(a) \\approx 0.066\\), \\(P\_3(b) \\approx 0.407\\), \\(P\_3(c) \\approx 0.138\\), \\(P\_3(d) \\approx 0.389\\)

Note that the exact PageRank scores are:

  - \\(P(a) \\approx 0.067\\), \\(P(b) \\approx 0.414\\), \\(P(c) \\approx 0.137\\), \\(P(d) \\approx 0.382\\)

#### Algorithm details

Since no source nodes were provided in the input, the set of source nodes consists of all nodes: \\(S = \\{a, b, c, d\\}\\). Initially, the algorithm assigns a score \\(P\_0(x) = 1 / |S| = 0.25\\) to each node.

  - \\(P\_0(a) = 0.25\\), \\(P\_0(b) = 0.25\\), \\(P\_0(c) = 0.25\\), \\(P\_0(d) = 0.25\\)

The weighted out-degrees of the nodes are:

  - \\(deg^+(a) = 2 + 3= 5\\), \\(deg^+(b) = 4 + 1 = 5\\), \\(deg^+(c) = 0\\), \\(deg^+(d) = 2\\)

Since only node \\(c\\) has a weighted out-degree of 0, \\(V\_{sink} = \\{c\\}\\).

*Recurrence relation for node \\(a\\)* :

![Incoming edges to node a highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example1/node_a.svg)

Nodes that have a positive-weight edge pointing to node a: \\(N^-(a) = \\{\\}\\)

\\\[ \\begin{align\*} P\_i(a) &= \[ (1 - \\gamma) + \\gamma \* P\_{i-1}(c) \] / |S| \\\\ &= \[ 0.15 + 0.85 \* P\_{i-1}(c) \] / 4 \\end{align\*} \\\]

*Recurrence relation for node \\(b\\)* :

![Incoming edges to node b highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example1/node_b.svg)

Nodes that have a positive-weight edge pointing to node \\(b\\): \\(N^-(b) = \\{a, d\\}\\)

\\\[ \\begin{align\*} P\_i(b) &= \\gamma \* \[ P\_{i-1}(a) w\_{ab} / deg^+(a) + P\_{i-1}(d) w\_{db} / deg^+(d)\] + \[ (1 - \\gamma) + \\gamma \* P\_{i-1}(c) \] / |S| \\\\ &= 0.85 \* \[ 0.4 \* P\_{i-1}(a) + P\_{i-1}(d)\] + \[ 0.15 + 0.85 \* P\_{i-1}(c) \] / 4 \\end{align\*} \\\]

*Recurrence relation for node \\(c\\)* :

![Incoming edges to node c highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example1/node_c.svg)

Nodes that have a positive-weight edge pointing to node \\(c\\): \\(N^-(c) = \\{b\\}\\)

\\\[ \\begin{align\*} P\_i(c) &= \\gamma \* \[ P\_{i-1}(b) w\_{bc} / deg^+(b)\] + \[ (1 - \\gamma) + \\gamma \* P\_{i-1}(c) \] / |S| \\\\ &= 0.85 \* 0.2 \* P\_{i-1}(b) + \[ 0.15 + 0.85 \* P\_{i-1}(c) \] / 4 \\end{align\*} \\\]

*Recurrence relation for node \\(d\\)* :

![Incoming edges to node d highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example1/node_d.svg)

Nodes that have a positive-weight edge pointing to node \\(d\\): \\(N^-(d) = \\{a, b\\}\\)

\\\[ \\begin{align\*} P\_i(d) &= \\gamma \* \[ P\_{i-1}(a) w\_{ad} / deg^+(a) + P\_{i-1}(b) w\_{bd} / deg^+(b) \] + \[ (1 - \\gamma) + \\gamma \* P\_{i-1}(c) \] / |S| \\\\ &= 0.85 \* \[ 0.6 \* P\_{i-1}(a) + 0.8 \* P\_{i-1}(b) \] + \[ 0.15 + 0.85 \* P\_{i-1}(c) \] / 4 \\end{align\*} \\\]

*Iteration \#1* :

The algorithm updates the PageRank values using the recurrence relations.

\\\[ \\begin{align\*} P\_1(a) &= \[ 0.15 + 0.85 \* P\_0(c) \] / 4 \\approx 0.091 \\\\ P\_1(b) &= 0.85 \* \[ 0.4 \* P\_0(a) + P\_0(d)\] + \[ 0.15 + 0.85 \* P\_0(c) \] / 4 \\approx 0.388 \\\\ P\_1(c) &= 0.85 \* 0.2 \* P\_0(b) + \[ 0.15 + 0.85 \* P\_0(c) \] / 4 \\approx 0.133 \\\\ P\_1(d) &= 0.85 \* \[ 0.6 \* P\_0(a) + 0.8 \* P\_0(b) \] + \[ 0.15 + 0.85 \* P\_0(c) \] / 4 \\approx 0.388 \\end{align\*} \\\]

The L1-distance between the initial PageRank values and the values from iteration \#1 is:

\\\[|P\_1(a) - P\_0(a)| + |P\_1(b) - P\_0(b)| + |P\_1(c) - P\_0(c)| + |P\_1(d) - P\_0(d)| \\approx 0.553\\\]

This is larger than 0.01 (the approximate precision in the configuration) x 4 (the number of nodes in the graph), so the algorithm proceeds to the next iteration.

*Iteration \#2* :

The algorithm updates the PageRank values using the recurrence relations.

\\\[ \\begin{align\*} P\_2(a) &= \[ 0.15 + 0.85 \* P\_1(c) \] / 4 \\approx 0.066 \\\\ P\_2(b) &= 0.85 \* \[ 0.4 \* P\_1(a) + P\_1(d)\] + \[ 0.15 + 0.85 \* P\_1(c) \] / 4 \\approx 0.427 \\\\ P\_2(c) &= 0.85 \* 0.2 \* P\_1(b) + \[ 0.15 + 0.85 \* P\_1(c) \] / 4 \\approx 0.132 \\\\ P\_2(d) &= 0.85 \* \[ 0.6 \* P\_1(a) + 0.8 \* P\_1(b) \] + \[ 0.15 + 0.85 \* P\_1(c) \] / 4 \\approx 0.376 \\\\ \\end{align\*} \\\]

The L1-distance between the PageRank values from iteration \#1 and iteration \#2 is:

\\\[|P\_2(a) - P\_1(a)| + |P\_2(b) - P\_1(b)| + |P\_2(c) - P\_1(c)| + |P\_2(d) - P\_1(d)| \\approx 0.077\\\]

This is larger than 0.01 (the approximate precision in the configuration) x 4 (the number of nodes in the graph), so the algorithm proceeds to the next iteration.

*Iteration \#3* :

The algorithm updates the PageRank values using the recurrence relations.

\\\[ \\begin{align\*} P\_3(a) &= \[ 0.15 + 0.85 \* P\_2(c) \] / 4 \\approx 0.066 \\\\ P\_3(b) &= 0.85 \* \[ 0.4 \* P\_2(a) + P\_2(d)\] + \[ 0.15 + 0.85 \* P\_2(c) \] / 4 \\approx 0.407 \\\\ P\_3(c) &= 0.85 \* 0.2 \* P\_2(b) + \[ 0.15 + 0.85 \* P\_2(c) \] / 4 \\approx 0.138 \\\\ P\_3(d) &= 0.85 \* \[ 0.6 \* P\_2(a) + 0.8 \* P\_2(b) \] + \[ 0.15 + 0.85 \* P\_2(c) \] / 4 \\approx 0.389 \\end{align\*} \\\]

The algorithm stops because the configuration specifies a maximum of 3 iterations.

### Example 2

#### Input graph

Consider the following input graph:

![Directed weighted graph with four nodes (a-d) and highlighted source nodes a and c.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example2/input_graph.svg)

This is like Example 1, but using a subset of the graph's nodes as sources.

For the provided directed weighted graph, assume that:

  - A number over an edge indicates its weight.
  - The damping factor \\(\\gamma\\) is 0.85.
  - The number of iterations is 3.
  - The approximate precision is 0.01.
  - The set of source nodes given in the input is \\(S = \\{a, c\\}\\) (shown in dark red).

#### Solution

PageRank scores that the algorithm obtains after three iterations for each node \\(x\\) in the graph, denoted \\(P\_3(x)\\):

  - \\(P\_3(a) \\approx 0.171\\), \\(P\_3(b) \\approx 0.290\\), \\(P\_3(c) \\approx 0.225\\), \\(P\_3(d) \\approx 0.314\\)

Note that the exact PageRank scores are:

  - \\(P(a) \\approx 0.169\\), \\(P(b) \\approx 0.311\\), \\(P(c) \\approx 0.222\\), \\(P(d) \\approx 0.298\\)

#### Algorithm details

Initially, the algorithm assigns a score \\(P\_0(x) = 1 / |S| = 0.5\\) to each of the source nodes \\(\\{a, c\\}\\), and a score of 0 to the remaining nodes.

  - \\(P\_0(a) = 0.5\\), \\(P\_0(b) = 0.0\\), \\(P\_0(c) = 0.5\\), \\(P\_0(d) = 0.0\\)

The weighted out-degrees of the nodes are:

  - \\(deg^+(a) = 2 + 3= 5\\), \\(deg^+(b) = 4 + 1 = 5\\), \\(deg^+(c) = 0\\), \\(deg^+(d) = 2\\)

The set of nodes with weighted out-degree equal to zero is \\(V\_{sink} = \\{c\\}\\).

In each iteration \\(i\\), the algorithm computes the PageRank values \\(P\_i(x)\\) for each node \\(x\\), using the values \\(P\_{i-1}(x)\\) from the previous iteration, using the following recurrence relations.

*Recurrence relation for node \\(a\\)* :

![Incoming edges to node a highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example2/node_a.svg)

Nodes that have a positive-weight edge pointing to node a: \\(N^-(a) = \\{\\}\\)

\\\[ \\begin{align\*} P\_i(a) &= \[ (1 - \\gamma) + \\gamma \* P\_{i-1}(c) \] / |S| \\\\ &= \[ 0.15 + 0.85 \* P\_{i-1}(c) \] / 2 \\end{align\*} \\\]

*Recurrence relation for node \\(b\\)* :

![Incoming edges to node b highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example2/node_b.svg)

Nodes that have a positive-weight edge pointing to node \\(b\\): \\(N^-(b) = \\{a, d\\}\\)

\\\[ \\begin{align\*} P\_i(b) &= \\gamma \* \[ P\_{i-1}(a) w\_{ab} / deg^+(a) + P\_{i-1}(d) w\_{db} / deg^+(d)\] \\\\ &= 0.85 \* \[ 0.4 \* P\_{i-1}(a) + P\_{i-1}(d)\] \\end{align\*} \\\]

*Recurrence relation for node \\(c\\)* :

![Incoming edges to node c highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example2/node_c.svg)

Nodes that have a positive-weight edge pointing to node \\(c\\): \\(N^-(c) = \\{b\\}\\)

\\\[ \\begin{align\*} P\_i(c) &= \\gamma \* \[ P\_{i-1}(b) w\_{bc} / deg^+(b)\] + \[ (1 - \\gamma) + \\gamma \* P\_{i-1}(c) \] / |S| \\\\ &= 0.85 \* 0.2 \* P\_{i-1}(b) + \[ 0.15 + 0.85 \* P\_{i-1}(c) \] / 2 \\end{align\*} \\\]

*Recurrence relation for node \\(d\\)* :

![Incoming edges to node d highlighted for the recurrence relation.](https://docs.cloud.google.com/static/spanner/docs/reference/graph-algorithms/node-centrality/images/pagerank/example2/node_d.svg)

Nodes that have a positive-weight edge pointing to node \\(d\\): \\(N^-(d) = \\{a, b\\}\\)

\\\[ \\begin{align\*} P\_i(d) &= \\gamma \* \[ P\_{i-1}(a) w\_{ad} / deg^+(a) + P\_{i-1}(b) w\_{bd} / deg^+(b) \] \\\\ &= 0.85 \* \[ 0.6 \* P\_{i-1}(a) + 0.8 \* P\_{i-1}(b) \] \\end{align\*} \\\]

*Iteration \#1* :

The algorithm updates the PageRank values using the recurrence relations.

\\\[ \\begin{align\*} P\_1(a) &= \[ 0.15 + 0.85 \* P\_0(c) \] / 2 \\approx 0.288 \\\\ P\_1(b) &= 0.85 \* \[ 0.4 \* P\_0(a) + P\_0(d)\] \\approx 0.17 \\\\ P\_1(c) &= 0.85 \* 0.2 \* P\_0(b) + \[ 0.15 + 0.85 \* P\_0(c) \] / 2 \\approx 0.288 \\\\ P\_1(d) &= 0.85 \* \[ 0.6 \* P\_0(a) + 0.8 \* P\_0(b) \] \\approx 0.255 \\end{align\*} \\\]

The L1-distance between the initial PageRank values and the values from iteration \#1 is:

\\\[|P\_1(a) - P\_0(a)| + |P\_1(b) - P\_0(b)| + |P\_1(c) - P\_0(c)| + |P\_1(d) - P\_0(d)| = 0.85\\\]

This is larger than 0.01 (the approximate precision in the configuration) x 4 (the number of nodes in the graph), so the algorithm proceeds to the next iteration.

*Iteration \#2* :

The algorithm updates the PageRank values using the recurrence relations.

\\\[ \\begin{align\*} P\_2(a) &= \[ 0.15 + 0.85 \* P\_1(c) \] / 2 \\approx 0.197 \\\\ P\_2(b) &= 0.85 \* \[ 0.4 \* P\_1(a) + P\_1(d)\] \\approx 0.314 \\\\ P\_2(c) &= 0.85 \* 0.2 \* P\_1(b) + \[ 0.15 + 0.85 \* P\_1(c) \] / 2 \\approx 0.226 \\\\ P\_2(d) &= 0.85 \* \[ 0.6 \* P\_1(a) + 0.8 \* P\_1(b) \] \\approx 0.262 \\\\ \\end{align\*} \\\]

The L1-distance between the PageRank values from iteration \#1 and iteration \#2 is:

\\\[|P\_2(a) - P\_1(a)| + |P\_2(b) - P\_1(b)| + |P\_2(c) - P\_1(c)| + |P\_2(d) - P\_1(d)| \\approx 0.303\\\]

This is larger than 0.01 (the approximate precision in the configuration) x 4 (the number of nodes in the graph), so the algorithm proceeds to the next iteration.

*Iteration \#3* :

The algorithm updates the PageRank values using the recurrence relations.

\\\[ \\begin{align\*} P\_3(a) &= \[ 0.15 + 0.85 \* P\_2(c) \] / 2 \\approx 0.171 \\\\ P\_3(b) &= 0.85 \* \[ 0.4 \* P\_2(a) + P\_2(d)\] \\approx 0.290 \\\\ P\_3(c) &= 0.85 \* 0.2 \* P\_2(b) + \[ 0.15 + 0.85 \* P\_2(c) \] / 2 \\approx 0.225 \\\\ P\_3(d) &= 0.85 \* \[ 0.6 \* P\_2(a) + 0.8 \* P\_2(b) \] \\approx 0.314 \\end{align\*} \\\]

The algorithm stops because the configuration specifies a maximum of 3 iterations.

## External references

  - PageRank centrality was introduced in [The Anatomy of a Large-Scale Hypertextual Web Search Engine](https://www.sciencedirect.com/science/article/abs/pii/S016975529800110X) (Computer Networks, 1998).
  - An earlier version of the algorithm, which was restricted to unweighted, undirected graphs, was benchmarked in [Theoretically Efficient Parallel Graph Algorithms Can Be Fast and Scalable](https://arxiv.org/abs/1805.05208) (Symposium on Parallelism in Algorithms and Architectures, 2018).
