Google Cloud provides [open source Dataflow templates](https://github.com/GoogleCloudPlatform/DataflowTemplates) that you can use to create Dataflow jobs. These jobs are based on prebuilt Docker images for common use cases using the Google Cloud console, the Google Cloud CLI, or REST API calls. This page lists the available, Spanner-related Dataflow templates.

## Batch templates

Use the following templates to process data in bulk:

  - [Any source database to Spanner](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/sourcedb-to-spanner)
  - [Cloud Storage Avro to Spanner](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/avro-to-cloud-spanner)
  - [Cloud Storage Text to Spanner](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-storage-to-cloud-spanner)
  - [Spanner to BigQuery](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-spanner-to-bigquery)
  - [Spanner to Cloud Storage Avro](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-spanner-to-avro)
  - [Spanner to Cloud Storage Text](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-spanner-to-cloud-storage)
  - [Spanner to Agent Platform Vector Search files on Cloud Storage](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-spanner-to-vertex-vector-search)

For more information about using these templates, see [Import, export, and modify data using Dataflow](https://docs.cloud.google.com/spanner/docs/dataflow-connector) .

## Streaming templates

Use the following templates to process data continuously:

  - [Datastream to Spanner](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/datastream-to-cloud-spanner)
  - [Spanner change streams to any source database](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/spanner-to-sourcedb)
  - [Spanner change streams to BigQuery](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-spanner-change-streams-to-bigquery)
  - [Spanner change streams to Cloud Storage](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-spanner-change-streams-to-cloud-storage)
  - [Spanner change streams to Pub/Sub](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided/cloud-spanner-change-streams-to-pubsub)

For more information about using these templates, see [Build change streams connections using Dataflow](https://docs.cloud.google.com/spanner/docs/change-streams/use-dataflow) .

## What's next

  - [View the complete list of Google-provided templates.](https://docs.cloud.google.com/dataflow/docs/guides/templates/provided-templates)
  - [Learn more about Spanner change streams.](https://docs.cloud.google.com/spanner/docs/change-streams)
