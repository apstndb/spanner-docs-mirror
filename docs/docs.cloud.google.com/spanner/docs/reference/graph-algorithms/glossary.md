---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/glossary
title: Graph algorithms glossary
description: Glossary of graph terms used in the graph algorithms documentation.
data_source: docs.cloud.google.com
---

This glossary provides definitions for terms used in the graph algorithms documentation.

  - Depth  
    The length of the longest chain of sequentially dependent operations in the algorithm. For more information, see [Computational model](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/computational-model) .
  - Work  
    The total number of operations performed by an algorithm across all processors. For more information, see [Computational model](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/computational-model) .
  - vCPU  
    A single hardware hyper-thread on a physical CPU core. For more information, see [CPU platforms](https://cloud.google.com/compute/docs/cpu-platforms) .
  - Graph  
    A structure consisting of a set of nodes and a set of edges connecting pairs of nodes.
  - Node  
    An entity in a graph. Also known as a vertex.
  - Edge  
    A connection or relationship between two nodes in a graph. An edge may be directed or undirected. A directed edge goes from a source node to a destination node: an edge from \\(x\\) to \\(y\\) (written \\(xy\\)) has source \\(x\\) and destination \\(y\\). An undirected edge connects two nodes without a specific direction.
  - Self-loop  
    An edge that connects a node to itself.
  - Directed graph  
    A graph where each edge has a direction, going from a source node to a destination node.
  - Undirected graph  
    A graph where edges connect two nodes without a specific direction. The connection between the two nodes is bidirectional. You can convert an undirected graph into a directed graph by replacing each undirected edge \\(\\{u, v\\}\\) with two directed edges \\((u, v)\\) and \\((v, u)\\).
  - Weighted graph  
    A graph where each edge has an associated numerical value, called a weight. Edge weights can represent various quantities, such as distances, costs, or strengths of connection, depending on the application.
  - Edge weight  
    A numerical value assigned to an edge of a weighted graph, typically representing a distance, cost, or strength of connection. Edge weights must be finite (positive or negative infinity and NaN are disallowed). When not explicitly provided, edge weights default to 1. Some algorithms impose additional constraints on edge weights (for example, non-negative or positive); see each algorithm's page for details.
  - Unweighted graph  
    A graph where edges do not have associated weights. In the context of the algorithms described here, unweighted graphs are treated as weighted graphs where all edge weights are equal to 1.
  - Neighbor  
    In an undirected graph, a node \\(x\\) is a neighbor of node \\(y\\) if there is an edge between \\(x\\) and \\(y\\).
  - In-neighbor  
    In a directed graph, a node \\(x\\) is an in-neighbor of node \\(y\\) if there is an edge from \\(x\\) to \\(y\\).
  - Out-neighbor  
    In a directed graph, a node \\(y\\) is an out-neighbor of node \\(x\\) if there is an edge from \\(x\\) to \\(y\\).
  - Degree  
    In an undirected graph, the number of edges connected to a node.
  - In-degree  
    In a directed graph, the number of incoming edges to a node (equivalently, the number of in-neighbors of the node).
  - Out-degree  
    In a directed graph, the number of outgoing edges from a node (equivalently, the number of out-neighbors of the node).
  - Path  
    A sequence of nodes connected by edges. In a directed graph, the edges in a path must follow the edge direction.
  - Shortest path  
    A path between two nodes with the minimum total edge weight. In an unweighted graph, a shortest path is a path with the fewest edges. There may be multiple distinct shortest paths between the same pair of nodes.
  - Shortest-path distance  
    The total edge weight of a shortest path between two nodes. If no path exists between the two nodes, the shortest-path distance is undefined.
  - Connected component  
    A maximal set of nodes such that there is a path between every pair of nodes in the set.
  - Clique  
    A set of nodes where every pair of nodes in the set is connected by an edge.
  - Cluster  
    A group of nodes identified by a clustering algorithm. Nodes within a cluster are typically more densely connected to each other than to nodes outside the cluster. Also known as a *community* .
