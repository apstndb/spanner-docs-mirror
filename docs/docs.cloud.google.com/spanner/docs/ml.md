This page provides an overview of Spanner Agent Platform integration. Spanner Agent Platform integration works with both GoogleSQL and PostgreSQL databases.

Spanner Agent Platform integration helps you to access classifier and regression ML models hosted on [Vertex AI](https://docs.cloud.google.com/vertex-ai) through the GoogleSQL and PostgreSQL interface. This helps to seamlessly integrate ML predictions serving functionality with general Spanner data access operations performed using DQL/DML queries.

## Benefits of Spanner Agent Platform integration

Generating ML predictions using Spanner Agent Platform integration provides multiple benefits compared to the approach where Spanner data access and access to the Vertex AI prediction endpoint are performed separately:

  - Performance:
      - Better latency: Spanner Agent Platform integration talking to the Vertex AI service directly eliminates additional round-trips between a compute node running a Spanner's client and the Vertex AI service.
      - Better throughput/parallelism: Spanner Agent Platform integration runs on top of Spanner's distributed query processing infrastructure, which supports highly parallelizable query execution.
  - User experience:
      - Ability to use a single, simple, coherent, and familiar SQL interface to facilitate both data transformation and ML serving scenarios on Spanner level of scale lowers the ML entry barrier and allows for a much smoother user experience.
  - Costs:
      - Spanner Agent Platform integration uses Spanner [compute capacity](https://docs.cloud.google.com/spanner/docs/compute-capacity) to merge the results of ML computations and SQL query execution, which eliminates the need to provision an additional compute (for example, in Compute Engine or Google Kubernetes Engine) for that.

## How does Spanner Agent Platform integration work?

Spanner Agent Platform integration doesn't host ML models, but relies on the Vertex AI service infrastructure instead. You don't need to train a model using Vertex AI to use it with Spanner Agent Platform integration, but you must deploy it to a Vertex AI endpoint.

To train models on data stored in Spanner, you can use the following:

  - [BigQuery Federated queries](https://docs.cloud.google.com/bigquery/docs/spanner-federated-queries) together with [BigQuery ML](https://docs.cloud.google.com/bigquery/docs/bqml-introduction) .

  - [Dataflow](https://docs.cloud.google.com/dataflow) to export data from Spanner into CSV format and import the [CSV data source](https://docs.cloud.google.com/vertex-ai/docs/datasets/prepare-tabular#csv) into Vertex AI.

Spanner Agent Platform integration extends the following functions for using ML models:

  - **Generate ML predictions** by calling a model using SQL on your Spanner data. You can use a model from the [Vertex AI Model Garden](https://docs.cloud.google.com/vertex-ai/docs/start/explore-models) or a model deployed to your [Vertex AI endpoint](https://docs.cloud.google.com/vertex-ai/docs/general/deployment) .

  - **Generate text embeddings** to have an LLM translate text prompts into numbers. To learn more about embeddings, see [Get text embeddings](https://docs.cloud.google.com/vertex-ai/docs/generative-ai/embeddings/get-text-embeddings) .

## Using Spanner Agent Platform integration functions

A model in Spanner Agent Platform integration can be used to generate predictions or text embeddings in your SQL code using the ML Predict functions. These functions are as follows:

### GoogleSQL

You can use the following ML predict function for GoogleSQL:

[`ML.PREDICT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/ml-functions#mlpredict)

You need to register your model using the [`CREATE MODEL`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create_model) DDL statement before using it with the `ML.PREDICT` function.

You can also use `SAFE.ML.PREDICT` to return `null` instead of an error in your predictions. This is helpful in cases when running large queries where some failed predictions are tolerable.

### PostgreSQL

You can use the following ML predict function for PostgreSQL:

[`spanner.ML_PREDICT_ROW`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions-and-operators#ml)

To use the functions, you can select a model from the [Vertex AI Model Garden](https://docs.cloud.google.com/vertex-ai/docs/start/explore-models) or use a model that you've deployed to Vertex AI.

For more information on how to deploy a model to an endpoint in Vertex AI, see [Deploy a model to an endpoint](https://docs.cloud.google.com/vertex-ai/docs/general/deployment) .

For more information on how to use these functions to generate an ML prediction, see [Generate ML predictions using SQL](https://docs.cloud.google.com/spanner/docs/ml-tutorial) .

For more information on how to use these functions to generate text embeddings, see [Get text embeddings](https://docs.cloud.google.com/vertex-ai/docs/generative-ai/embeddings/get-text-embeddings) .

## Pricing

There are no additional charges from Spanner when you use it with Spanner Agent Platform integration. However, there are other potential charges associated with this feature:

  - You pay the [standard rates](https://docs.cloud.google.com/vertex-ai/pricing) for Vertex AI online prediction. The total charge depends on the model type you use. Some model types have a flat per hour rate, depending on the machine type and number of nodes that you use. Some model types have per call rates. We recommend you deploy the latter in a dedicated project where you have set explicit prediction quotas.

  - You pay the [standard rates](https://docs.cloud.google.com/spanner/pricing#network) for data transfer between Spanner and Vertex AI. The total charge depends on the region hosting the server that executes the query and the region hosting the called endpoint. To minimize charges, deploy your Vertex AI endpoints in the same region as your Spanner instance. When using multi-regional instance configurations or multiple Vertex AI endpoints, deploy your endpoints on the same continent.

## SLA

Due to [Vertex AI online prediction availability](https://docs.cloud.google.com/vertex-ai/sla) being lower, you must properly configure Spanner ML models to maintain [Spanner's high availability](https://docs.cloud.google.com/spanner/sla) while using Spanner Agent Platform integration:

1.  Spanner ML models must use multiple Vertex AI endpoints on the backend to enable failover.
2.  Vertex AI endpoints must conform to the [Vertex AI SLA](https://docs.cloud.google.com/vertex-ai/sla) .
3.  Vertex AI endpoints must provision enough capacity to handle incoming traffic.
4.  Vertex AI endpoints must use separate regions close to the Spanner database to avoid regional outages.
5.  Vertex AI endpoints should use separate projects to avoid issues with per-project prediction quotas.

The number of redundant Vertex AI endpoints depends on their SLA, and the number of rows in Spanner queries:

| Spanner SLA | Vertex AI SLA | 1 row | 10 rows | 100 rows | 1000 rows |
| ----------- | ------------- | ----- | ------- | -------- | --------- |
| 99.99%      | 99.9%         | 2     | 2       | 2        | 3         |
| 99.99%      | 99.5%         | 2     | 3       | 3        | 4         |
| 99.999%     | 99.9%         | 2     | 2       | 3        | 3         |
| 99.999%     | 99.5%         | 3     | 3       | 4        | 4         |

Vertex AI endpoints don't need to host exactly the same model. We recommend that you configure the Spanner ML model to have a primary, complex and compute intensive model as its first endpoint. Subsequent failover endpoints can point to simplified models that are less compute intensive, scale better and can absorb traffic spikes.

## Limitations

  - Model input and output must be a JSON object.

## Compliance

[Assured Workloads](https://docs.cloud.google.com/assured-workloads/docs/overview) don't support the Vertex AI Prediction API. Enabling a [restrict resource usage constraint](https://docs.cloud.google.com/assured-workloads/docs/restrict-resource-usage) disables the Vertex AI API and effectively the Spanner Agent Platform integration feature.

Additionally, we recommend that you create a [VPC Service Controls perimeter](https://docs.cloud.google.com/assured-workloads/docs/configure-vpc-sc) to ensure your production databases cannot connect to Vertex AI endpoints in your non-production projects that might not have the proper compliance configuration.
