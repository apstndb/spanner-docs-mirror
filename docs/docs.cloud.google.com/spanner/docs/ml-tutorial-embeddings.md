This page describes how to use the [Vertex AI text embedding](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) API to generate, store, and update text embeddings for data stored in Spanner for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

A text embedding is a vector representation of text data, and they are used in many ways to find similar items. You interact with them every time you complete a Google Search or see recommendations when shopping online. When you create text embeddings, you get vector representations of natural text as arrays of floating point numbers. This means that all of your input text is assigned a numerical representation. By comparing the numerical distance between the vector representations of two pieces of text, an application can determine the similarity between the text or the objects represented by the text.

With the Vertex AI text embeddings API, you can create a text embedding with [Generative AI](/vertex-ai/generative-ai/docs/overview) . In this tutorial, you use the Vertex AI text embedding model to generate text embeddings for the data stored in Spanner.

To learn more about text embeddings and supported models, see [Get text embeddings](/vertex-ai/docs/generative-ai/embeddings/get-text-embeddings) .

## Objective

In this tutorial, you learn how to:

  - Register a [Vertex AI text embedding model](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) in a Spanner schema using DDL statements.
  - Reference the registered model using SQL queries to generate embeddings from data stored in Spanner.

## Pricing

This tutorial uses billable components of Google Cloud, including:

  - Spanner
  - Vertex AI

For more information about Spanner costs, see the [Spanner pricing](/spanner/pricing) page.

For more information about Vertex AI costs, see the [Vertex AI pricing](/vertex-ai/pricing) page.

## Generate and store text embeddings

Depending on the model you use, generating embeddings might take some time. For more performance sensitive workloads, the best practice is to avoid generating embeddings in read-write transactions. Instead, generate the embeddings in a [read-only transaction](/spanner/docs/samples/spanner-read-only-transaction) using the following SQL examples.

### GoogleSQL

**Register a text embeddings model in Spanner**

In GoogleSQL, you must register a model before using it with the `  ML.PREDICT  ` function. To register the Vertex AI text embedding model in a Spanner database, [execute](/spanner/docs/schema-updates) the following DDL [statement](/spanner/docs/reference/standard-sql/data-definition-language#create_model) :

``` text
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
  endpoint = '//aiplatform.googleapis.com/projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME'
);
```

Replace the following:

  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model
  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint

Spanner grants appropriate permissions automatically. If it doesn't, review the [model endpoint access control](/spanner/docs/reference/standard-sql/data-definition-language#model_endpoint_access_control) .

Schema discovery and validation is not available for Generative AI models. You are required to provide `  INPUT  ` and `  OUTPUT  ` clauses which match against the models schema. For the full schema of the text embedding model, see [Get text embeddings](/vertex-ai/docs/generative-ai/embeddings/get-text-embeddings) .

**Generate text embeddings**

To generate embeddings, pass a piece of text directly to the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function using the following SQL:

``` text
SELECT embeddings.values
FROM ML.PREDICT(
  MODEL MODEL_NAME,
  (SELECT "A product description" as content)
);
```

To generate embeddings for data stored in a table, use the following SQL:

``` text
SELECT id, embeddings.values
FROM ML.PREDICT(
  MODEL MODEL_NAME,
  (SELECT id, description as content FROM Products)
);
```

To specify [task type](/vertex-ai/docs/generative-ai/embeddings/task-types) and [output dimensions](/vertex-ai/docs/generative-ai/embeddings/get-text-embeddings#choose_an_embedding_dimension) :

``` text
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

After generating the embeddings in a read-only transaction, store them in Spanner so they can be managed with your operational data. To store the embeddings, use a [read-write transaction](/spanner/docs/samples/spanner-read-write-transaction) .

For workloads that are less performance sensitive, you can generate and insert embeddings with the following SQL in a read-write transaction:

``` text
CREATE TABLE Products(
  id INT64 NOT NULL,
  description STRING(MAX),
  embeddings ARRAY<FLOAT32>,
) PRIMARY KEY(id);
```

``` text
INSERT INTO Products (id, description, embeddings)
SELECT @Id, @Description, embeddings.values
FROM ML.PREDICT(
  MODEL MODEL_NAME,
  (SELECT @Description as content)
);
```

### PostgreSQL

**Generate text embeddings**

To generate embeddings, pass a piece of text directly to the [`  spanner.ML_PREDICT_ROW  `](/spanner/docs/reference/postgresql/functions#ml) function using the following SQL:

``` text
SELECT
  spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    '{"instances": [{"content": "A product description"}]}'::jsonb
  ) ->'predictions'->0->'embeddings'->'values';
```

Replace the following:

  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint
  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

To generate embeddings for data stored in a table, use the following SQL:

``` text
SELECT id, spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', description))))
  ) -> 'predictions'->0->'embeddings'->'values'
FROM Products;
```

Replace the following:

  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint
  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

To specify [task type](/vertex-ai/docs/generative-ai/embeddings/task-types) and [output dimensions](/vertex-ai/docs/generative-ai/embeddings/get-text-embeddings#choose_an_embedding_dimension) :

``` text
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

  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint
  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

**Store text embeddings**

After generating the embeddings in a read-only transaction, store them in Spanner so they can be managed with your operational data. To store the embeddings, use a [read-write transaction](/spanner/docs/samples/spanner-read-write-transaction) .

For workloads that are less performance sensitive, you can generate and insert embeddings with the following SQL in a read-write transaction:

``` text
CREATE TABLE Products (
  id INT8 NOT NULL,
  description TEXT,
  embeddings REAL[],
  PRIMARY KEY(id)
);
```

``` text
INSERT INTO Products (id, description, embeddings)
SELECT @Id, @Description, spanner.FLOAT32_ARRAY(spanner.ML_PREDICT_ROW(
    'projects/PROJECT/locations/LOCATION/publishers/google/models/$MODEL_NAME',
    JSONB_BUILD_OBJECT('instances', JSONB_BUILD_ARRAY(JSONB_BUILD_OBJECT('content', @Description)))
  ) -> 'predictions'->0->'embeddings'->'values'
));
```

Replace the following:

  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint
  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

## Update text embeddings

To update your embeddings or to ingest data in realtime, use the `  UPDATE  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/dml-syntax#update-statement) and [PostgreSQL](/spanner/docs/reference/postgresql/dml-syntax#update-statement) ) statement.

To update the `  Products  ` table in the previous example, use the following SQL:

### GoogleSQL

``` text
UPDATE Products
SET
  description = @description,
  embeddings = (SELECT embeddings.values
                  FROM ML.PREDICT(MODEL MODEL_NAME, (SELECT @description as content))
              )
WHERE id = @id;
```

Replace the following:

  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

### PostgreSQL

``` text
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

  - `  PROJECT  ` : the project hosting the Vertex AI endpoint
  - `  LOCATION  ` : the location of the Vertex AI endpoint
  - `  MODEL_NAME  ` : the name of the Vertex AI text embedding model

## What's next

  - Learn [how to use Vertex AI Vector Search](/vertex-ai/docs/vector-search/overview) to search for semantically similar items.
  - Learn more about machine learning and embeddings in our [crash course on embeddings](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture) .
  - Learn more about [Vertex AI text embedding models](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) .
