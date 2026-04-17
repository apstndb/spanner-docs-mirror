This page describes tips that can help you if you encounter issues using Spanner vector search.

## Query fails with a 'no vector index found' error

This issue occurs when a query that you expect to use a [vector index](https://docs.cloud.google.com/spanner/docs/find-approximate-nearest-neighbors) runs, but the query optimizer can't find a suitable index.

To resolve this issue, check for the following common causes:

  - **Distance type mismatch** : Verify that the distance type defined on the index matches the distance type used in the query.
  - **Index backfilling** : Confirm that the index backfilling process is complete. Vector indexes aren't available for queries until backfilling finishes. For more information, see [manage and observe long-running operations](https://docs.cloud.google.com/spanner/docs/manage-and-observe-long-running-operations) .
  - **Missing `IS NOT NULL` filter** : Ensure your query includes an `IS NOT NULL` filter on the embedding column. This filter must match the filter in the vector index definition for the query optimizer to consider the index.

## A query fails with an 'unsupported use of an approximate distance function' error

Not all query patterns support ANN search. Review the approximate distance function documentation for detailed usage information and limitations:

  - [`APPROX_COSINE_DISTANCE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_cosine_distance)
  - [`APPROX_DOT_PRODUCT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product)
  - [`APPROX_EUCLIDEAN_DISTANCE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_euclidean_distance)

## Verify that a query uses a vector index

You can verify that a query uses a vector index by checking the [query execution plan](https://docs.cloud.google.com/spanner/docs/query-execution-plans) .

In the query execution plan, look for `Scan` nodes that reference your vector index.

## What's next

  - [Perform K-nearest neighbors (KNN) search](https://docs.cloud.google.com/spanner/docs/find-k-nearest-neighbors)
  - [Perform approximate nearest neighbors (ANN) search with vector indexes](https://docs.cloud.google.com/spanner/docs/find-approximate-nearest-neighbors)
  - [Vector indexing best practices](https://docs.cloud.google.com/spanner/docs/vector-index-best-practices)
  - [Vector search best practices](https://docs.cloud.google.com/spanner/docs/vector-search-best-practices)
