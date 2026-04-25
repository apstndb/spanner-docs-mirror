This page describes vector search and how it works in Spanner.

Vector search is a high-performance, built-in capability that enables semantic search and similarity matching on high-dimensional vector data. By storing and indexing vector embeddings directly within your transactional database, Spanner eliminates the need for separate vector databases and complex ETL pipelines.

## Key concepts

This section introduces the following key concepts of vector search:

  - Vector embeddings
  - Search methods (KNN and ANN)
  - Distance functions

### Vector embeddings

Vector embeddings are high-dimensional, numerical representations of unstructured data. They are generated from the unstructured data using machine learning models. For example, you can use the Vertex AI text embedding API to [generate, store, and update text embeddings](https://docs.cloud.google.com/spanner/docs/ml-tutorial-embeddings) for data stored in Spanner.

### Search methods

Spanner supports two methods for finding similar vectors:

  - **[K-nearest neighbors (KNN)](https://docs.cloud.google.com/spanner/docs/find-k-nearest-neighbors)** : Performs an exact search by calculating the distance between the query and every vector in the dataset. It provides more accurate recall, but is computationally expensive for massive datasets.

  - **[Approximate nearest neighbor (ANN)](https://docs.cloud.google.com/spanner/docs/find-approximate-nearest-neighbors)** : Uses a [vector index](https://docs.cloud.google.com/spanner/docs/vector-indexes) (based on Google's ScaNN algorithm) to find matches quickly across a large number of vectors. It trades a small amount of accuracy (recall) for gains in speed and scalability.

## Key capabilities

  - **Scalability** : Supports billions of vectors for unpartitioned ANN search, or trillions of vectors for partitioned KNN workloads.

  - **Unified AI database and engine** : Use the [GoogleSQL `ML.PREDICT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/ml-functions#mlpredict) or [PostgreSQL `spanner.ML_PREDICT_ROW`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#ml) function to generate embeddings from Vertex AI models directly within your query flow.

  - **Inline filtering** : Efficiently combine vector search with structured metadata filters (for example, "Find similar images where category = 'shoes' and price \< 100") without losing performance.

  - **LangChain integration** : Built-in support for [LangChain](https://docs.cloud.google.com/spanner/docs/langchain) lets you build [retrieval-augmented generation (RAG)](https://cloud.google.com/use-cases/retrieval-augmented-generation) applications using Spanner as the vector store.

### Hybrid search capabilities

Vector search is most powerful when combined with Spanner's other data features:

| Combination                                                                                                   | Benefit                                                                                                                        |
| ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Vector search with SQL filtering                                                                              | Efficiently combine vector search with filters (for example, "Find similar images where category = 'shoes' and price \< 100"). |
| [Vector search + full-text search](https://docs.cloud.google.com/spanner/docs/hybrid-full-text-vector-search) | Combine semantic similarity with keyword precision using reciprocal rank fusion (RRF) for superior search relevance.           |
| [Vector + graph](https://docs.cloud.google.com/spanner/docs/graph/perform-vector-similarity-search)           | Use vector search to find relevant entry points (nodes) in a property graph and then traverse complex relationships.           |

## What's next

  - Learn more about [Spanner AI](https://docs.cloud.google.com/spanner/docs/spanner-ai-overview) .
  - Learn more about how to [get Vertex AI text embeddings](https://docs.cloud.google.com/spanner/docs/ml-tutorial-embeddings) .
  - Learn more about how to [perform K-nearest neighbors (KNN) search](https://docs.cloud.google.com/spanner/docs/find-k-nearest-neighbors) .
  - Learn more about how to [perform approximate nearest neighbors (ANN) search](https://docs.cloud.google.com/spanner/docs/find-approximate-nearest-neighbors) .
