This page describes how to create, modify, and view Spanner change streams for GoogleSQL-dialect databases and PostgreSQL-dialect databases. To learn more about change streams, see [About change streams](/spanner/docs/change-streams) .

Because change streams are schema objects, you create and manage them through the same DDL-driven [schema updates](/spanner/docs/schema-updates) used for any other kind of database-definition work, such as creating tables or adding indexes.

Spanner begins a [long-running operation](/spanner/docs/manage-long-running-operations) after you submit a schema-changing DDL statement, including those used to create, alter, or delete change streams. A new or altered change stream begins to watch the columns or tables specified by its new configuration upon the completion of this long-running operation.

## Create a change stream

To create a change stream, you need to provide its name and the schema objects it watches: either the entire database, or a list of specific tables and columns. You can optionally configure a change stream with any of the following:

  - [Specify the data retention period](/spanner/docs/change-streams#data-retention) to override the default, one-day retention period.

  - [Specify the value capture type](/spanner/docs/change-streams#value-capture-type) to override the default value capture type `  OLD_AND_NEW_VALUES  ` .

  - [Apply a TTL-based deletes filter](/spanner/docs/change-streams#ttl-filter) to filter out TTL-based deletes from your change streams.

  - [Apply a table modifications filter](/spanner/docs/change-streams#mod-type-filter) to exclude all `  INSERT  ` , `  UPDATE  ` , or `  DELETE  ` table modifications.

  - [Enable a transaction-level records exclusion](/spanner/docs/change-streams#transaction-exclusion) to exclude certain transactions from your change streams.

### GoogleSQL

The DDL syntax for creating a change stream using GoogleSQL looks like this:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME
  [FOR column_or_table_watching_definition[, ... ] ]
  [
    OPTIONS (
      retention_period = timespan,
      value_capture_type = type,
      exclude_ttl_deletes = boolean,
      exclude_insert = boolean,
      exclude_update = boolean,
      exclude_delete = boolean,
      allow_txn_exclusion = boolean
    )
  ]
```

### PostgreSQL

The DDL syntax for creating a change stream using PostgreSQL looks like this:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME
  [FOR column_or_table_watching_definition[, ... ] ]
  [
    WITH (
      retention_period = timespan,
      value_capture_type = type,
      exclude_ttl_deletes = boolean,
      exclude_insert = boolean,
      exclude_update = boolean,
      exclude_delete = boolean,
      allow_txn_exclusion = boolean
    )
  ]
```

A new change stream begins to watch its assigned schema objects as soon as the long-running operation that created it completes.

The following examples illustrate the creation of change streams with various configurations.

### Watch an entire database

To create a change stream that watches every data change performed throughout a database's tables, use the `  ALL  ` keyword:

``` text
CREATE CHANGE STREAM EverythingStream
FOR ALL;
```

The `  ALL  ` configuration [implicitly includes](/spanner/docs/change-streams#implicit-watching) all of the database's future data tables and columns, as soon as they get created. It [doesn't include](/spanner/docs/change-streams#what-gets-captured) views, information schema tables, or other objects aside from normal data tables.

### Watch specific tables

To limit a change stream's scope to specific tables, rather than an entire database, specify a list of one or more tables:

``` text
CREATE CHANGE STREAM SingerAlbumStream
FOR Singers, Albums;
```

Spanner [automatically updates](/spanner/docs/change-streams#implicit-watching) change streams that watch whole tables to reflect any schema changes affecting those tables, such as added or dropped columns.

### Watch specific columns

Use the `  table ( column_1 [, column_2 , ...])  ` syntax to watch changes to one or more specific, non-key columns within tables that you name:

``` text
CREATE CHANGE STREAM NamesAndTitles
FOR Singers(FirstName, LastName), Albums(Title);
```

You can't specify primary key columns here because every change stream *always* tracks the primary keys of every table it watches. This allows every data change record to identify the changed row by its primary key.

### Watch tables and columns in a single stream

You can combine table-watching and column-watching syntax from the previous two examples in a single change stream:

``` text
CREATE CHANGE STREAM NamesAndAlbums
FOR Singers(FirstName, LastName), Albums;
```

### Specify a longer retention period

To specify a [change stream data retention period](/spanner/docs/change-streams#data-retention) longer than the default of one day, set the `  retention_period  ` to a period of up to thirty days, expressed as either hours ( `  h  ` ) or days ( `  d  ` ).

Two examples:

### GoogleSQL

``` text
CREATE CHANGE STREAM LongerDataRetention
FOR ALL
OPTIONS ( retention_period = '36h' );
```

``` text
CREATE CHANGE STREAM MaximumDataRetention
FOR ALL
OPTIONS ( retention_period = '7d' );
```

### PostgreSQL

``` text
CREATE CHANGE STREAM LongerDataRetention
FOR ALL
WITH ( retention_period = '36h' );
```

``` text
CREATE CHANGE STREAM MaximumDataRetention
FOR ALL
WITH ( retention_period = '7d' );
```

### Specify a different value capture type

To specify a [change stream value capture type](/spanner/docs/change-streams#value-capture-type) other than `  OLD_AND_NEW_VALUES  ` , set the `  value_capture_type  ` to either `  NEW_VALUES  ` or `  NEW_ROW  ` , as shown in the following examples:

### GoogleSQL

``` text
CREATE CHANGE STREAM NewRowChangeStream
FOR ALL
OPTIONS ( value_capture_type = 'NEW_ROW' );
```

``` text
CREATE CHANGE STREAM NewValuesChangeStream
FOR ALL
OPTIONS ( value_capture_type = 'NEW_VALUES' );
```

### PostgreSQL

``` text
CREATE CHANGE STREAM NewRowChangeStream
FOR ALL
WITH ( value_capture_type = 'NEW_ROW' );
```

``` text
CREATE CHANGE STREAM NewValuesChangeStream
FOR ALL
WITH ( value_capture_type = 'NEW_VALUES' );
```

### Filter TTL-based deletes

You can filter [TTL-based deletes](/spanner/docs/ttl) from your change stream's scope using the `  exclude_ttl_deletes  ` filter.

For more information on how this filter work, see [Time to live based deletes filter](/spanner/docs/change-streams#ttl-filter) .

### GoogleSQL

To create a change stream with the TTL-based deletes filter, run the following example:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
OPTIONS (exclude_ttl_deletes = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your new change stream

The following example creates a change stream called `  NewFilterChangeStream  ` that excludes all TTL-based deletes:

``` text
CREATE CHANGE STREAM NewFilterChangeStream FOR ALL
OPTIONS (exclude_ttl_deletes = true)
```

### PostgreSQL

To create a change stream with the TTL-based deletes filter, run the following example:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
WITH (exclude_ttl_deletes = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your new change stream

The following example creates a change stream called `  NewFilterChangeStream  ` that excludes all TTL-based deletes:

``` text
CREATE CHANGE STREAM NewFilterChangeStream FOR ALL
WITH (exclude_ttl_deletes = true)
```

To add or remove the TTL-based deletes filter from an existing change stream, see [Modify TTL-based deletes filter](#modify-ttl-deletes) . You can confirm your change stream filters by [viewing definitions of the change stream as DDL](/spanner/docs/change-streams/manage#view-ddl) .

### Filter by table modification type

Filter one or more of these table modifications from your change stream's scope using the following available filter options:

  - `  exclude_insert  ` : exclude all `  INSERT  ` table modifications
  - `  exclude_update  ` : exclude all `  UPDATE  ` table modifications
  - `  exclude_delete  ` : exclude all `  DELETE  ` table modifications

For more information on how these filters work, see [Table modification type filters](/spanner/docs/change-streams#mod-type-filter) .

### GoogleSQL

To create a change stream with one or more table modification type filters, run the following:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
OPTIONS (MOD_TYPE_FILTER_NAME = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your new change stream
  - MOD\_TYPE\_FILTER\_NAME : the filter you want to add: `  exclude_insert  ` , `  exclude_update  ` , or `  exclude_delete  ` . If adding more than one filter at a time, separate each with a comma.

The following example creates a change stream called `  NewFilterChangeStream  ` that excludes the `  INSERT  ` and `  UPDATE  ` table modification types:

``` text
CREATE CHANGE STREAM NewFilterChangeStream FOR ALL
OPTIONS (exclude_insert = true, exclude_update = true)
```

### PostgreSQL

To create a change stream with one or more table modification type filters, run the following:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
WITH (MOD_TYPE_FILTER_NAME = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your new change stream
  - MOD\_TYPE\_FILTER\_NAME : the filter you want to add: `  exclude_insert  ` , `  exclude_update  ` , or `  exclude_delete  ` . If adding more than one filter at a time, separate each with a comma.

The following example creates a change stream called `  NewFilterChangeStream  ` that excludes the `  INSERT  ` and `  UPDATE  ` table modification types:

``` text
CREATE CHANGE STREAM NewFilterChangeStream FOR ALL
WITH (exclude_insert = true, exclude_update = true)
```

To add or remove a table modification type filter from an existing change stream, see [Modify filter by table modification type](#modify-mod-type) . You can confirm which table modification type filters exist for your change stream by [viewing definitions of the change stream as DDL](/spanner/docs/change-streams/manage#view-ddl) .

### Enable exclusion of transaction-level records

You can enable your change streams to exclude records from specified write transactions by setting the `  allow_txn_exclusion  ` option when you create a change stream, or by [modifying an existing change stream](#modify-allow-transaction-exclusion) .

For more information on how this option works, see [Transaction-level records exclusion](/spanner/docs/change-streams#transaction-exclusion) .

### GoogleSQL

To create a change stream that can exclude records from specified write transactions, run the following:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
OPTIONS (allow_txn_exclusion = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of the new change stream

The following example creates a change stream named `  NewChangeStream  ` that can exclude records from specified write transactions:

``` text
CREATE CHANGE STREAM NewChangeStream FOR ALL
OPTIONS (allow_txn_exclusion = true)
```

### PostgreSQL

To create a change stream that can exclude records from specified write transactions, run the following:

``` text
CREATE CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
WITH (allow_txn_exclusion = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of the new change stream

The following example creates a change stream named `  NewChangeStream  ` that can exclude records from specified write transactions,:

``` text
CREATE CHANGE STREAM NewChangeStream FOR ALL
WITH (allow_txn_exclusion = true)
```

To enable or disable the transaction-level records exclusion from an existing change stream, see [Modify transaction-level records exclusion](#modify-allow-transaction-exclusion) . To check the setting of this option, see [view definitions of change streams as DDL](/spanner/docs/change-streams/manage#view-ddl) .

#### Specify a write transaction to be excluded from change streams

To specify a write transaction to be excluded from change streams, you must set the `  exclude_txn_from_change_streams  ` parameter to `  true  ` . The following code samples shows how you can specify a write transaction to be excluded from change streams using the client library.

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

// readWriteTxnExcludedFromChangeStreams executes the insert and update DMLs on
// Singers table excluded from tracking change streams with ddl option
// allow_txn_exclusion = true.
func readWriteTxnExcludedFromChangeStreams(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return fmt.Errorf("readWriteTxnExcludedFromChangeStreams.NewClient: %w", err)
 }
 defer client.Close()

 _, err = client.ReadWriteTransactionWithOptions(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT Singers (SingerId, FirstName, LastName)
                 VALUES (111, 'Virginia', 'Watson')`,
     }
     _, err := txn.Update(ctx, stmt)
     if err != nil {
         return fmt.Errorf("readWriteTxnExcludedFromChangeStreams.Update: %w", err)
     }
     fmt.Fprintln(w, "New singer inserted.")
     stmt = spanner.Statement{
         SQL: `UPDATE Singers SET FirstName = 'Hi' WHERE SingerId = 111`,
     }
     _, err = txn.Update(ctx, stmt)
     if err != nil {
         return fmt.Errorf("readWriteTxnExcludedFromChangeStreams.Update: %w", err)
     }
     fmt.Fprint(w, "Singer first name updated.")
     return nil
 }, spanner.TransactionOptions{ExcludeTxnFromChangeStreams: true})
 if err != nil {
     return err
 }
 return nil
}
```

### Java

``` java
static void readWriteTxnExcludedFromChangeStreams(DatabaseClient client) {
  // Exclude the transaction from allowed tracking change streams with alloww_txn_exclusion=true.
  // This exclusion will be applied to all the individual operations inside this transaction.
  client
      .readWriteTransaction(Options.excludeTxnFromChangeStreams())
      .run(
          transaction -> {
            transaction.executeUpdate(
                Statement.of(
                    "INSERT Singers (SingerId, FirstName, LastName)\n"
                        + "VALUES (1341, 'Virginia', 'Watson')"));
            System.out.println("New singer inserted.");

            transaction.executeUpdate(
                Statement.of("UPDATE Singers SET FirstName = 'Hi' WHERE SingerId = 111"));
            System.out.println("Singer first name updated.");

            return null;
          });
}
```

## Modify a change stream

To modify a change stream's configuration, use an `  ALTER CHANGE STREAM  ` DDL statement. It uses syntax similar to `  CREATE CHANGE STREAM  ` . You can change which columns a stream watches, or the length of its data retention period. You can also suspend its watching altogether while retaining its data change records.

### Modify what a change stream watches

This example adds the entire `  Songs  ` table to the `  NamesAndAlbums  ` change stream configured earlier:

``` text
ALTER CHANGE STREAM NamesAndAlbums
SET FOR Singers(FirstName, LastName), Albums, Songs;
```

Spanner replaces the named change stream's behavior with the new configuration upon the completion of the long-running operation that updates the change stream's definition within the database's schema.

### Modify a change stream's data retention period

To modify the length of time that a change stream retains its internal records, set the `  retention_period  ` in an `  ALTER CHANGE STREAM  ` DDL statement.

This example adjusts the data retention period to the `  NamesAndAlbums  ` change stream created previously:

### GoogleSQL

``` text
ALTER CHANGE STREAM NamesAndAlbums
SET OPTIONS ( retention_period = '36h' );
```

### PostgreSQL

``` text
ALTER CHANGE STREAM NamesAndAlbums
SET ( retention_period = '36h' );
```

### Modify a change stream's value capture type

To modify the value capture type of a change stream, set the `  value_capture_type  ` clause in an `  ALTER CHANGE STREAM  ` DDL statement.

This example adjusts the value capture type to `  NEW_VALUES  ` .

### GoogleSQL

``` text
ALTER CHANGE STREAM NamesAndAlbums
SET OPTIONS ( value_capture_type = 'NEW_VALUES' );
```

### PostgreSQL

``` text
ALTER CHANGE STREAM NamesAndAlbums
SET ( value_capture_type = 'NEW_VALUES' );
```

**Note:** Modifying the value capture type of a change stream doesn't cause columns to backfill. For example, modifying the value capture type from `  NEW_VALUES  ` to `  OLD_AND_NEW_VALUES  ` doesn't cause backfill of old values for a change stream.

### Modify TTL-based deletes filter

To modify the [TTL-based deletes](/spanner/docs/ttl) filter for a change stream, set the `  exclude_ttl_deletes  ` filter in an `  ALTER CHANGE STREAM  ` DDL statement. You can use this to add the filter to or remove the filter from your existing change streams.

For more information on how these filters work, see [Time to live based deletes filter](/spanner/docs/change-streams#ttl-filter) .

#### Add the TTL-based deletes filter to an existing change stream

### GoogleSQL

To add the TTL-based deletes filter to an existing change stream, run the following to set the filter to `  true  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET OPTIONS (exclude_ttl_deletes = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your existing change stream

In the following example, the `  exclude_ttl_deletes  ` filter is added to an existing change stream called `  NewFilterChangeStream  ` that excludes all TTL-based deletes:

``` text
ALTER CHANGE STREAM NewFilterChangeStream FOR ALL
SET OPTIONS (exclude_ttl_deletes = true)
```

This excludes all future TTL-based deletes from the change stream.

### PostgreSQL

To add the TTL-based deletes filter to an existing change stream, run the following to set the filter to `  true  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET (exclude_ttl_deletes = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your existing change stream

In the following example, the `  exclude_ttl_deletes  ` filter is added to an existing change stream called `  NewFilterChangeStream  ` that excludes all TTL-based deletes:

``` text
ALTER CHANGE STREAM NewFilterChangeStream FOR ALL
SET (exclude_ttl_deletes = true)
```

This excludes all future TTL-based deletes from the change stream.

#### Remove the TTL-based deletes filter from an existing change stream

### GoogleSQL

To remove the TTL-based deletes filter to an existing change stream, run the following to set the filter to `  false  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET OPTIONS (exclude_ttl_deletes = false)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your new change stream

In the following example, the `  exclude_ttl_deletes  ` filter is removed from an existing change stream called `  NewFilterChangeStream  ` :

``` text
ALTER CHANGE STREAM NewFilterChangeStream FOR ALL
SET OPTIONS (exclude_ttl_deletes = false)
```

This includes all future TTL-based deletes to the change stream.

You can also set the filter to `  null  ` to remove the TTL-based deletes filter.

### PostgreSQL

To remove the TTL-based deletes filter to an existing change stream, run the following to set the filter to `  false  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET (exclude_ttl_deletes = false)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your new change stream

In the following example, the `  exclude_ttl_deletes  ` filter is removed from an existing change stream called `  NewFilterChangeStream  ` :

``` text
ALTER CHANGE STREAM NewFilterChangeStream FOR ALL
SET (exclude_ttl_deletes = false)
```

This includes all future TTL-based deletes to the change stream.

You can also set the filter to `  null  ` to remove the TTL-based deletes filter.

### Modify filter by table modification type

To modify the table modification type filters for a change stream, set the filter type in an `  ALTER CHANGE STREAM  ` DDL statement. You can use this to add a new filter to or remove an existing filter from your change stream.

#### Add a table modification type filter to an existing change stream

### GoogleSQL

To add one or more new table modification type filters to an existing change stream, run the following to set the filter to `  true  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME
SET OPTIONS (MOD_TYPE_FILTER_NAME = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : replace with the name of your existing change stream
  - MOD\_TYPE\_FILTER\_NAME : replace with the filter you want to add: `  exclude_insert  ` , `  exclude_update  ` , or `  exclude_delete  ` . If adding more than one filter at a time, separate each filter with a comma.

In the following example, the `  exclude_delete  ` filter is added to an existing change stream called `  NewFilterChangeStream  ` :

``` text
ALTER CHANGE STREAM NewFilterChangeStream
SET OPTIONS (exclude_delete = true)
```

### PostgreSQL

To add one or more new table modification type filters to an existing change stream, run the following to set the filter to `  true  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME
SET (MOD_TYPE_FILTER_NAME = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : replace with the name of your existing change stream
  - MOD\_TYPE\_FILTER\_NAME : replace with the filter you want to add: `  exclude_insert  ` , `  exclude_update  ` , or `  exclude_delete  ` . If adding more than one filter at a time, separate each filter with a comma.

In the following example, the `  exclude_delete  ` filter is added to an existing change stream called `  NewFilterChangeStream  ` :

``` text
ALTER CHANGE STREAM NewFilterChangeStream
SET (exclude_delete = true)
```

#### Remove a table modification type filter from an existing change stream

### GoogleSQL

To remove one or more existing table modification type filters in change stream, run the following to set the filter to `  false  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME
SET OPTIONS (MOD_TYPE_FILTER_NAME = false)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : replace with the name of your existing change stream
  - MOD\_TYPE\_FILTER\_NAME : replace with the filter you want to remove: `  exclude_insert  ` , `  exclude_update  ` , or `  exclude_delete  ` . If removing more than one filter at a time, separate each with a comma.

In the following example, the `  exclude_delete  ` filter is removed from an existing change stream called `  NewFilterChangeStream  ` :

``` text
ALTER CHANGE STREAM NewFilterChangeStream
SET OPTIONS (exclude_delete = false)
```

You can also remove a table modification filter by setting the filter back to default value. To do this, set the filter value to `  null  ` .

### PostgreSQL

To remove one or more existing table modification type filters in change stream, run the following to set the filter to `  false  ` :

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME
SET (MOD_TYPE_FILTER_NAME = false)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : replace with the name of your existing change stream
  - MOD\_TYPE\_FILTER\_NAME : replace with the filter you want to remove: `  exclude_insert  ` , `  exclude_update  ` , or `  exclude_delete  ` . If removing more than one filter at a time, separate each with a comma.

In the following example, the `  exclude_delete  ` filter is removed from an existing change stream called `  NewFilterChangeStream  ` :

``` text
ALTER CHANGE STREAM NewFilterChangeStream
SET (exclude_delete = false)
```

You can also remove a table modification filter by setting the filter back to default value. To do this, set the filter value to `  null  ` .

### Modify change stream to allow exclusion of transaction-level records

You can modify your change stream to allow it to exclude records from specified write transactions. To do this, set the `  allow_txn_exclusion  ` option to `  true  ` in an `  ALTER CHANGE STREAM  ` DDL statement. If you don't set this option, or if you set it to `  false  ` , then the change stream watches all write transactions.

For more information about how this option works, see [Transaction-level records exclusion](/spanner/docs/change-streams#transaction-exclusion) .

#### Enable transaction-level records exclusion for an existing change stream

### GoogleSQL

To enable transaction-level records exclusion for an existing change stream, run the following:

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET OPTIONS (allow_txn_exclusion = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your existing change stream

In the following example, the `  allow_txn_exclusion  ` option is enabled on an existing change stream, `  NewAllowedChangeStream  ` :

``` text
ALTER CHANGE STREAM NewAllowedChangeStream FOR ALL
SET OPTIONS (allow_txn_exclusion = true)
```

This allows the change stream to exclude records from specified write transactions.

### PostgreSQL

To enable transaction-level records exclusion for an existing change stream, run the following:

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET (allow_txn_exclusion = true)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your existing change stream

In the following example, the `  allow_txn_exclusion  ` option is enabled on an existing change stream, `  NewAllowedChangeStream  ` :

``` text
ALTER CHANGE STREAM NewAllowedChangeStream FOR ALL
SET (allow_txn_exclusion = true)
```

This allows the change stream to exclude records from specified write transactions.

#### Disable transaction-level records exclusion for an existing change stream

### GoogleSQL

To disable the transaction-level records exclusion on an existing change stream, run the following:

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET OPTIONS (allow_txn_exclusion = false)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your change stream

In the following example, the `  allow_txn_exclusion  ` option is disabled on an existing change stream called `  NewAllowedChangeStream  ` :

``` text
ALTER CHANGE STREAM NewFilterChangeStream FOR ALL
SET OPTIONS (allow_txn_exclusion = false)
```

The change stream watches all write transactions.

### PostgreSQL

To disable the transaction-level records exclusion on an existing change stream, run the following:

``` text
ALTER CHANGE STREAM CHANGE_STREAM_NAME FOR ALL
SET (allow_txn_exclusion = false)
```

Replace the following:

  - CHANGE\_STREAM\_NAME : the name of your change stream

In the following example, the `  allow_txn_exclusion  ` option is disabled on an existing change stream called `  NewAllowedChangeStream  ` :

``` text
ALTER CHANGE STREAM NewAllowedChangeStream FOR ALL
SET (allow_txn_exclusion = false)
```

The change stream watches all write transactions.

### Suspend a change stream

If you want to have a change stream halt its activity but retain its internal records—at least for the life of its data retention period—you can alter it so that it watches nothing.

To do this, issue an `  ALTER CHANGE STREAM  ` DDL statement that replaces the change stream's definition with the special phrase `  DROP FOR ALL  ` . For example:

``` text
ALTER CHANGE STREAM MyStream DROP FOR ALL;
```

The stream continues to exist in the database, but watches no objects, and generates no further data change records. Its existing change records remain intact, subject to the stream's data retention policy.

To resume a suspended stream, issue another `  ALTER CHANGE STREAM  ` statement with its previous configuration.

**Note:** You can create a change stream in a suspended state by omitting the `  FOR  ` clause entirely from the DDL: `  CREATE CHANGE STREAM MyStream;  ` .

## Delete a change stream

To permanently delete a change stream, issue a `  DROP CHANGE STREAM  ` statement that includes the stream's name:

``` text
DROP CHANGE STREAM NamesAndAlbums;
```

Spanner immediately stops the stream, removes it from the database's schema, and deletes its data change records.

## List and view change streams

The Google Cloud console provides a web interface for listing and reviewing a database's change stream definitions. You can also view the structure of change streams as their equivalent DDL statements, or by querying the information schema for the database.

### View change streams with Google Cloud console

To see a list of a database's change streams and review their definitions:

1.  Visit the Spanner instances page of the Google Cloud console.

2.  Navigate to the appropriate instance and database.

3.  Click **Change streams** in the navigational menu.

This displays a list of all of that database's change streams, and summarizes the configuration of each one. Clicking a stream's name reveals further details about the tables and columns that it watches.

### View definitions of change streams as DDL

Viewing a database's schema as DDL includes descriptions of all its change streams, where they appear as `  CREATE CHANGE STREAM  ` statements.

  - To do this from the console, click The **Show equivalent DDL** link on the database's page in the Google Cloud console.

  - To do this from the command line, use [the `  ddl describe  ` command of Google Cloud CLI](/sdk/gcloud/reference/spanner/databases/ddl/describe) .

### Query the information schema about change streams

You can directly query a database's [information schema](/spanner/docs/information-schema) about its change streams. The following tables contain the metadata that defines change streams' names, the tables and columns that they watch, and their retention periods:

  - [`  information_schema.change_streams  `](/spanner/docs/information-schema#information_schemachange_streams)

  - [`  information_schema.change_stream_tables  `](/spanner/docs/information-schema#information_schemachange_stream_tables)

  - [`  information_schema.change_stream_columns  `](/spanner/docs/information-schema#information_schemachange_stream_columns)

  - [`  information_schema.change_stream_options  `](/spanner/docs/information-schema#information_schemachange_stream_options)

## Change streams best practices

The following are some best practices for configuring and managing change streams.

### Consider a separate metadata database

Change streams use a metadata database to maintain internal state. The metadata database can be the same as or different from the database that contains the change streams. We recommend creating a separate database for metadata storage.

The Spanner change streams connector needs read-write permissions to the metadata database. You don't need to prepare this database with a schema; the connector takes care of that.

Using a separate metadata database eliminates complexities that might arise from allowing the connector to write directly to your application database:

  - By separating the metadata database from the production database with the change stream, the connector needs only read permissions to the production database.

  - By restricting the traffic of the connector to a separate metadata database, writes performed by the connector itself are not included in the production change streams. This is especially relevant for change streams that watch the entire database.

If no separate database is being used to store the metadata, we recommend monitoring the CPU impact of the change streams connector on their instances.

### Benchmark new change streams and resize if needed

Before adding new change streams to your production instance, consider benchmarking a realistic workload on a staging instance with change streams enabled. This lets you determine whether you need to add nodes to your instance, in order to increase its compute and storage capacities.

Run these tests until CPU and storage metrics stabilize. Optimally, the instance's CPU utilization should remain under the [recommended maximums](/spanner/docs/cpu-utilization#recommended-max) , and its storage usage shouldn't exceed the instance's storage limit.

### Use different regions to load-balance

When using change streams in a [multi-region instance configuration](/spanner/docs/instance-configurations#multi-region-configurations) , consider running their processing pipelines in a different region than the default leader region. This helps to spread the streaming load among non-leader replicas. If you need to prioritize the lowest possible streaming delay over load balancing, however, run the streaming load in the leader region.

## What's next

  - [Build change streams pipelines with Dataflow](/spanner/docs/change-streams/use-dataflow) .

  - Use [templates](/dataflow/docs/guides/templates/provided-streaming) to rapidly connect a change stream to BigQuery, Cloud Storage, or Pub/Sub.
