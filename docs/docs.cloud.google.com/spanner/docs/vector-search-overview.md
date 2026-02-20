This page describes vector search and how it works in Spanner.

Vector search is a high-performance, built-in capability that enables semantic search and similarity matching on high-dimensional vector data. By storing and indexing vector embeddings directly within your transactional database, Spanner eliminates the need for separate vector databases and complex ETL pipelines.

## Key concepts

This section introduces the following key concepts of vector search:

  - Vector embeddings
  - Search methods (KNN and ANN)
  - Distance functions

### Vector embeddings

Vector embeddings are high-dimensional, numerical representations of unstructured data. They are generated from the unstructured data using machine learning models. For example, you can use the Vertex AI text embedding API to [generate, store, and update text embeddings](/spanner/docs/ml-tutorial-embeddings) for data stored in Spanner.

### Search methods

Spanner supports two methods for finding similar vectors:

  - **[K-nearest neighbors (KNN)](/spanner/docs/find-k-nearest-neighbors)** : Performs an exact search by calculating the distance between the query and every vector in the dataset. It provides more accurate recall, but is computationally expensive for massive datasets.

  - **[Approximate nearest neighbor (ANN)](/spanner/docs/find-approximate-nearest-neighbors)** : Uses a [vector index](/spanner/docs/vector-indexes) (based on Google's ScaNN algorithm) to find matches quickly across a large number of vectors. It trades a small amount of accuracy (recall) for gains in speed and scalability.

## Key capabilities

  - **Scalability** : Supports billions of vectors for unpartitioned ANN search, or trillions of vectors for partitioned KNN workloads.

  - **Unified AI database and engine** : Use the [GoogleSQL `  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) or [PostgreSQL `  spanner.ML_PREDICT_ROW  `](/spanner/docs/reference/postgresql/functions#ml) function to generate embeddings from Vertex AI models directly within your query flow.

  - **Inline filtering** : Efficiently combine vector search with structured metadata filters (for example, "Find similar images where category = 'shoes' and price \< 100") without losing performance.

  - **LangChain integration** : Built-in support for [LangChain](/spanner/docs/langchain) lets you build [retrieval-augmented generation (RAG)](https://cloud.google.com/use-cases/retrieval-augmented-generation) applications using Spanner as the vector store.

### Hybrid search capabilities

Vector search is most powerful when combined with Spanner's other data features:

<table>
<thead>
<tr class="header">
<th>Combination</th>
<th>Benefit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Vector search with SQL filtering</td>
<td>Efficiently combine vector search with filters (for example, "Find similar images where category = 'shoes' and price &lt; 100").</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/hybrid-full-text-vector-search">Vector search + full-text search</a></td>
<td>Combine semantic similarity with keyword precision using reciprocal rank fusion (RRF) for superior search relevance.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/graph/perform-vector-similarity-search">Vector + graph</a></td>
<td>Use vector search to find relevant entry points (nodes) in a property graph and then traverse complex relationships.</td>
</tr>
</tbody>
</table>

## What's next

  - Learn more about [Spanner AI](/spanner/docs/spanner-ai-overview) .
  - Learn more about how to [get Vertex AI text embeddings](/spanner/docs/ml-tutorial-embeddings) .
  - Learn more about how to [perform K-nearest neighbors (KNN) search](/spanner/docs/find-k-nearest-neighbors) .
  - Learn more about how to [perform approximate nearest neighbors (ANN) search](/spanner/docs/find-approximate-nearest-neighbors) .
