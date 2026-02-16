This page describes how to use the Dataflow connector for Spanner to import, export, and modify data in Spanner GoogleSQL-dialect databases and PostgreSQL-dialect databases.

[Dataflow](/dataflow) is a managed service for transforming and enriching data. The Dataflow connector for Spanner lets you read data from and write data to Spanner in a Dataflow pipeline, optionally transforming or modifying the data. You can also create pipelines that transfer data between Spanner and other Google Cloud products.

The Dataflow connector is the recommended method for efficiently moving data into and out of Spanner in bulk. It's also the recommended method for performing large transformations to a database which are not supported by [Partitioned DML](/spanner/docs/dml-partitioned) , such as table moves and bulk deletes that require a JOIN. When working with individual databases, there are other methods you can use to import and export data:

  - Use the Google Cloud console to [export](/spanner/docs/export) an individual database from Spanner to Cloud Storage in [Avro](https://en.wikipedia.org/wiki/Apache_Avro) format.
  - Use the Google Cloud console to [import](/spanner/docs/import) a database back into Spanner from files you exported to Cloud Storage.
  - Use the REST API or Google Cloud CLI to run [export](/dataflow/docs/templates/provided-templates#cloud_spanner_to_gcs_avro) or [import](/dataflow/docs/templates/provided-templates#gcs_avro_to_cloud_spanner) jobs from Spanner to Cloud Storage and back also using Avro format.

The Dataflow connector for Spanner is part of the [Apache Beam Java SDK](https://beam.apache.org/documentation/sdks/javadoc/current/) , and it provides an API for performing the previous actions. For more information about some of the concepts discussed in this page, such as `  PCollection  ` objects and transforms, see the [Apache Beam programming guide](https://beam.apache.org/documentation/programming-guide/) .

**Note:** The Dataflow connector for Spanner only supports the Dataflow SDK 2.x for Java. For more information, see [Release notes: Dataflow SDK 2.x for Java](/dataflow/release-notes/release-notes-java-2) .

## Add the connector to your Maven project

To add the Google Cloud Dataflow connector to a Maven project, add the `  beam-sdks-java-io-google-cloud-platform  ` Maven artifact to your `  pom.xml  ` file as a dependency.

For example, assuming that your `  pom.xml  ` file sets `  beam.version  ` to the appropriate version number, you would add the following dependency:

``` text
<dependency>
    <groupId>org.apache.beam</groupId>
    <artifactId>beam-sdks-java-io-google-cloud-platform</artifactId>
    <version>${beam.version}</version>
</dependency>
```

## Read data from Spanner

To read from Spanner, apply the [`  SpannerIO.read  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.html#read--) transform. Configure the read using the methods in the [`  SpannerIO.Read  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html) class. Applying the transform returns a [`  PCollection<Struct>  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/values/PCollection.html) , where each element in the collection represents an individual row returned by the read operation. You can read from Spanner with and without a specific SQL query, depending on your needed output.

Applying the `  SpannerIO.read  ` transform returns a consistent view of data by performing a strong read. Unless you specify otherwise, the result of the read is snapshotted at the time that you started the read. See [reads](/spanner/docs/reads#read_types) for more information about the different types of reads Spanner can perform.

### Read data using a query

To read a specific set of data from Spanner, configure the transform using the [`  SpannerIO.Read.withQuery  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html#withQuery-java.lang.String-) method to specify a SQL query. For example:

``` java
// Query for all the columns and rows in the specified Spanner table
PCollection<Struct> records = pipeline.apply(
    SpannerIO.read()
        .withInstanceId(instanceId)
        .withDatabaseId(databaseId)
        .withQuery("SELECT * FROM " + options.getTable()));
```

### Read data without specifying a query

To read from a database without using a query, you can specify a table name using the [`  SpannerIO.Read.withTable  `](https://beam.apache.org/releases/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html#withTable-java.lang.String-) method, and specify a list of columns to read using the [`  SpannerIO.Read.withColumns  `](https://beam.apache.org/releases/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html#withColumns-java.util.List-) method. For example:

### GoogleSQL

``` java
// Query for all the columns and rows in the specified Spanner table
PCollection<Struct> records = pipeline.apply(
    SpannerIO.read()
        .withInstanceId(instanceId)
        .withDatabaseId(databaseId)
        .withTable("Singers")
        .withColumns("singerId", "firstName", "lastName"));
```

### PostgreSQL

``` java
// Query for all the columns and rows in the specified Spanner table
PCollection<Struct> records = pipeline.apply(
    SpannerIO.read()
        .withInstanceId(instanceId)
        .withDatabaseId(databaseId)
        .withTable("singers")
        .withColumns("singer_id", "first_name", "last_name"));
```

To limit the rows read, you can specify a set of primary keys to read using the [`  SpannerIO.Read.withKeySet  `](https://beam.apache.org/releases/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html#withKeySet-com.google.cloud.spanner.KeySet-) method.

You can also read a table using a specified secondary index. As with the [`  readUsingIndex  ` API call](/spanner/docs/secondary-indexes#read-with-index) , the index must contain all of the data that appears in the query results.

To do so, specify the table as shown in the previous example, and specify the [index](/spanner/docs/secondary-indexes) that contains the needed column values using the [`  SpannerIO.Read.withIndex  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html#withIndex-java.lang.String-) method. The index must store all the columns that the transform needs to read. The base table's primary key is implicitly stored. For example, to read the `  Songs  ` table using the index [`  SongsBySongName  `](/spanner/docs/secondary-indexes#add-index) , you use the following code:

### GoogleSQL

``` java
// Read the indexed columns from all rows in the specified index.
PCollection<Struct> records =
    pipeline.apply(
        SpannerIO.read()
            .withInstanceId(instanceId)
            .withDatabaseId(databaseId)
            .withTable("Songs")
            .withIndex("SongsBySongName")
            // Can only read columns that are either indexed, STORED in the index or
            // part of the primary key of the Songs table,
            .withColumns("SingerId", "AlbumId", "TrackId", "SongName"));
```

### PostgreSQL

``` java
// // Read the indexed columns from all rows in the specified index.
PCollection<Struct> records =
    pipeline.apply(
        SpannerIO.read()
            .withInstanceId(instanceId)
            .withDatabaseId(databaseId)
            .withTable("Songs")
            .withIndex("SongsBySongName")
            // Can only read columns that are either indexed, STORED in the index or
            // part of the primary key of the songs table,
            .withColumns("singer_id", "album_id", "track_id", "song_name"));
```

**Note:** You can't use the `  SpannerIO.Read  ` `  withQuery  ` and `  withTable  ` methods together. This is because `  withQuery  ` overrides values that you pass into the `  withTable  ` method.

### Control the staleness of transaction data

A transform is guaranteed to be executed on a consistent snapshot of data. To control the [staleness](/spanner/docs/timestamp-bounds#timestamp_bound_types) of data, use the [`  SpannerIO.Read.withTimestampBound  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html#withTimestampBound-com.google.cloud.spanner.TimestampBound-) method. See [transactions](/spanner/docs/transactions) for more information.

### Read from multiple tables in the same transaction

If you want to read data from multiple tables at the same point in time to ensure data consistency, perform all of the reads in a single transaction. To do this, apply a [`  createTransaction  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.CreateTransaction.html) transform, creating a `  PCollectionView<Transaction>  ` object which then creates a transaction. The resulting view can be passed to a read operation using [`  SpannerIO.Read.withTransaction  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Read.html#withTransaction-org.apache.beam.sdk.values.PCollectionView-) .

### GoogleSQL

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

### PostgreSQL

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
            .withQuery("SELECT singer_id, first_name, last_name FROM singers")
            .withTransaction(tx));
PCollection<Struct> albums =
    pipeline.apply(
        SpannerIO.read()
            .withSpannerConfig(spannerConfig)
            .withQuery("SELECT singer_id, album_id, album_title FROM albums")
            .withTransaction(tx));
```

### Read data from all available tables

You can read data from all available tables in a Spanner database.

### GoogleSQL

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

### PostgreSQL

``` java
PCollection<Struct> allRecords =
    pipeline
        .apply(
            SpannerIO.read()
                .withSpannerConfig(spannerConfig)
                .withBatching(false)
                .withQuery(
                    Statement.newBuilder(
                            "SELECT t.table_name FROM information_schema.tables AS t "
                                + "WHERE t.table_catalog = $1 AND t.table_schema = $2")
                        .bind("p1")
                        .to(spannerConfig.getDatabaseId().get())
                        .bind("p2")
                        .to("public")
                        .build()))
        .apply(
            MapElements.into(TypeDescriptor.of(ReadOperation.class))
                .via(
                    (SerializableFunction<Struct, ReadOperation>)
                        input -> {
                          String tableName = input.getString(0);
                          return ReadOperation.create()
                              .withQuery("SELECT * FROM \"" + tableName + "\"");
                        }))
        .apply(SpannerIO.readAll().withSpannerConfig(spannerConfig));
```

### Troubleshoot unsupported queries

The Dataflow connector only supports Spanner SQL queries where the first operator in the query execution plan is a [**Distributed Union**](/spanner/docs/query-execution-operators#distributed_union) . If you attempt to read data from Spanner using a query and you get an exception stating that the query `  does not have a DistributedUnion at the root  ` , follow the steps in [Understand how Spanner executes queries](/spanner/docs/sql-best-practices#how-execute-queries) to retrieve an execution plan for your query using the Google Cloud console.

If your SQL query isn't supported, simplify it to a query that has a distributed union as the first operator in the query execution plan. Remove aggregate functions, table joins, as well as the operators `  DISTINCT  ` , `  GROUP BY  ` , and `  ORDER  ` , as they are the operators that are most likely to prevent the query from working.

## Create mutations for a write

Use the [`  Mutation  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation) class's [`  newInsertOrUpdateBuilder  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation#com_google_cloud_spanner_Mutation_newInsertOrUpdateBuilder_java_lang_String_) method instead of the [`  newInsertBuilder  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation#com_google_cloud_spanner_Mutation_newInsertBuilder_java_lang_String_) method unless absolutely necessary for Java pipelines. For Python pipelines, use [`  SpannerInsertOrUpdate  `](https://beam.apache.org/releases/pydoc/current/apache_beam.io.gcp.spanner.html#apache_beam.io.gcp.spanner.SpannerInsertOrUpdate) instead of [`  SpannerInsert  `](https://beam.apache.org/releases/pydoc/current/apache_beam.io.gcp.spanner.html#apache_beam.io.gcp.spanner.SpannerInsert) . Dataflow provides at-least-once guarantees, meaning that the mutation might be written several times. As a result, `  INSERT  ` only mutations might generate `  com.google.cloud.spanner.SpannerException: ALREADY_EXISTS  ` errors that cause the pipeline to fail. To prevent this error, use the `  INSERT_OR_UPDATE  ` mutation instead, which adds a new row or updates column values if the row already exists. The `  INSERT_OR_UPDATE  ` mutation can be applied more than once.

## Write to Spanner and transform data

You can write data to Spanner with the Dataflow connector by using a [`  SpannerIO.write  `](https://beam.apache.org/documentation/sdks/javadoc/current/index.html?org/apache/beam/sdk/io/gcp/spanner/SpannerIO.html) transform to execute a collection of input row mutations. The Dataflow connector groups mutations into batches for efficiency.

The following example shows how to apply a write transform to a `  PCollection  ` of mutations:

### GoogleSQL

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

### PostgreSQL

``` java
PCollectionView<Dialect> dialectView =
    pipeline.apply(Create.of(Dialect.POSTGRESQL)).apply(View.asSingleton());
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
        .withDatabaseId(databaseId)
        .withDialectView(dialectView));
```

If a transform unexpectedly stops before completion, mutations that have already been applied aren't rolled back.

**Note:** The `  SpannerIO.write  ` transform doesn't guarantee that all of the mutations in the `  PCollection  ` are applied atomically in a single transaction. If a small set of mutations must be applied atomically, see [Apply groups of mutations atomically](#mutationgroup) .

### Apply groups of mutations atomically

You can use the [`  MutationGroup  `](https://beam.apache.org/documentation/sdks/javadoc/current/index.html?org/apache/beam/sdk/io/gcp/spanner/SpannerIO.html) class to ensure that a group of mutations are applied together atomically. Mutations in a `  MutationGroup  ` are guaranteed to be submitted in the same transaction, but the transaction might be retried.

Mutation groups perform best when they are used to group together mutations that affect data stored close together in the key space. Because Spanner interleaves parent and child table data together in the parent table, that data is always close together in the key space. We recommend that you either structure your mutation group so that it contains one mutation that's applied to a parent table and additional mutations that are applied to child tables, or so that all of its mutations modify data that's close together in the key space. For more information about how Spanner stores parent and child table data, see [Schema and data model](/spanner/docs/schema-and-data-model#parent-child_table_relationships) . If you don't organize your mutation groups around the recommended table hierarchies, or if the data being accessed isn't close together in the key space, Spanner might need to perform two-phase commits, which results in slower performance. For more information, see [Locality tradeoffs](/spanner/docs/whitepapers/optimizing-schema-design#tradeoffs_of_locality) .

To use `  MutationGroup  ` , build a `  SpannerIO.write  ` transform and call the [`  SpannerIO.Write.grouped  `](https://beam.apache.org/documentation/sdks/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.Write.html#grouped--) method, which returns a transform that you can then apply to a `  PCollection  ` of `  MutationGroup  ` objects.

When creating a `  MutationGroup  ` , the first mutation listed becomes the primary mutation. If your mutation group affects both a parent and a child table, the primary mutation should be a mutation to the parent table. Otherwise, you can use any mutation as the primary mutation. The Dataflow connector uses the primary mutation to determine partition boundaries in order to efficiently batch mutations together.

For example, imagine that your application monitors behavior and flags problematic user behavior for review. For each flagged behavior, you want to update the `  Users  ` table to block the user's access to your application, and you also need to record the incident in the `  PendingReviews  ` table. To make sure that both of the tables are updated atomically, use a `  MutationGroup  ` :

### GoogleSQL

``` java
PCollection<MutationGroup> mutations =
    suspiciousUserIds.apply(
        MapElements.via(
            new SimpleFunction<>() {

              @Override
              public MutationGroup apply(String userId) {
                // Immediately block the user.
                Mutation userMutation =
                    Mutation.newUpdateBuilder("Users")
                        .set("id")
                        .to(userId)
                        .set("state")
                        .to("BLOCKED")
                        .build();
                long generatedId =
                    Hashing.sha1()
                        .newHasher()
                        .putString(userId, Charsets.UTF_8)
                        .putLong(timestamp.getSeconds())
                        .putLong(timestamp.getNanos())
                        .hash()
                        .asLong();

                // Add an entry to pending review requests.
                Mutation pendingReview =
                    Mutation.newInsertOrUpdateBuilder("PendingReviews")
                        .set("id")
                        .to(generatedId) // Must be deterministically generated.
                        .set("userId")
                        .to(userId)
                        .set("action")
                        .to("REVIEW ACCOUNT")
                        .set("note")
                        .to("Suspicious activity detected.")
                        .build();

                return MutationGroup.create(userMutation, pendingReview);
              }
            }));

mutations.apply(SpannerIO.write()
    .withInstanceId(instanceId)
    .withDatabaseId(databaseId)
    .grouped());
```

### PostgreSQL

``` java
PCollectionView<Dialect> dialectView =
    pipeline.apply(Create.of(Dialect.POSTGRESQL)).apply(View.asSingleton());
PCollection<MutationGroup> mutations = suspiciousUserIds
    .apply(MapElements.via(new SimpleFunction<String, MutationGroup>() {

      @Override
      public MutationGroup apply(String userId) {
        // Immediately block the user.
        Mutation userMutation = Mutation.newUpdateBuilder("Users")
            .set("id").to(userId)
            .set("state").to("BLOCKED")
            .build();
        long generatedId = Hashing.sha1().newHasher()
            .putString(userId, Charsets.UTF_8)
            .putLong(timestamp.getSeconds())
            .putLong(timestamp.getNanos())
            .hash()
            .asLong();

        // Add an entry to pending review requests.
        Mutation pendingReview = Mutation.newInsertOrUpdateBuilder("PendingReviews")
            .set("id").to(generatedId)  // Must be deterministically generated.
            .set("userId").to(userId)
            .set("action").to("REVIEW ACCOUNT")
            .set("note").to("Suspicious activity detected.")
            .build();

        return MutationGroup.create(userMutation, pendingReview);
      }
    }));

mutations.apply(SpannerIO.write()
    .withInstanceId(instanceId)
    .withDatabaseId(databaseId)
    .withDialectView(dialectView)
    .grouped());
```

When creating a mutation group, the first mutation supplied as an argument becomes the primary mutation. In this case, the two tables are unrelated, so there is no clear primary mutation. We've selected `  userMutation  ` as primary by placing it first. Applying the two mutations separately would be faster, but wouldn't guarantee atomicity, so the mutation group is the best choice in this situation.

## What's next

  - Learn more about [designing an Apache Beam data pipeline](https://beam.apache.org/documentation/pipelines/design-your-pipeline/) .
  - [Export](/spanner/docs/export) and [import](/spanner/docs/import) Spanner databases in the Google Cloud console using Dataflow.
