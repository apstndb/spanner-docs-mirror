Use the Dataflow connector to write data by using mutations.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Import, export, and modify data using Dataflow](/spanner/docs/dataflow-connector)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
albums
    // Spanner expects a Mutation object, so create it using the Album's data
    .apply("CreateAlbumMutation", ParDo.of(new DoFn<Album, Mutation>() {
      @ProcessElement
      public void processElement(ProcessContext c) {
        Album album = c.element();
        c.output(Mutation.newInsertOrUpdateBuilder("albums")
            .set("singerId").to(album.singerId)
            .set("albumId").to(album.albumId)
            .set("albumTitle").to(album.albumTitle)
            .build());
      }
    }))
    // Write mutations to Spanner
    .apply("WriteAlbums", SpannerIO.write()
        .withInstanceId(instanceId)
        .withDatabaseId(databaseId));
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
