Use SQL queries with the Dataflow connector to read data from all available tables in a database.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Import, export, and modify data using Dataflow](/spanner/docs/dataflow-connector)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
PCollection<Struct> allRecords =
    pipeline
        .apply(
            SpannerIO.read()
                .withSpannerConfig(spannerConfig)
                .withBatching(false)
                .withQuery(
                    "SELECT t.table_name FROM information_schema.tables AS t WHERE t"
                        + ".table_catalog = '' AND t.table_schema = ''"))
        .apply(
            MapElements.into(TypeDescriptor.of(ReadOperation.class))
                .via(
                    (SerializableFunction<Struct, ReadOperation>)
                        input -> {
                          String tableName = input.getString(0);
                          return ReadOperation.create().withQuery("SELECT * FROM " + tableName);
                        }))
        .apply(SpannerIO.readAll().withSpannerConfig(spannerConfig));
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
