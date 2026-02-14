This document explains how to generate and backfill vector embeddings in bulk for textual data ( `  STRING  ` or `  JSON  ` ) that is stored in Spanner using SQL and the [Vertex AI text embedding models](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) .

**Tip:** We recommend backfilling your embeddings at a time when you're not making a large number of updates to the text data that you're generating embeddings for. This is to minimize lock contention and maintain low latency in your database.

## Prerequisites

You must have a table in your Spanner database that contains textual data ( `  STRING  ` or `  JSON  ` ). For more information about importing data, see the [Spanner import and export overview](/spanner/docs/import-export-overview#overview) .

## Example use case

Suppose you have a table in Spanner with the following schema. This table contains millions of records.

### GoogleSQL

``` text
CREATE TABLE Products (
  product_id INT64 NOT NULL,
  name STRING(MAX),
  description STRING(MAX)
) PRIMARY KEY(product_id);
```

### PostgreSQL

``` text
CREATE TABLE Products (
  product_id INT8 NOT NULL,
  name TEXT,
  description TEXT,
  PRIMARY KEY(product_id)
);
```

Your goal is to generate vector embeddings for the `  description  ` column in this table to find similar items to recommend to customers to improve their shopping experience using [vector search](/spanner/docs/find-k-nearest-neighbors) .

## Register an embedding model

### GoogleSQL

Register a text embedding model with the [Vertex AI model endpoint](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) in your Spanner database:

``` text
CREATE MODEL MODEL_NAME
INPUT(
  content STRING(MAX)
)
OUTPUT(
  embeddings STRUCT<values ARRAY<FLOAT32>>
)
REMOTE OPTIONS(
    endpoint = '//aiplatform.googleapis.com/projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
  default_batch_size = 5
)
```

Replace the following:

  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model
  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint

### PostgreSQL

In the PostgreSQL dialect, there is no need to register the model. You pass the endpoint name directly to the `  spanner.ML_PREDICT_ROW  ` function call.

For best practices, consider the following:

  - To maintain isolation of quotas, use an endpoint in a different project to generate and backfill embeddings than the production endpoint. Reserve the production endpoint to serve production traffic.
  - Make sure that the model endpoint supports the value of `  default_batch_size  ` . You can override the `  default_batch_size  ` with the query hint `  @{remote_udf_max_rows_per_rpc=NEW_NUMBER}  ` . For information about the `  default_batch_size  ` limit for each region, see [Get text embeddings for a snippet of text](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#get_text_embeddings_for_a_snippet_of_text) .
  - Define the endpoint with a specific model version (e.g. `  @003  ` ) instead of `  @latest  ` . This is because the embedding vectors generated for the same piece of text might differ depending on the version of the model that you use; which is why you want to avoid using different model versions to generate embeddings in the same dataset. In addition, updating the model version in the model definition statement doesn't update the embeddings that are already generated with this model. One way to manage the model version for embeddings is to create an additional column in the table which stores the model version.
  - Custom tuned text embedding models aren't supported with the GoogleSQL `  ML.PREDICT  ` and PostgreSQL `  spanner.ML_PREDICT_ROW  ` functions.

### Test the end-to-end integration of the embeddings model

You can execute a query to test that the embedding model is configured successfully, and embeddings are retrieved. For example, run the following query:

### GoogleSQL

``` text
SELECT embeddings.values
FROM SAFE.ML.PREDICT(
  MODEL MODEL_NAME,
  (SELECT description AS content FROM products LIMIT 10)
);
```

Replace the following:

  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

### PostgreSQL

``` text
SELECT spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', description))))
FROM Products
LIMIT 10;
```

Replace the following:

  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint
  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

## Update the source table to include additional columns to store the embeddings

Next, update the source table schema to include an additional column of the data type `  ARRAY<FLOAT32>  ` to store the generated embeddings:

### GoogleSQL

``` text
ALTER TABLE TABLE_NAME
ADD COLUMN EMBEDDING_COLUMN_NAME ARRAY<FLOAT32>;
```

Replace the following:

  - `  TABLE_NAME  ` : the name of the source table
  - `  EMBEDDING_COLUMN_NAME  ` : the name of the column in which you want to add generated embeddings

### PostgreSQL

``` text
ALTER TABLE TABLE_NAME
ADD COLUMN EMBEDDING_COLUMN_NAME real[];
```

Replace the following:

  - `  TABLE_NAME  ` : the name of the source table
  - `  EMBEDDING_COLUMN_NAME  ` : the name of the column in which you want to add generated embeddings

For example, using the `  products  ` table example, run:

### GoogleSQL

``` text
ALTER TABLE Products
ADD COLUMN desc_embed ARRAY<FLOAT32>;
```

### PostgreSQL

``` text
ALTER TABLE Products
ADD COLUMN desc_embed real[];
```

You can add another column to manage the version of the embedding model.

### GoogleSQL

``` text
ALTER TABLE Products
ADD COLUMN desc_embed_model_version INT64;
```

### PostgreSQL

``` text
ALTER TABLE Products
ADD COLUMN desc_embed_model_version INT8;
```

## Increase the quota for Vertex AI

You might need to increase the Vertex AI API quota for the region which uses the text embedding model. To request an increase, see Vertex AI [Quota increases](/vertex-ai/docs/quotas#quota_increases) .

For more information, see [Vertex AI quotas and limits](/vertex-ai/docs/quotas) .

## Backfill embeddings

Finally, [execute the following `  UPDATE  ` statement using partitioned DML](/spanner/docs/dml-tasks#partitioned-dml) to generate embeddings for the textual data column and store the embeddings in your database. You can store the model version along with the embeddings. We recommend that you execute this query during a low-traffic window in your database.

### GoogleSQL

``` text
UPDATE TABLE_NAME
SET
  TABLE_NAME.EMBEDDING_COLUMN_NAME = (
    SELECT embeddings.values
    FROM SAFE.ML.PREDICT(
      MODEL MODEL_NAME,
      (SELECT TABLE_NAME.DATA_COLUMN_NAME AS content)
    ) @{remote_udf_max_rows_per_rpc=MAX_ROWS}
  ),
  TABLE_NAME.EMBEDDING_VERSION_COLUMN = MODEL_VERSION
WHERE FILTER_CONDITION;
```

Replace the following:

  - `  TABLE_NAME  ` : the name of the table with the textual data
  - `  EMBEDDING_COLUMN_NAME  ` : the name of the column in which you want to add generated embeddings
  - `  DATA_COLUMN_NAME  ` : the name of the column with the textual data
  - `  MODEL_NAME  ` : the name of the Vertex AI embedding model
  - `  MAX_ROWS  ` : the maximum number of rows per RPC
  - `  EMBEDDING_VERSION_COLUMN  ` : the column that manages the version of the embedding model used to backfill your embeddings
  - `  MODEL_VERSION  ` : the version of the text embedding model
  - `  FILTER_CONDITION  ` : a [partitionable](/spanner/docs/dml-partitioned#partitionable-idempotent) filter condition that you want to apply

Using `  SAFE.ML.PREDICT  ` returns `  NULL  ` for failed requests. You can also use `  SAFE.ML.PREDICT  ` in combination with a `  WHERE embedding_column IS NULL  ` filter to rerun your query without computing the embeddings for the fields that are already computed.

### PostgreSQL

``` text
UPDATE TABLE_NAME
SET
  EMBEDDING_COLUMN_NAME = spanner.FLOAT32_ARRAY(spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', DATA_COLUMN_NAME)))
  ) /*@ remote_udf_max_rows_per_rpc=MAX_ROWS */ ->'predictions'->0->'embeddings'->'values'),
  EMBEDDING_VERSION_COLUMN = MODEL_VERSION
WHERE FILTER_CONDITION;
```

Replace the following:

  - `  TABLE_NAME  ` : the name of the table with the textual data
  - `  EMBEDDING_COLUMN_NAME  ` : the name of the column in which you want to add generated embeddings
  - `  DATA_COLUMN_NAME  ` : the name of the column with the textual data
  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint
  - `  MODEL_NAME  ` : the name of the Vertex AI embedding model
  - `  MODEL_VERSION  ` : the version of the Vertex AI embedding model
  - `  MAX_ROWS  ` : the maximum number of rows per RPC
  - `  EMBEDDING_VERSION_COLUMN  ` : the column that manages the version of the text embedding model used to backfill your embeddings
  - `  FILTER_CONDITION  ` : a [partitionable](/spanner/docs/dml-partitioned#partitionable-idempotent) filter condition that you want to apply

An example backfill query for the `  products  ` table:

### GoogleSQL

``` text
UPDATE products
SET
  products.desc_embed = (
    SELECT embeddings.values
    FROM SAFE.ML.PREDICT(
      MODEL embedding_model,
      (SELECT products.description AS content)
    ) @{remote_udf_max_rows_per_rpc=200}
  ),
  products.desc_embed_model_version = 3
WHERE products.desc_embed IS NULL;
```

### PostgreSQL

``` text
UPDATE products
SET
  desc_embed = spanner.FLOAT32_ARRAY(spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', description)))
  ) /*@ remote_udf_max_rows_per_rpc=200 */ ->'predictions'->0->'embeddings'->'values'),
  desc_embed_model_version = 3
WHERE desc_embed IS NULL;
```

For best practices, consider the following:

  - The default gRPC timeout for the Spanner API is one hour. Depending on the amount of embeddings you are backfilling, you might need to increase this timeout to ensure that the `  UPDATE  ` partitioned DML has sufficient time to complete. For more information, see [Configure custom timeouts and retries](/spanner/docs/custom-timeout-and-retry) .

## Performance and other considerations

Consider the following to optimize performance when backfilling embedding data.

### Number of nodes

Partitioned DML executes the given DML statement on different partitions in parallel. For instances with a high number of nodes, you might observe quota errors during the execution of partitioned DML. If the Vertex AI API requests are throttled due to Vertex AI API quota limits, then Spanner retries these failures under the [partitioned DML transaction mode](/spanner/docs/dml-partitioned#dml-partitioned-dml) for a maximum of 20 times. If you observe a high rate of quota errors in Vertex AI, then [increase the quota for Vertex AI](#increase-quota) . You can also tune the parallelism using the statement-level hint `  @{pdml_max_parallelism=DESIRED_NUMBER}  ` while using GoogleSQL. The following example sets the parallelism to '5':

### GoogleSQL

``` text
@{pdml_max_parallelism=5} UPDATE products
SET products.desc_embed =(
  SELECT embeddings.values
  FROM SAFE.ML.PREDICT(MODEL embedding_model, (
        SELECT products.value AS CONTENT
        )
  )
      @{remote_udf_max_rows_per_rpc=200}
),
products.desc_embed_model_version = MODEL_VERSION
WHERE products.desc_embed IS NULL;
```

### Size of text in the data column

The Vertex AI embedding model has limits on the maximum number of tokens for each text input. Different model versions have different token limits. Each Vertex AI request can have multiple input text fields, but there is a limit on the maximum number of tokens which is present in a single request. For GoogleSQL databases, if you encounter an `  INVALID_ARGUMENT  ` error with a "Request is too large" message, try reducing the batch size to avoid the error. To do so, you can configure `  default_batch_size  ` or use the `  @{remote_udf_max_outstanding_rpcs}  ` query hint when registering the model.

### Number of API requests sent to Vertex AI

You can use the query hint `  @{remote_udf_max_outstanding_rpcs}  ` to increase or decrease the number of requests sent to Vertex AI from Spanner. Be aware that increasing this limit can increase the CPU and memory usage of the Spanner instance. For GoogleSQL databases, using this query hint overrides the `  default_batch_size  ` configured for your model.

### Monitor backfill progress

You can monitor the number of requests, latency, and network bytes sent to Vertex AI from Spanner using the [system insights dashboard](/spanner/docs/monitoring-console) .

## What's next

  - Learn how to [perform a similarity vector search by finding the K-nearest neighbors](/spanner/docs/find-k-nearest-neighbors) .
  - Learn more about machine learning and embeddings in our [crash course on embeddings](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture) .
  - Learn more about [Vertex AI text embedding models](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) .
