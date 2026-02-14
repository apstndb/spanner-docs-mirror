Use a SQL query with the Dataflow connector to read data from all the columns and rows in the specified table.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Import, export, and modify data using Dataflow](/spanner/docs/dataflow-connector)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
// Query for all the columns and rows in the specified Spanner table
PCollection<Struct> records = pipeline.apply(
    SpannerIO.read()
        .withInstanceId(instanceId)
        .withDatabaseId(databaseId)
        .withQuery("SELECT * FROM " + options.getTable()));
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
