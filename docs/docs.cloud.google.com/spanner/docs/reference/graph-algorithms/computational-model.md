---
name: documents/docs.cloud.google.com/spanner/docs/reference/graph-algorithms/computational-model
uri: https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/computational-model
title: Computational model
description: The work-depth model of parallel computation used to characterize the efficiency of graph algorithms. Defines work, depth, and their practical implications.
data_source: docs.cloud.google.com
---

With the exception of [Clique aggregator clustering](https://docs.cloud.google.com/spanner/docs/reference/graph-algorithms/clustering/clique-aggregator) , the algorithms in this library leverage parallelism to achieve high performance. Additionally, this documentation uses the work-depth model of parallel computation to characterize their efficiency. For a detailed definition of the model, see [Optimal (Randomized) Parallel Algorithms in the Binary-Forking Model](https://arxiv.org/abs/1903.04650) (Symposium on Parallelism in Algorithms and Architectures, 2020). In this model, algorithms express parallelism by *forking* to spawn tasks that can run in parallel, and *joining* to synchronize and combine their results. A fork-join computation can be viewed as a directed acyclic graph (DAG) of the dependencies that the fork and join operations induce.

Two quantities measure the computational complexity of each algorithm:

  - **Work** : The total number of operations that the algorithm performs. This corresponds to the running time of the algorithm if it runs sequentially.
  - **Depth** : The length of the longest chain of dependent operations that the algorithm must execute sequentially. This quantity, also known as *span* , determines the inherent parallelizability of the algorithm.

When executed with a parallelism level of \\(P\\), the running time of an algorithm with work \\(W\\) and depth \\(D\\) is approximately \\(O(W/P + D)\\). This implies that:

  - When sufficient parallelism is available (large \\(P\\)), the depth \\(D\\) dominates the running time. Algorithms with low depth can take full advantage of high parallelism.
  - When parallelism is limited (small \\(P\\)), the work \\(W\\) dominates. In this regime, work-efficient algorithms (those with work close to the best sequential algorithm) perform well.
