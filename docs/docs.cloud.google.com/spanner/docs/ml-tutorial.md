This page shows how to generate machine learning (ML) predictions from a Spanner database. ML predictions work with both GoogleSQL-dialect databases and PostgreSQL-dialect databases.

Spanner Vertex AI integration gives you the ability to generate predictions using your SQL code by calling the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function for GoogleSQL or the [`  spanner.ML_PREDICT_ROW  `](/spanner/docs/reference/postgresql/functions-and-operators#ml) function for PostgreSQL. For more information about Spanner Vertex AI integration, see [Spanner Vertex AI integration overview](/spanner/docs/ml) .

## Before you begin

Before you can generate predictions from a Spanner instance, you must prepare your database and select a model.

### Configure access for Spanner Vertex AI integration to Vertex AI endpoints

Spanner creates the service agent and grants the necessary permissions automatically when Spanner executes the first MODEL DDL statement. If both the Spanner database and the Vertex AI endpoint are in the same project, no additional setup is required.

If the Spanner service agent account doesn't exist for your Spanner project, [create](/sdk/gcloud/reference/beta/services/identity/create) it by running the following command:

``` text
gcloud beta services identity create --service=spanner.googleapis.com --project={PROJECT}`
```

Follow the steps described in the [Grant a single role](/iam/docs/granting-changing-revoking-access#iam-grant-single-role-gcloud) to grant the [`  Spanner API Service Agent  `](/iam/docs/roles-permissions/spanner#spanner.serviceAgent) role to the Spanner [service agent](/iam/docs/service-agents) account `  service- PROJECT_NUMBER @gcp-sa-spanner.iam.gserviceaccount.com  ` on your Vertex AI project.

### Select a model

When you use the `  ML.PREDICT  ` (for GoogleSQL) or the `  spanner.ML_PREDICT_ROW  ` (for PostgreSQL) function, you must specify the location of the ML model. Your selected model can be one of the following:

  - A model running in the [Vertex AI Model Garden](/vertex-ai/docs/start/explore-models) .

  - A Vertex AI model with an active endpoint that your [Spanner service agent](/iam/docs/service-agents) has IAM permission to access.

To learn more about Spanner Vertex AI integration, see [How does Spanner Vertex AI integration work?](/spanner/docs/ml#how-does-it-work) .

## Generate predictions

Depending on the type of your selected model, the steps to generating your predictions will differ.

### Use a model in the Vertex AI Model Garden

To generate a prediction using a model from the Vertex AI Model Garden, [select a model from the Model Garden](/vertex-ai/docs/start/explore-models) .

### GoogleSQL

Before you use a model with `  ML.PREDICT()  ` , you need to register the model using the `  CREATE MODEL  ` statement.

``` text
CREATE MODEL 'MODEL_NAME'
INPUT (INPUT_COLUMN_NAME INPUT_COLUMN_TYPE)
OUTPUT (OUTPUT_COLUMN_NAME OUTPUT_COLUMN_TYPE)
REMOTE
OPTIONS (
  endpoint = '//aiplatform.googleapis.com/projects/PROJECT_ID/locations/LOCATION_ID/publishers/google/models/model_id'
);
```

Replace the following:

  - `  MODEL_NAME  ` : the name you want to give your model

  - `  INPUT_COLUMN_NAME  ` : the name of your input column. For example, if using the `  gemini-pro  ` model, the input column name is `  prompt  ` .

  - `  INPUT_COLUMN_TYPE  ` : the data type for `  INPUT_COLUMN_NAME  `

  - `  OUTPUT_COLUMN_NAME  ` : the name of your output column. For example, if using the `  gemini-pro  ` model, the output column name is `  content  ` .

  - `  OUTPUT_COLUMN_TYPE  ` : the data type for `  OUTPUT_COLUMN_NAME  `

  - `  PROJECT_ID  ` : the ID of your Google Cloud project

  - `  REGION_ID  ` : the ID of the Google Cloud region that the model is located in—for example, `  us-central1  `

  - `  MODEL_ID  ` : the ID of the ML model you want to use—for example, `  gemini-pro  `
    
    For more information about models, see [Model API reference for Generative AI](/vertex-ai/docs/generative-ai/model-reference/overview#models) .

Use the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) GoogleSQL function with the model selected from the Model Garden to generate your prediction.

``` text
SELECT * FROM ML.PREDICT(
  MODEL `MODEL_NAME`,
  `INPUT_RELATION`[, `PARAMETERS`])
```

Replace the following:

  - `  MODEL_NAME  ` : the name you want to give your model
    
    For more information about models, see [Model API reference for Generative AI](/vertex-ai/docs/generative-ai/model-reference/overview#models) .

  - `  INPUT_RELATION  ` : either `  TABLE table_name  ` or a subquery the table or subquery supplying data to run the ML prediction on.

  - `  PARAMETERS  ` : a `  STRUCT  ` value that contains parameters supported by `  model_id  ` .

You can also use `  SAFE.ML.PREDICT  ` to return `  null  ` instead of an error in your predictions. This is helpful in cases when running large queries where some failed predictions are tolerable.

### PostgreSQL

Use the `  ML_PREDICT_ROW  ` PostgreSQL function with the model selected from the Model Garden to generate your prediction.

``` text
SELECT spanner.ml_predict_row(
  'projects/PROJECT_ID/locations/REGION_ID/publishers/google/models/MODEL_ID'::text,
  '{
    "instances": [ INSTANCES ],
    "parameters": { PARAMETERS }
   }'::jsonb);
```

Replace the following:

  - `  PROJECT_ID  ` : the ID of your Google Cloud project

  - `  REGION_ID  ` : the ID of the Google Cloud region that the model is located in—for example, `  us-central1  `

  - `  MODEL_ID  ` : the ID of the ML model you want to use—for example, `  gemini-pro  `
    
    For more information about models, see [Model API reference for Generative AI](/vertex-ai/docs/generative-ai/model-reference/overview#models) .

  - `  INSTANCES  ` : the inputs for the prediction call, in JSON format

  - `  PARAMETERS  ` : optional parameters for the prediction call, in JSON format

This query produces a JSON response. For more information about the model's JSON response messages, see [PredictResponse](/vertex-ai/docs/reference/rest/v1/PredictResponse) .

### Use a Vertex AI model endpoint

To use a trained or downloaded model with Spanner Vertex AI integration, you need to deploy the model to Vertex AI. For more information on how to deploy a model to an endpoint in Vertex AI, see [Deploy a model to an endpoint](/vertex-ai/docs/general/deployment) .

### GoogleSQL

Use the `  ML.PREDICT  ` GoogleSQL function with the model in a Vertex AI endpoint to generate your prediction. Before you use a model with `  ML.PREDICT()  ` , you need to register the model using the [`  CREATE MODEL  `](/spanner/docs/reference/standard-sql/data-definition-language#create-model) statement. Each deployed model has its own unique schema. The following is an example schema from [Classification and regression Overview](/vertex-ai/docs/tabular-data/classification-regression/overview)

``` text
CREATE MODEL MyClassificationModel
INPUT (
  length FLOAT64,
  material STRING(MAX),
  tag_array ARRAY<STRING(MAX)>
)
OUTPUT (
  scores ARRAY<FLOAT64>,
  classes ARRAY<STRING(MAX)>
)
REMOTE
OPTIONS (
  endpoint = '//aiplatform.googleapis.com/projects/PROJECT/locations/LOCATION/endpoints/ENDPOINT_ID'
)
```

Replace the following:

  - `  PROJECT_ID  ` : the ID of your Google Cloud project

  - `  LOCATION  ` : the ID of the Google Cloud region that the model is located in—for example, `  us-central1  `

  - `  ENDPOINT_ID  ` : the ID of the ML model you want to use—for example, `  gemini-pro  `
    
    For more information about models, see [Model API reference for Generative AI](/vertex-ai/docs/generative-ai/model-reference/overview#models) .

Use the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) GoogleSQL function with the model selected from the Model Garden to generate your prediction.

``` text
SELECT * FROM ML.PREDICT(
  `MODEL_ID`,
  `INPUT_RELATION`[, `PARAMETERS`])
```

Replace the following:

  - `  MODEL_ID  ` : the ID of the ML model you want to use.

  - `  INPUT_RELATION  ` : the table or subquery that you want to run the ML prediction on.

  - `  PARAMETERS  ` : a `  STRUCT  ` value that contains parameters supported by `  model_name  ` .

This query produces a relation containing all output columns of the model and all columns of the input relation.

### PostgreSQL

Use the `  ML.PREDICT  ` PostgreSQL function with the model in a Vertex AI endpoint to generate your prediction.

```` text
  SELECT spanner.ml_predict_row(
    'projects/PROJECT_ID/locations/REGION_ID/endpoints/ENDPOINT_ID'::text,
    '{
      "instances": [ INSTANCES ],
      "parameters": { PARAMETERS }
      }'::jsonb);
  ```

Replace the following:
````

  - `  PROJECT_ID  ` : the ID of the Google Cloud project that the model is located in

  - `  REGION_ID  ` : the ID of the Google Cloud region the model is located in—for example, `  us-central1  `

  - `  ENDPOINT_ID  ` : the ID of the model endpoint

  - `  INSTANCES  ` : the inputs to the prediction call, in JSON format

  - `  PARAMETERS  ` : optional parameters to the prediction call, in JSON format

This query produces a JSON response. For more information about the model's JSON response messages, see [PredictResponse](/vertex-ai/docs/reference/rest/v1/PredictResponse) .

### Examples for using ML functions to generate predictions

**Preview — [Gemini](/gemini/docs/overview) in Spanner**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

The following example uses the [gemini-pro](/vertex-ai/docs/generative-ai/model-reference/gemini) model, from the Model Garden, to generate text based on a short prompt that is provided as an argument. This model is available as part of Gemini in Spanner.

### GoogleSQL

**Register the `  gemini-pro  ` model**

``` text
CREATE MODEL GeminiPro
INPUT (prompt STRING(MAX))
OUTPUT (content STRING(MAX))
REMOTE
OPTIONS (
  endpoint = '//aiplatform.googleapis.com/projects/PROJECT/locations/LOCATION/publishers/google/models/gemini-pro',
  default_batch_size = 1
);
```

Replace the following:

  - `  PROJECT  ` : the project ID
  - `  LOCATION  ` : the region where you are using Vertex AI

**Run the model**

``` text
SELECT content
FROM ML.PREDICT(
  MODEL GeminiPro,
  (SELECT "Is 7 a prime number?" AS prompt),
  STRUCT(256 AS maxOutputTokens, 0.2 AS temperature, 40 as topK, 0.95 AS topP)
);
```

**Expected output**

The expected out is as follows:

``` text
+--------------------+
| content            |
+--------------------+
| "Yes"              |
+--------------------+
```

### PostgreSQL

**Run the model**

``` text
select spanner.ml_predict_row(
  '{
    "endpoint": "projects/PROJECT_ID/locations/us-central1/publishers/google/models/gemini-pro",
    "default_batch_size": 1
   }'::jsonb,
  '{
    "instances":[{"prompt": "Is 7 a prime number?"}],
    "parameters":{"maxOutputTokens":256, "topK": 40, "topP":0.96, "temperature":0.2}
    }'
);
```

**Expected output**

The expected out is the following:

``` text
+--------------------+
| content            |
+--------------------+
| "Yes"              |
+--------------------+
```
