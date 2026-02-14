This page describes how to generate ML predictions using the Spanner emulator for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

Spanner Vertex AI integration can be used with the Spanner emulator to generate predictions using the GoogleSQL or PostgreSQL ML predict functions. The emulator is a binary that mimics a Spanner server, and can also be used in unit and integration testing. You can use the emulator as an [open source project](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator) or [locally](/spanner/docs/emulator) using the Google Cloud CLI. To learn more about the ML predict functions, see [How does Spanner Vertex AI integration work?](/spanner/docs/ml#how_does_work) .

You can use any model with the emulator to generate predictions. You can also use a model from the [Vertex AI Model Garden](/vertex-ai/docs/start/explore-models) or a model deployed to your [Vertex AI endpoint](/vertex-ai/docs/general/deployment) . Since the emulator doesn't connect to Vertex AI, the emulator can't verify the model or its schema for any model used from the Vertex AI Model Garden or deployed to the Vertex AI endpoints.

By default, when you use a prediction function with the emulator, the function yields a random value based on the provided model inputs and model output schema. You can use a callback function to modify the model input and output, and generate prediction results based on specific behaviors.

## Before you begin

Complete the following steps before you use the Spanner emulator to generate ML predictions.

### Install the Spanner emulator

You can either [install the emulator locally](/spanner/docs/emulator#install) or set it up using the [GitHub repository](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator) .

### Select a model

When you use the `  ML.PREDICT  ` (for GoogleSQL) or the `  ML_PREDICT_ROW  ` (for PostgreSQL) function, you must specify the location of the ML model. You can use any trained model. If you select a model that is running in the [Vertex AI Model Garden](/vertex-ai/docs/start/explore-models) or a model that is [deployed to your Vertex AI endpoint](/vertex-ai/docs/general/deployment) , you must provide the `  input  ` and `  output  ` values for these models.

To learn more about Spanner Vertex AI integration, see [How does Spanner Vertex AI integration work?](/spanner/docs/ml#how-does-it-work) .

## Generate predictions

You can use the emulator to generate predictions using the [Spanner ML predict functions](/spanner/docs/ml#ml-functions) .

### Default behavior

You can use any model deployed to an endpoint with the Spanner emulator to generate predictions. The following example uses a model called `  FraudDetection  ` to generate a result.

### GoogleSQL

To learn more about how to use the `  ML.PREDICT  ` function to generate predictions, see [Generate ML predictions using SQL](/spanner/docs/ml-tutorial#googlesql) .

**Register the model**

Before you can use a model with the [ML.PREDICT](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function, you must register the model using the [CREATE MODEL](/spanner/docs/reference/standard-sql/data-definition-language#create-model) statement and provide the `  input  ` and `  output  ` values:

``` text
CREATE MODEL FraudDetection
INPUT (Amount INT64, Name STRING(MAX))
OUTPUT (Outcome BOOL)
REMOTE OPTIONS (
endpoint = '//aiplatform.googleapis.com/projects/PROJECT_ID/locations/REGION_ID/endpoints/ENDPOINT_ID'
);
```

Replace the following:

  - `  PROJECT_ID  ` : the ID of the Google Cloud project that the model is located in

  - `  REGION_ID  ` : the ID of the Google Cloud region the model is located in—for example, `  us-central1  `

  - `  ENDPOINT_ID  ` : the ID of the model endpoint

**Run the prediction**

Use the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) GoogleSQL function to generate your prediction.

``` text
SELECT Outcome
FROM ML.PREDICT(
    MODEL FraudDetection,
    (SELECT 1000 AS Amount, "John Smith" AS Name))
```

The expected output of this query is `  TRUE  ` .

### PostgreSQL

To learn more about how to use the `  spanner.ML_PREDICT_ROW  ` function to generate predictions, see [Generate ML predictions using SQL](/spanner/docs/ml-tutorial#postgresql) .

**Run the prediction**

Use the `  spanner.ML_PREDICT_ROW  ` PostgreSQL function to generate your prediction.

``` text
SELECT (spanner.ml_predict_row(
'projects/`MODEL_ID`/locations/`REGION_ID`/endpoints/`ENDPOINT_ID`',
'{"instances": [{"Amount": "1000", "Name": "John Smith"}]}'
)->'predictions'->0->'Outcome')::boolean
```

Replace the following:

  - `  PROJECT_ID  ` : the ID of the Google Cloud project that the model is located in

  - `  REGION_ID  ` : the ID of the Google Cloud region the model is located in—for example, `  us-central1  `

  - `  ENDPOINT_ID  ` : the ID of the model endpoint

The expected output of this query is `  TRUE  ` .

### Custom Callback

You can use a custom callback function to implement selected model behaviors, and to transform specific model inputs to outputs. The following example uses the `  gemini-pro  ` model from the Vertex AI Model Garden and the Spanner emulator to generate predictions using a custom callback.

When using a custom callback for a model, you must [fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) the [Spanner emulator repository](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator) , then build and deploy it. For more information on how to build and deploy the Spanner emulator, see the [Spanner emulator quickstart](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator?tab=readme-ov-file#quickstart) .

### GoogleSQL

**Register the model**

Before you can use a model with the [ML.PREDICT](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function, you must register the model using the [CREATE MODEL](/spanner/docs/reference/standard-sql/data-definition-language#create-model) statement:

``` text
CREATE MODEL GeminiPro
INPUT (prompt STRING(MAX))
OUTPUT (content STRING(MAX))
REMOTE OPTIONS (
endpoint = '//aiplatform.googleapis.com/projects/PROJECT_ID/locations/REGION_ID/publishers/google/models/gemini-pro',
default_batch_size = 1
);
```

Since the emulator doesn't connect to the Vertex AI, you must provide the `  input  ` and `  output  ` values.

Replace the following:

  - `  PROJECT_ID  ` : the ID of the Google Cloud project that the model is located in

  - `  REGION_ID  ` : the ID of the Google Cloud region the model is located in—for example, `  us-central1  `

**Callback**

Use a callback to add custom logic to the `  GeminiPro  ` model.

``` text
absl::Status ModelEvaluator::Predict(
    const googlesql::Model* model,
    const CaseInsensitiveStringMap<const ModelColumn>& model_inputs,
    CaseInsensitiveStringMap<ModelColumn>& model_outputs) {
  // Custom logic for GeminiPro.
  if (model->Name() == "GeminiPro") {
    RET_CHECK(model_inputs.contains("prompt"));
    RET_CHECK(model_inputs.find("prompt")->second.value->type()->IsString());
    RET_CHECK(model_outputs.contains("content"));
    std::string content;

    // Process prompts used in tests.
    int64_t number;
    static LazyRE2 is_prime_prompt = {R"(Is (\d+) a prime number\?)"};
    if (RE2::FullMatch(
            model_inputs.find("prompt")->second.value->string_value(),
            *is_prime_prompt, &number)) {
        content = IsPrime(number) ? "Yes" : "No";
    } else {
        // Default response.
        content = "Sorry, I don't understand";
    }
    *model_outputs["content"].value = googlesql::values::String(content);
    return absl::OkStatus();
  }
  // Custom model prediction logic can be added here.
  return DefaultPredict(model, model_inputs, model_outputs);
}
```

**Run the prediction**

Use the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) GoogleSQL function to generate your prediction.

``` text
SELECT content
    FROM ML.PREDICT(MODEL GeminiPro, (SELECT "Is 7 a prime number?" AS prompt))
```

The expected output of this query is `  "YES"  ` .

### PostgreSQL

Use the `  spanner.ML_PREDICT_ROW  ` PostgreSQL function to generate your prediction.

**Callback**

Use a callback to add custom logic to the `  GeminiPro  ` model.

``` text
absl::Status ModelEvaluator::PgPredict(
    absl::string_view endpoint, const googlesql::JSONValueConstRef& instance,
    const googlesql::JSONValueConstRef& parameters,
    lesql::JSONValueRef prediction) {
  if (endpoint.ends_with("publishers/google/models/gemini-pro")) {
    RET_CHECK(instance.IsObject());
    RET_CHECK(instance.HasMember("prompt"));
    std::string content;

    // Process prompts used in tests.
    int64_t number;
    static LazyRE2 is_prime_prompt = {R"(Is (\d+) a prime number\?)"};
    if (RE2::FullMatch(instance.GetMember("prompt").GetString(),
                        *is_prime_prompt, &number)) {
        content = IsPrime(number) ? "Yes" : "No";
    } else {
        // Default response.
        content = "Sorry, I don't understand";
    }
    prediction.SetToEmptyObject();
    prediction.GetMember("content").SetString(content);
    return absl::OkStatus();
  }

  // Custom model prediction logic can be added here.
  return DefaultPgPredict(endpoint, instance, parameters, prediction);
}
```

**Run the prediction**

``` text
SELECT (spanner.ml_predict_row(
'projects/`PROJECT_ID`/locations/`REGION_ID`/publishers/google/models/gemini-pro',
'{"instances": [{"prompt": "Is 7 a prime number?"}]}'
)->'predictions'->0->'content')::text
```

Replace the following:

  - `  PROJECT_ID  ` : the ID of the Google Cloud project that the model is located in

  - `  REGION_ID  ` : the ID of the Google Cloud region the model is located in—for example, `  us-central1  `

The expected output of this query is `  "YES"  ` .

## What's next?

  - [Generate ML predictions using SQL](/spanner/docs/ml-tutorial) .
  - [Get Vertex AI text embeddings](/spanner/docs/ml-tutorial-embeddings) .
