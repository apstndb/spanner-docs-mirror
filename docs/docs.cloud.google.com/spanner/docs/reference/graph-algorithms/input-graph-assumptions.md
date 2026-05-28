---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/input-graph-assumptions
title: General assumptions on input graphs
description: General assumptions that the graph algorithms make on input graphs, including size limits, node identifiers, edge types, and edge weight constraints.
data_source: docs.cloud.google.com
---

The following are the general assumptions that the algorithms make on the input graphs. Individual algorithm pages document additional requirements.

  - Input graphs can have at most \\(2^{31} - 1\\) nodes.
  - Nodes are identified by consecutive integer node IDs \\(0, 1, ..., n-1\\).
  - Input graphs may be directed or undirected. In a directed graph, each edge has a source node and a destination node. In an undirected graph, edges have no direction.
  - In a directed graph, for any two nodes there is at most one edge from one to the other (but there may be edges in both directions). In an undirected graph, for any two nodes there is at most one edge between them.
  - Input graphs may have self-loops (edges connecting a node to itself).
  - Each edge has an associated weight, which must be a finite floating-point number (\\(\\pm\\infty\\) and NaN are not allowed). If the weight of an edge is not explicitly provided, it defaults to 1.
