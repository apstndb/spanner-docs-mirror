---
name: documents/docs.cloud.google.com/spanner-omni/vector-search-overview
uri: https://docs.cloud.google.com/spanner-omni/vector-search-overview
title: Spanner Omni vector search overview
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Vector search in Spanner Omni is a high-performance, built-in feature that enables semantic search and similarity matching on high-dimensional vector data. By storing and indexing vector embeddings directly within your transactional database, Spanner Omni eliminates separate vector databases and complex extract, transform, load (ETL) pipelines.

The topics in this document apply to Spanner Omni in the same way they apply to Spanner.

## Vector search overview

Vector search lets you find semantically similar items by representing data as numerical vectors (embeddings). Spanner Omni supports two primary search methods:

  - **K-nearest neighbors (KNN)** : Performs an exact search by calculating the distance between the query and every vector in the dataset. It provides the highest recall but can be computationally expensive for large datasets.

  - **Approximate nearest neighbors (ANN)** : Uses a vector index to find matches fast across large datasets. It trades a small amount of accuracy (recall) for gains in speed and scalability.

Vector search is especially powerful when combined with other features:

| Combination                      | Benefit                                                                                                                        |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Vector search with SQL filtering | Efficiently combine vector search with filters (for example, "Find similar images where category = 'shoes' and price \< 100"). |
| Vector search + full-text search | Combine semantic similarity with keyword precision using reciprocal rank fusion (RRF) for improved search relevance.           |
| Vector + graph                   | Use vector search to find relevant entry points (nodes) in a property graph and then traverse complex relationships.           |

For more information, see the [Spanner vector search overview](https://docs.cloud.google.com/spanner/docs/vector-search-overview) in the Spanner documentation.

## Perform K-nearest neighbors search

Spanner Omni supports K-nearest neighbors (KNN) search using built-in distance functions. You can provide a vector embedding as an input parameter to find the nearest vectors in N-dimensional space.

The following distance functions are available:

  - `COSINE_DISTANCE()` : Measures the cosine of the angle between two vectors

  - `EUCLIDEAN_DISTANCE()` : Measures the shortest straight-line distance between two vectors

  - `DOT_PRODUCT()` : Calculates the cosine of the angle multiplied by the product of vector magnitudes (ideal for normalized data)

For more information, see [Perform vector similarity search by finding the K-nearest neighbors](https://docs.cloud.google.com/spanner/docs/find-k-nearest-neighbors) in the Spanner documentation.

## Choose the best vector distance function

Selecting the appropriate distance function depends on your data and the model used to generate embeddings.

| Function           | Description                                                                                  | Relationship to increasing similarity |
| ------------------ | -------------------------------------------------------------------------------------------- | ------------------------------------- |
| Dot product        | Calculates the cosine of angle multiplied by the product of corresponding vector magnitudes. | Increases                             |
| Cosine distance    | Measures the cosine of the angle between two vectors (1 - cosine similarity).                | Decreases                             |
| Euclidean distance | Measures the straight line distance between two vectors.                                     | Decreases                             |

If your embeddings are normalized (magnitude = 1.0), `DOT_PRODUCT()` is typically an efficient choice. For non-normalized data, experiment with `COSINE_DISTANCE()` or `EUCLIDEAN_DISTANCE()` to determine which produces better results for your use case.

For more information, see [Choose among vector distance functions](https://docs.cloud.google.com/spanner/docs/choose-vector-distance-function) in the Spanner documentation.

## Approximate nearest neighbors (ANN)

ANN search is designed for very large datasets where exact KNN search becomes too slow or expensive. It uses a vector index to provide fast results with a small tradeoff in recall.

Approximate nearest neighbor (ANN) search in Spanner Omni supports datasets of up to 1 million vectors for vectors up to 128 dimensions in length. If your vectors have more dimensions, then the supported number of vectors decreases proportionately.

### Perform ANN search with vector indexes

To perform an ANN search, you use approximate distance functions such as `APPROX_COSINE_DISTANCE()` , `APPROX_EUCLIDEAN_DISTANCE()` , or `APPROX_DOT_PRODUCT()` . These functions require:

  - An existing vector index on the embedding column.

  - An `ORDER BY` clause using the approximate distance function.

  - A `LIMIT` clause to specify the number of results.

For more information, see [Find approximate nearest neighbors (ANN) and query vector embeddings](https://docs.cloud.google.com/spanner/docs/find-approximate-nearest-neighbors) in the Spanner documentation.

### Create and manage vector indexes

When creating a vector index, you must specify the `vector_length` of your embedding column and can use the `STORING` clause to include additional columns for faster filtering.

The following is an example of how to create a vector index:

    CREATE VECTOR INDEX INDEX_NAME
      ON TABLE_NAME(EMBEDDING_COLUMN)
      OPTIONS (distance_type = 'DISTANCE_TYPE', tree_depth = 2, num_leaves = 1000);

For more information, see [Create and manage vector indexes](https://docs.cloud.google.com/spanner/docs/vector-indexes) in the Spanner documentation.

### Vector indexing best practices

To maintain high search performance and recall:

  - **Tune index options** : Adjust `num_leaves` and `num_leaves_to_search` based on your data size and performance requirements.

  - **Rebuild periodically** : Rebuild your index if the distribution of your vectors changes significantly over time.

  - **Use filtering effectively** : Store frequently filtered columns in the index to improve search efficiency.

For more information, see [Vector indexing best practices](https://docs.cloud.google.com/spanner/docs/vector-index-best-practices) in the Spanner documentation.
