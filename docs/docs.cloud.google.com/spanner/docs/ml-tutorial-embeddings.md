---
name: documents/docs.cloud.google.com/spanner/docs/ml-tutorial-embeddings
uri: https://docs.cloud.google.com/spanner/docs/ml-tutorial-embeddings
title: Get Agent Platform text embeddings
description: Generate and store vector embeddings in Spanner using Gemini Enterprise Agent Platform models, and perform similarity searches.
data_source: docs.cloud.google.com
---

This page describes how to use the [Gemini Enterprise Agent Platform text embedding](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/get-text-embeddings#supported-models) API to generate, store, and update text embeddings for data stored in Spanner for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

A text embedding is a vector representation of text data, and they are used in many ways to find similar items. You interact with them every time you complete a Google Search or see recommendations when shopping online. When you create text embeddings, you get vector representations of natural text as arrays of floating point numbers. This means that all of your input text is assigned a numerical representation. By comparing the numerical distance between the vector representations of two pieces of text, an application can determine the similarity between the text or the objects represented by the text.

With the Agent Platform text embeddings API, you can create a text embedding with [Generative AI](https://docs.cloud.google.com/gemini-enterprise-agent-platform/overview) . In this tutorial, you use the Agent Platform text embedding model to generate text embeddings for the data stored in Spanner.

To learn which models and endpoint locations work with Spanner, see [Supported models and endpoint locations](https://docs.cloud.google.com/spanner/docs/ml-tutorial-embeddings#choose-model-location) . For more information about text embeddings, see [Get text embeddings](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/get-text-embeddings) .

## Objective

In this tutorial, you learn how to:

- Register a [Agent Platform text embedding model](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/get-text-embeddings#supported-models) in a Spanner schema using DDL statements.
- Reference the registered model using SQL queries to generate embeddings from data stored in Spanner.

## Pricing

This tutorial uses billable components of Google Cloud, including:

- Spanner
- Agent Platform

For more information about Spanner costs, see the [Spanner pricing](https://docs.cloud.google.com/products/gemini-enterprise-agent-platform/pricing) page.

For more information about Agent Platform costs, see the [Agent Platform pricing](https://cloud.google.com/products/gemini-enterprise-agent-platform/pricing) page.

## Supported models and endpoint locations

In your endpoint path, specify a text embedding model that uses the Agent Platform `predict` API and a location where Agent Platform serves that model. Spanner doesn't host embedding models or keep its own list of supported models. Instead, the `ML.PREDICT` function in GoogleSQL and the `spanner.ML_PREDICT_ROW` function in PostgreSQL send each embedding request to the `predict` API at the endpoint path that you specify.

In the endpoint path, Spanner accepts regional locations such as `us-central1` or `europe-west1` , the `us` and `eu` multi-region locations, and the `global` location. The model, not Spanner, determines which of these locations work. Your Spanner instance configuration doesn't restrict the endpoint location. To reduce latency, choose an endpoint in the same region as your Spanner instance, or on the same continent for multi-region instances.

If the endpoint is in a different region than the Spanner server that runs the query, you pay for [data transfer between regions](https://docs.cloud.google.com/spanner/docs/ml#pricing) .

The following table lists the Agent Platform publisher model IDs that use the `predict` API. Use the publisher model ID in the `publishers/google/models/` segment of the endpoint path.

| Publisher model ID                | Output dimensions |
|-----------------------------------|-------------------|
| `gemini-embedding-001`            | Up to 3,072       |
| `text-embedding-005`              | Up to 768         |
| `text-multilingual-embedding-002` | Up to 768         |

All of these models are served at regional locations only, such as `us-central1` or `europe-west1` . None of them support the `us` , `eu` , or `global` locations. For the full list of regions, see [Google model endpoint locations](https://docs.cloud.google.com/gemini-enterprise-agent-platform/resources/locations#google-models) and select a regional tab such as **United States** or **Europe** . On that page, `gemini-embedding-001` appears as **Gemini Embedding** , and `text-embedding-005` and `text-multilingual-embedding-002` appear under **Embeddings for Text** .

In GoogleSQL, the name that you give the model in the `CREATE MODEL` statement is a Spanner schema identifier that is separate from the publisher model ID.

Before you choose a model, check its availability and retirement date in [Model versions and lifecycle](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/model-versions) .

The `ML.PREDICT` and `spanner.ML_PREDICT_ROW` functions require a model that uses the `predict` API. The `gemini-embedding-2` model uses the `embedContent` API instead, so you can't use it with these functions. This model is also served only at the `global` , `us` , and `eu` locations. Use `gemini-embedding-001` instead.

In GoogleSQL, the `CREATE MODEL` statement doesn't verify publisher `predict` endpoints when you register the model. In PostgreSQL, the `spanner.ML_PREDICT_ROW` function doesn't register a model. In both dialects, a model ID, version, or location that Agent Platform doesn't serve fails at query time with a `NOT_FOUND` error. In GoogleSQL, an `OUTPUT` clause that doesn't match the response fails with an `INVALID_ARGUMENT` error. After you register or choose a model, run a test query to confirm that the model and location work. For more information, see [Troubleshoot Spanner Agent Platform integration](https://docs.cloud.google.com/spanner/docs/ml-troubleshoot) .

## Generate and store text embeddings

Depending on the model you use, generating embeddings might take some time. For more performance sensitive workloads, the best practice is to avoid generating embeddings in read-write transactions. Instead, generate the embeddings in a [read-only transaction](https://docs.cloud.google.com/spanner/docs/samples/spanner-read-only-transaction) using the following SQL examples.

### GoogleSQL

**Register a text embeddings model in Spanner**

In GoogleSQL, you must register a model before using it with the `ML.PREDICT` function. To register the Agent Platform text embedding model in a Spanner database, [execute](https://docs.cloud.google.com/spanner/docs/schema-updates) the following DDL [statement](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create_model) :

```
CREATE MODEL MODEL_NAME
INPUT(
  content STRING(MAX),
  -- Optional: For models that support specifying task type.
  task_type STRING(MAX),
)
OUTPUT(
  embeddings
    STRUCT<
      statistics STRUCT<truncated BOOL, token_count FLOAT64>,
      values ARRAY<FLOAT64>>
)
REMOTE OPTIONS (
  endpoint = '//aiplatform.googleapis.com/projects/PROJECT/locations/LOCATION/publishers/google/models/PUBLISHER_MODEL_ID'
);
```

Replace the following:

- `MODEL_NAME` : the name of the model in your Spanner schema, such as `EmbeddingsModel`
- `PROJECT` : the project hosting the Agent Platform endpoint
- `LOCATION` : the location of the Agent Platform endpoint
- `PUBLISHER_MODEL_ID` : the publisher model ID from [Supported models and endpoint locations](https://docs.cloud.google.com/spanner/docs/ml-tutorial-embeddings#choose-model-location) , such as `gemini-embedding-001`

Spanner grants appropriate permissions automatically. If it doesn't, review the [model endpoint access control](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#model_endpoint_access_control) .

Schema discovery and validation is not available for Generative AI models. You must provide `INPUT` and `OUTPUT` clauses that match the model's schema. The text embeddings API returns each prediction as an object with an `embeddings` field, which contains a `statistics` object and a `values` array. The preceding `OUTPUT` clause therefore declares the `embeddings` column as a `STRUCT` (with `values` and optional `statistics` ), not as an `ARRAY<FLOAT64>` type. The `values` field holds the embedding vector, so the queries in this tutorial select `embeddings.values` . If the `OUTPUT` clause doesn't match the response, the `ML.PREDICT` call fails at query time with an `INVALID_ARGUMENT` error. For the response format, see [Text embeddings API](https://docs.cloud.google.com/gemini-enterprise-agent-platform/reference/models/text-embeddings-api#response_body) .

**Generate text embeddings**

To generate embeddings, pass a piece of text directly to the [`ML.PREDICT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function using the following SQL:

```
SELECT embeddings.values
FROM ML.PREDICT(
  MODEL MODEL_NAME,
  (SELECT "A product description" as content)
);
```

To generate embeddings for data stored in a table, use the following SQL:

```
SELECT id, embeddings.values
FROM ML.PREDICT(
  MODEL MODEL_NAME,
  (SELECT id, description as content FROM Products)
);
```

To specify [task type](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/task-types) and [output dimensions](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/get-text-embeddings#choose_an_embedding_dimension) :

```
UPDATE Products p
SET description_embedding = (
  SELECT embeddings.values
  FROM ML.PREDICT(
    MODEL MODEL_NAME,
    (SELECT p.description as content, "RETRIEVAL_DOCUMENT" as task_type),
    STRUCT(768 AS outputDimensionality)
  ));

SELECT p.product_id, p.name, p.description, COSINE_DISTANCE(
    p.description_embedding,
    (
      SELECT embeddings.values 
      FROM ML.PREDICT(
        MODEL MODEL_NAME,
        (SELECT @user_query as content, "RETRIEVAL_QUERY" as task_type),
        STRUCT(768 AS outputDimensionality)
      ) 
    )
  ) AS distance
FROM Products p
ORDER BY distance
LIMIT 5;
```

**Store text embeddings**

After generating the embeddings in a read-only transaction, store them in Spanner so they can be managed with your operational data. To store the embeddings, use a [read-write transaction](https://docs.cloud.google.com/spanner/docs/samples/spanner-read-write-transaction) .

For workloads that are less performance sensitive, you can generate and insert embeddings with the following SQL in a read-write transaction. Specifying `vector_length` is optional for storing embeddings, [recommended for KNN search](https://docs.cloud.google.com/spanner/docs/vector-search-best-practices#annotate-embedding-column) , and required for [vector indexes](https://docs.cloud.google.com/spanner/docs/vector-indexes) . Replace `VECTOR_LENGTH` with your model's output dimension, such as `3072` or `768` , or omit `vector_length` if the column stores vectors of varying dimensions:

```
CREATE TABLE Products(
  id INT64 NOT NULL,
  description STRING(MAX),
  embeddings ARRAY<FLOAT32>(vector_length=>VECTOR_LENGTH),
) PRIMARY KEY(id);
```

```
INSERT INTO Products (id, description, embeddings)
SELECT @Id, @Description, embeddings.values
FROM ML.PREDICT(
  MODEL MODEL_NAME,
  (SELECT @Description as content)
);
```

### PostgreSQL

**Generate text embeddings**

To generate embeddings, pass a piece of text directly to the [`spanner.ML_PREDICT_ROW`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#ml) function using the following SQL:

```
SELECT
  spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    '{"instances": [{"content": "A product description"}]}'::jsonb
  ) ->'predictions'->0->'embeddings'->'values';
```

Replace the following:

- `PROJECT` : the project hosting the Agent Platform endpoint
- `LOCATION` : the location of the Agent Platform endpoint
- `MODEL_NAME` : the name of the Agent Platform text embedding model

The `spanner.ML_PREDICT_ROW` function returns a `JSONB` object with a `predictions` array. Each element in `predictions` has an `embeddings` object that contains a `statistics` object and a `values` array. In the examples in this tutorial, the `->` operators extract the embedding vector from the `predictions[0].embeddings.values` path. For the response format, see [Text embeddings API](https://docs.cloud.google.com/gemini-enterprise-agent-platform/reference/models/text-embeddings-api#response_body) .

To generate embeddings for data stored in a table, use the following SQL:

```
SELECT id, spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', description))))
  ) -> 'predictions'->0->'embeddings'->'values'
FROM Products;
```

Replace the following:

- `PROJECT` : the project hosting the Agent Platform endpoint
- `LOCATION` : the location of the Agent Platform endpoint
- `MODEL_NAME` : the name of the Agent Platform text embedding model

To specify [task type](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/task-types) and [output dimensions](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/get-text-embeddings#choose_an_embedding_dimension) :

```
UPDATE Products p
SET description_embedding = spanner.float64_array(
  spanner.ML_PREDICT_ROW(
   'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT(
      'instances', JSONB_BUILD_ARRAY(
        JSONB_BUILD_OBJECT(
          'content', p.description,
          'task_type', 'RETRIEVAL_DOCUMENT'
        )
      ),
      'parameters', JSONB_BUILD_OBJECT('outputDimensionality', 768)
    )
  )->'predictions'->0->'embeddings'->'values'
);

SELECT p.product_id, p.name, p.description, spanner.COSINE_DISTANCE(
    p.description_embedding,
    spanner.float64_array(
      spanner.ML_PREDICT_ROW(
        'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
        JSONB_BUILD_OBJECT(
          'instances', JSONB_BUILD_ARRAY(
            JSONB_BUILD_OBJECT(
              'content', $1,
              'task_type', 'RETRIEVAL_QUERY'
            )
          ),
          'parameters', JSONB_BUILD_OBJECT('outputDimensionality', 768)
        )
      )->'predictions'->0->'embeddings'->'values'
    )
  ) AS distance
FROM Products p
ORDER BY distance
LIMIT 5;
```

Replace the following:

- `PROJECT` : the project hosting the Agent Platform endpoint
- `LOCATION` : the location of the Agent Platform endpoint
- `MODEL_NAME` : the name of the Agent Platform text embedding model

**Store text embeddings**

After generating the embeddings in a read-only transaction, store them in Spanner so they can be managed with your operational data. To store the embeddings, use a [read-write transaction](https://docs.cloud.google.com/spanner/docs/samples/spanner-read-write-transaction) .

For workloads that are less performance sensitive, you can generate and insert embeddings with the following SQL in a read-write transaction. Specifying `VECTOR LENGTH` is optional for storing embeddings, [recommended for KNN search](https://docs.cloud.google.com/spanner/docs/vector-search-best-practices#annotate-embedding-column) , and required for [vector indexes](https://docs.cloud.google.com/spanner/docs/vector-indexes) . Replace `VECTOR_LENGTH` with your model's output dimension, such as `3072` or `768` , or omit `VECTOR LENGTH` if the column stores vectors of varying dimensions:

```
CREATE TABLE Products (
  id INT8 NOT NULL,
  description TEXT,
  embeddings REAL[] VECTOR LENGTH VECTOR_LENGTH,
  PRIMARY KEY(id)
);
```

```
INSERT INTO Products (id, description, embeddings)
SELECT @Id, @Description, spanner.FLOAT32_ARRAY(spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', @Description)))
  ) -> 'predictions'->0->'embeddings'->'values'
));
```

Replace the following:

- `PROJECT` : the project hosting the Agent Platform endpoint
- `LOCATION` : the location of the Agent Platform endpoint
- `MODEL_NAME` : the name of the Agent Platform text embedding model

## Update text embeddings

To update your embeddings or to ingest data in realtime, use the `UPDATE` ( [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/dml-syntax#update-statement) and [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/dml-syntax#update-statement) ) statement.

To update the `Products` table in the previous example, use the following SQL:

### GoogleSQL

```
UPDATE Products
SET
  description = @description,
  embeddings = (SELECT embeddings.values
                  FROM ML.PREDICT(MODEL MODEL_NAME, (SELECT @description as content))
              )
WHERE id = @id;
```

Replace the following:

- ` `*`MODEL_NAME`*` ` : the name of the Agent Platform text embedding model

### PostgreSQL

```
UPDATE
  Products
SET
  description = $1,
  embeddings = spanner.FLOAT32_ARRAY(
    spanner.ML_PREDICT_ROW(
      'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
      JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', $1)))
    ) -> 'predictions'->0->'embeddings'->'values')
WHERE
  id = $2;
```

Replace the following:

- `PROJECT` : the project hosting the Agent Platform endpoint
- `LOCATION` : the location of the Agent Platform endpoint
- `MODEL_NAME` : the name of the Agent Platform text embedding model

## What's next

- Learn [how to use Agent Platform Vector Search](https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/vector-search/overview) to search for semantically similar items.
- Learn more about machine learning and embeddings in our [crash course on embeddings](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture) .
- Learn more about [Agent Platform text embedding models](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/embeddings/get-text-embeddings#supported-models) .
