Use the Dataflow connector to read data from multiple tables at the same point in time to ensure data consistency by performing all of the reads in a single transaction.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Import, export, and modify data using Dataflow](/spanner/docs/dataflow-connector)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
SpannerConfig spannerConfig =
    SpannerConfig.create().withInstanceId(instanceId).withDatabaseId(databaseId);
PCollectionView<Transaction> tx =
    pipeline.apply(
        SpannerIO.createTransaction()
            .withSpannerConfig(spannerConfig)
            .withTimestampBound(TimestampBound.strong()));
PCollection<Struct> singers =
    pipeline.apply(
        SpannerIO.read()
            .withSpannerConfig(spannerConfig)
            .withQuery("SELECT SingerID, FirstName, LastName FROM Singers")
            .withTransaction(tx));
PCollection<Struct> albums =
    pipeline.apply(
        SpannerIO.read()
            .withSpannerConfig(spannerConfig)
            .withQuery("SELECT SingerId, AlbumId, AlbumTitle FROM Albums")
            .withTransaction(tx));
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
