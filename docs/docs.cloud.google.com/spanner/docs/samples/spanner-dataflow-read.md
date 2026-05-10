---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-dataflow-read
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-dataflow-read
title: Dataflow read
description: Use a SQL query with the Dataflow connector to read data from all the columns and rows in the specified table.
data_source: docs.cloud.google.com
update_time: "2026-05-08T21:33:00Z"
---

Use a SQL query with the Dataflow connector to read data from all the columns and rows in the specified table.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Import, export, and modify data using Dataflow](https://docs.cloud.google.com/spanner/docs/dataflow-connector)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    // Query for all the columns and rows in the specified Spanner table
    PCollection<Struct> records = pipeline.apply(
        SpannerIO.read()
            .withInstanceId(instanceId)
            .withDatabaseId(databaseId)
            .withQuery("SELECT * FROM " + options.getTable()));

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=spanner) .
