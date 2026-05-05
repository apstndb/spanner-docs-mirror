> **PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

[Video](https://www.youtube.com/watch?v=qQtE23YKZZk)

This page describes how to find approximate nearest neighbors (ANN) and query vector embeddings using the ANN distance functions.

When a dataset is small, you can use [K-nearest neighbors (KNN)](https://docs.cloud.google.com/spanner/docs/find-k-nearest-neighbors) to find the exact k-nearest vectors. However, as your dataset grows, the latency and cost of a KNN search also increase. You can use ANN to find the approximate k-nearest neighbors with significantly reduced latency and cost.

In an ANN search, the k-returned vectors aren't the true top k-nearest neighbors because the ANN search calculates approximate distances and might not look at all the vectors in the dataset. Occasionally, a few vectors that aren't among the top k-nearest neighbors are returned. This is known as *recall loss* . How much recall loss is acceptable to you depends on the use case, but in most cases, losing a bit of recall in return for improved database performance is an acceptable tradeoff.

For more details about the approximate distance functions supported in Spanner, see the following reference pages for your database dialect:

  - **GoogleSQL**
      - [`APPROX_COSINE_DISTANCE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_cosine_distance)
      - [`APPROX_EUCLIDEAN_DISTANCE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_euclidean_distance)
      - [`APPROX_DOT_PRODUCT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product)
  - **PostgreSQL**
      - [`spanner.approx_cosine_distance()`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#mathematical)
      - [`spanner.approx_euclidean_distance()`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#mathematical)
      - [`spanner.approx_dot_product()`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#mathematical)

## Query vector embeddings

Spanner accelerates approximate nearest neighbor (ANN) vector searches by using a [vector index](https://docs.cloud.google.com/spanner/docs/vector-indexes) . You can use a vector index to query vector embeddings. To query vector embeddings, you must first [create a vector index](https://docs.cloud.google.com/spanner/docs/vector-indexes#create-vector-index) . You can then use any one of the three approximate distance functions to find the ANN.

Restrictions when using the approximate distance functions include the following:

  - The approximate distance function must calculate the distance between an embedding column and a constant expression (for example, a parameter or a literal).
  - The approximate distance function output must be used in a `ORDER BY` clause as the sole sort key, and a `LIMIT` must be specified after the `ORDER BY` .
  - The query must explicitly filter out rows that aren't indexed. In most cases, this means that the query must include a `WHERE <column_name> IS NOT NULL` clause that matches the vector index definition, unless the column is already marked as `NOT NULL` in the table definition.

For a detailed list of limitations, see the [approximate distance function reference page](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions) .

**Examples**

Consider a `Documents` table that has a `DocEmbedding` column of precomputed text embeddings from the `DocContents` bytes column, and a `NullableDocEmbedding` column populated from other sources that might be null.

### GoogleSQL

    CREATE TABLE Documents (
      UserId       INT64 NOT NULL,
      DocId        INT64 NOT NULL,
      Author       STRING(1024),
      DocContents  BYTES(MAX),
      DocEmbedding ARRAY<FLOAT32> NOT NULL,
      NullableDocEmbedding ARRAY<FLOAT32>,
      WordCount    INT64
    ) PRIMARY KEY (UserId, DocId);

### PostgreSQL

    CREATE TABLE documents (
      user_id      bigint not null,
      doc_id       bigint not null,
      author       varchar(1024),
      doc_contents bytea,
      doc_embedding float4[] not null,
      nullable_doc_embedding float4[],
      word_count   bigint,
      PRIMARY KEY (user_id, doc_id)
    );

To search for the nearest 100 vectors to `[1.0, 2.0, 3.0]` :

### GoogleSQL

    SELECT DocId
    FROM Documents
    WHERE WordCount > 1000
    ORDER BY APPROX_EUCLIDEAN_DISTANCE(
      ARRAY<FLOAT32>[1.0, 2.0, 3.0], DocEmbedding,
      options => JSON '{"num_leaves_to_search": 10}')
    LIMIT 100

### PostgreSQL

    SELECT doc_id
    FROM documents
    WHERE word_count > 1000
    ORDER BY spanner.approx_euclidean_distance(
      ARRAY[1.0, 2.0, 3.0]::float4[], doc_embedding,
      options=>jsonb'{"num_leaves_to_search": 10}'
    )
    LIMIT 100;

If the embedding column is nullable:

### GoogleSQL

    SELECT DocId
    FROM Documents
    WHERE NullableDocEmbedding IS NOT NULL AND WordCount > 1000
    ORDER BY APPROX_EUCLIDEAN_DISTANCE(
      ARRAY<FLOAT32>[1.0, 2.0, 3.0], NullableDocEmbedding,
      options => JSON '{"num_leaves_to_search": 10}')
    LIMIT 100

### PostgreSQL

    SELECT doc_id
    FROM documents
    WHERE nullable_doc_embedding IS NOT NULL AND word_count > 1000
    ORDER BY spanner.approx_euclidean_distance(
      ARRAY[1.0, 2.0, 3.0]::float4[], nullable_doc_embedding,
      options=>jsonb'{"num_leaves_to_search": 10}'
    )
    LIMIT 100;

## What's next

  - Learn more about Spanner [vector indexes](https://docs.cloud.google.com/spanner/docs/vector-indexes) .

  - Learn more about the approximate distance functions in [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions) and [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#mathematical) .

  - Learn more about index statements for [GoogleSQL `VECTOR INDEX`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#vector_index_statements) and [PostgreSQL `INDEX`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/data-definition-language#index-statements) .

  - Learn more about [vector index best practices](https://docs.cloud.google.com/spanner/docs/vector-index-best-practices) .

  - Try the [Getting started with Spanner Vector Search](https://codelabs.developers.google.com/codelabs/spanner-getting-started-vector-search) for a step-by-step example of using ANN.
