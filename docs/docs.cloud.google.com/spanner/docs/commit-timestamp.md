This document describes how to write a commit timestamp for each insert and update operation that you perform with Spanner. To use this feature, set the `  allow_commit_timestamp  ` option on a `  TIMESTAMP  ` column, then write the timestamp as part of each transaction.

## Overview

The commit timestamp, based on [TrueTime](/spanner/docs/true-time-external-consistency) technology, is the time when a transaction is committed in the database. The `  allow_commit_timestamp  ` column option lets you atomically store the commit timestamp into a column. Using the commit timestamps stored in tables, you can determine the exact ordering of mutations and build features like changelogs.

To insert commit timestamps in your database, complete the following steps:

1.  [Create a column with type `  TIMESTAMP  `](#create-column) with the column option `  allow_commit_timestamp  ` set to `  true  ` in the schema definition. For example:
    
    ``` text
    CREATE TABLE Performances (
        ...
        LastUpdateTime  TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
        ...
    ) PRIMARY KEY (...);
    ```

2.  If you are performing inserts or updates with DML, [use the `  PENDING_COMMIT_TIMESTAMP  ` function](#dml) to write the commit timestamp.
    
    If you are performing inserts or updates with mutations, [use the placeholder string `  spanner.commit_timestamp()  `](#create-column) on insertions or updates to your commit timestamp column. You can also use the commit timestamp constant provided by the client library. For example, this constant in the Java client is `  Value.COMMIT_TIMESTAMP  ` .

When Spanner commits the transaction using these placeholders as column values, the actual commit timestamp is written to the specified column (For example: the `  LastUpdateTime  ` column). You could then use this column value to create a history of updates to the table.

Commit timestamp values are not guaranteed to be unique. Transactions that write to non-overlapping sets of fields might have the same timestamp. Transactions that write to overlapping sets of fields have unique timestamps.

Spanner commit timestamps have microsecond granularity, and they are converted to nanoseconds when stored in `  TIMESTAMP  ` columns.

## Create and delete a commit timestamp column

Use the `  allow_commit_timestamp  ` column option to add and remove support for commit timestamps:

  - When [creating a new table](/spanner/docs/reference/standard-sql/data-definition-language#table_statements) to specify that a column supports commit timestamps.
  - When [altering an existing table](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) :
      - to add a new column supporting commit timestamps,
      - to alter an existing `  TIMESTAMP  ` column to support commit timestamps,
      - to alter an existing `  TIMESTAMP  ` column to remove commit timestamp support

### Keys and indexes

You can use a commit timestamp column as a primary key column or as a non-key column. Primary keys can be defined as `  ASC  ` or `  DESC  ` .

  - `  ASC  ` (default) - Ascending keys are ideal for answering queries from a specific time forward.
  - `  DESC  ` - Descending keys keep the latest rows at the top of the table. They provide quick access to the latest records.

The `  allow_commit_timestamp  ` option must be consistent across the primary keys of parent and child tables. If the option is not consistent across primary keys, Spanner returns an error. The only time the option can be inconsistent is when you are creating or updating the schema.

Using commit timestamps under the following scenarios creates [hotspots](/spanner/docs/schema-design#primary-key-prevent-hotspots) which reduce data performance:

  - Commit timestamp column as the first part of the primary key of a table:
    
    ``` text
    CREATE TABLE Users (
      LastAccess TIMESTAMP NOT NULL,
      UserId     INT64 NOT NULL,
      ...
    ) PRIMARY KEY (LastAccess, UserId);
    ```

  - The first part of the primary key of a secondary index:
    
    ``` text
    CREATE INDEX UsersByLastAccess ON Users(LastAccess)
    ```
    
    or
    
    ``` text
    CREATE INDEX UsersByLastAccessAndName ON Users(LastAccess, FirstName)
    ```

[Hotspots](/spanner/docs/schema-design#primary-key-prevent-hotspots) reduce data performance, even with low write rates. There is no performance overhead if commit timestamps are enabled on non-key columns that are not indexed.

### Create a commit timestamp column

The following DDL creates a table with a column that supports commit timestamps.

``` text
CREATE TABLE Performances (
    SingerId        INT64 NOT NULL,
    VenueId         INT64 NOT NULL,
    EventDate       Date,
    Revenue         INT64,
    LastUpdateTime  TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
) PRIMARY KEY (SingerId, VenueId, EventDate),
  INTERLEAVE IN PARENT Singers ON DELETE CASCADE
```

Adding the option changes the timestamp column as follows:

  - You can use the `  spanner.commit_timestamp()  ` placeholder string (or a constant provided by the client library) for inserts and updates.
  - The column can only contain values in the past. For more information, see [Providing your own value for the timestamp](#provide-timestamp) .

The option `  allow_commit_timestamp  ` is case sensitive.

### Add a commit timestamp column to an existing table

To add a commit timestamp column to an existing table, use the `  ALTER TABLE  ` statement. For example to add a `  LastUpdateTime  ` column to the `  Performances  ` table, use the following statement:

``` text
ALTER TABLE Performances ADD COLUMN LastUpdateTime TIMESTAMP
    NOT NULL OPTIONS (allow_commit_timestamp=true)
```

### Convert a timestamp column to a commit timestamp column

You can convert an existing timestamp column into a commit timestamp column, but doing so requires Spanner to validate that the existing timestamp values are in the past. For example:

``` text
ALTER TABLE Performances ALTER COLUMN LastUpdateTime
    SET OPTIONS (allow_commit_timestamp=true)
```

You cannot change the data type or `  NULL  ` annotation of a column in an `  ALTER TABLE  ` statement that includes `  SET OPTIONS  ` . For details, see [Data Definition Language](/spanner/docs/reference/standard-sql/data-definition-language) .

### Remove the commit timestamp option

If you want to remove commit timestamp support from a column, use the option `  allow_commit_timestamp=null  ` in an `  ALTER TABLE  ` statement. The commit timestamp behavior is removed, but the column is still a timestamp. Changing the option does not alter any other characteristics of the column, such as type or nullability ( `  NOT NULL  ` ). For example:

``` text
ALTER TABLE Performances ALTER COLUMN LastUpdateTime
    SET OPTIONS (allow_commit_timestamp=null)
```

## Write a commit timestamp using a DML statement

You use the `  PENDING_COMMIT_TIMESTAMP  ` function to write the commit timestamp in a DML statement. Spanner selects the commit timestamp when the transaction commits.

**Note:** After you call the `  PENDING_COMMIT_TIMESTAMP  ` function, the table and any derived index is unreadable to any future SQL statements in the transaction. Because of this, the change stream can't extract the previous value for the column that has a pending commit timestamp, if the coloumn is modified again later in the same transaction. You must write commit timestamps as the last statement in a transaction to prevent the possibility of trying to read the table. If you try to read the table, then Spanner produces an error.

The following DML statement updates the `  LastUpdateTime  ` column in the `  Performances  ` table with the commit timestamp:

``` text
UPDATE Performances SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP()
   WHERE SingerId=1 AND VenueId=2 AND EventDate="2015-10-21"
```

The following code example uses the [`  PENDING_COMMIT_TIMESTAMP  `](/spanner/docs/reference/standard-sql/timestamp_functions) function to write the commit timestamp in the `  LastUpdateTime  ` column.

### C++

``` cpp
void DmlStandardUpdateWithTimestamp(google::cloud::spanner::Client client) {
  using ::google::cloud::StatusOr;
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(
      [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto update = client.ExecuteDml(
            std::move(txn),
            spanner::SqlStatement(
                "UPDATE Albums SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP()"
                "  WHERE SingerId = 1"));
        if (!update) return std::move(update).status();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Update was successful "
            << "[spanner_dml_standard_update_with_timestamp]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateUsingDmlWithTimestampCoreAsyncSample
{
    public async Task<int> UpdateUsingDmlWithTimestampCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("UPDATE Albums SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP() WHERE SingerId = 1");
        int rowCount = await cmd.ExecuteNonQueryAsync();

        Console.WriteLine($"{rowCount} row(s) updated...");
        return rowCount;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func updateUsingDMLWithTimestamp(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `UPDATE Albums
             SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP()
             WHERE SingerId = 1`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) updated.\n", rowCount)
     return nil
 })
 return err
}
```

### Java

``` java
static void updateUsingDmlWithTimestamp(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(transaction -> {
        String sql =
            "UPDATE Albums "
                + "SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP() WHERE SingerId = 1";
        long rowCount = transaction.executeUpdate(Statement.of(sql));
        System.out.printf("%d records updated.\n", rowCount);
        return null;
      });
}
```

### Node.js

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  try {
    const [rowCount] = await transaction.runUpdate({
      sql: `UPDATE Albums
        SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP()
        WHERE SingerId = 1`,
    });

    console.log(`Successfully updated ${rowCount} records.`);
    await transaction.commit();
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
});
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;

/**
 * Update data with a DML statement using timestamps.
 *
 * The database and table must already exist and can be created using
 * `create_database`.
 * Example:
 * ```
 * insert_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_dml_timestamp(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) {
        $rowCount = $t->executeUpdate(
            'UPDATE Albums '
            . 'SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP() WHERE SingerId = 1');
        $t->commit();
        printf('Updated %d row(s).' . PHP_EOL, $rowCount);
    });
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

def update_albums(transaction):
    row_ct = transaction.execute_update(
        "UPDATE Albums "
        "SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP() "
        "WHERE SingerId = 1"
    )

    print("{} record(s) updated.".format(row_ct))

database.run_in_transaction(update_albums)
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id
row_count = 0

client.transaction do |transaction|
  row_count = transaction.execute_update(
    "UPDATE Albums SET LastUpdateTime = PENDING_COMMIT_TIMESTAMP() WHERE SingerId = 1"
  )
end

puts "#{row_count} records updated."
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

commit_timestamp = client.commit_timestamp

client.commit do |c|
  c.update "Albums", [
    { SingerId: 1, AlbumId: 1, MarketingBudget: 100_000, LastUpdateTime: commit_timestamp },
    { SingerId: 2, AlbumId: 2, MarketingBudget: 750_000, LastUpdateTime: commit_timestamp }
  ]
end

puts "Updated data"
```

Commit timestamps can only be written to columns annotated with the `  allow_commit_timestamp=true  ` option.

If you have mutations on rows in multiple tables, you must specify `  spanner.commit_timestamp()  ` (or the client library constant) for the commit timestamp column in each table.

## Query a commit timestamp column

The following example queries the commit timestamp column of the table.

### C++

``` cpp
void QueryDataWithTimestamp(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  spanner::SqlStatement select(
      "SELECT SingerId, AlbumId, MarketingBudget, LastUpdateTime"
      "  FROM Albums"
      " ORDER BY LastUpdateTime DESC");
  using RowType =
      std::tuple<std::int64_t, std::int64_t, absl::optional<std::int64_t>,
                 absl::optional<spanner::Timestamp>>;

  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << std::get<0>(*row) << " " << std::get<1>(*row);
    auto marketing_budget = std::get<2>(*row);
    if (!marketing_budget) {
      std::cout << " NULL";
    } else {
      std::cout << ' ' << *marketing_budget;
    }
    auto last_update_time = std::get<3>(*row);
    if (!last_update_time) {
      std::cout << " NULL";
    } else {
      std::cout << ' ' << *last_update_time;
    }
    std::cout << "\n";
  }
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithTimestampColumnAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public DateTime? LastUpdateTime { get; set; }
        public long? MarketingBudget { get; set; }
    }

    public async Task<List<Album>> QueryDataWithTimestampColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, MarketingBudget, LastUpdateTime FROM Albums ORDER BY LastUpdateTime DESC");

        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                LastUpdateTime = reader.IsDBNull(reader.GetOrdinal("LastUpdateTime")) ? (DateTime?)null : reader.GetFieldValue<DateTime>("LastUpdateTime"),
                MarketingBudget = reader.IsDBNull(reader.GetOrdinal("MarketingBudget")) ? 0 : reader.GetFieldValue<long>("MarketingBudget")
            });
        }
        return albums;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "strconv"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func queryWithTimestamp(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{
     SQL: `SELECT SingerId, AlbumId, MarketingBudget, LastUpdateTime
             FROM Albums ORDER BY LastUpdateTime DESC`}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var marketingBudget spanner.NullInt64
     var lastUpdateTime spanner.NullTime
     if err := row.ColumnByName("SingerId", &singerID); err != nil {
         return err
     }
     if err := row.ColumnByName("AlbumId", &albumID); err != nil {
         return err
     }
     if err := row.ColumnByName("MarketingBudget", &marketingBudget); err != nil {
         return err
     }
     budget := "NULL"
     if marketingBudget.Valid {
         budget = strconv.FormatInt(marketingBudget.Int64, 10)
     }
     if err := row.ColumnByName("LastUpdateTime", &lastUpdateTime); err != nil {
         return err
     }
     timestamp := "NULL"
     if lastUpdateTime.Valid {
         timestamp = lastUpdateTime.String()
     }
     fmt.Fprintf(w, "%d %d %s %s\n", singerID, albumID, budget, timestamp)
 }
}
```

### Java

``` java
static void queryMarketingBudgetWithTimestamp(DatabaseClient dbClient) {
  // Rows without an explicit value for MarketingBudget will have a MarketingBudget equal to
  // null. A try-with-resource block is used to automatically release resources held by
  // ResultSet.
  try (ResultSet resultSet =
      dbClient
          .singleUse()
          .executeQuery(
              Statement.of(
                  "SELECT SingerId, AlbumId, MarketingBudget, LastUpdateTime FROM Albums"
                      + " ORDER BY LastUpdateTime DESC"))) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s %s\n",
          resultSet.getLong("SingerId"),
          resultSet.getLong("AlbumId"),
          // We check that the value is non null. ResultSet getters can only be used to retrieve
          // non null values.
          resultSet.isNull("MarketingBudget") ? "NULL" : resultSet.getLong("MarketingBudget"),
          resultSet.isNull("LastUpdateTime") ? "NULL" : resultSet.getTimestamp("LastUpdateTime"));
    }
  }
}
```

### Node.js

``` javascript
// ...

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const query = {
  sql: `SELECT SingerId, AlbumId, MarketingBudget, LastUpdateTime
          FROM Albums ORDER BY LastUpdateTime DESC`,
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();

    console.log(
      `SingerId: ${json.SingerId}, AlbumId: ${
        json.AlbumId
      }, MarketingBudget: ${
        json.MarketingBudget ? json.MarketingBudget : null
      }, LastUpdateTime: ${json.LastUpdateTime}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished
  database.close();
}
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from a database with a commit timestamp column.
 *
 * This sample uses the `MarketingBudget` column. You can add the column
 * by running the `add_column` sample or by running this DDL statement against
 * your database:
 *
 *      ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * This sample also uses the 'LastUpdateTime' commit timestamp column. You can
 * add the column by running the `add_timestamp_column` sample or by running
 * this DDL statement against your database:
 *
 *       ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP OPTIONS (allow_commit_timestamp=true)
 *
 * Example:
 * ```
 * query_data_with_timestamp_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_timestamp_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT SingerId, AlbumId, MarketingBudget, LastUpdateTime ' .
        ' FROM Albums ORDER BY LastUpdateTime DESC'
    );

    foreach ($results as $row) {
        if ($row['MarketingBudget'] == null) {
            $row['MarketingBudget'] = 'NULL';
        }
        if ($row['LastUpdateTime'] == null) {
            $row['LastUpdateTime'] = 'NULL';
        }
        printf('SingerId: %s, AlbumId: %s, MarketingBudget: %s, LastUpdateTime: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['MarketingBudget'], $row['LastUpdateTime']);
    }
}
````

### Python

``` python
def query_data_with_timestamp(instance_id, database_id):
    """Queries sample data from the database using SQL.

    This updates the `LastUpdateTime` column which must be created before
    running this sample. You can add the column by running the
    `add_timestamp_column` sample or by running this DDL statement
    against your database:

        ALTER TABLE Performances ADD COLUMN LastUpdateTime TIMESTAMP
        OPTIONS (allow_commit_timestamp=true)

    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId, AlbumId, MarketingBudget FROM Albums "
            "ORDER BY LastUpdateTime DESC"
        )

    for row in results:
        print("SingerId: {}, AlbumId: {}, MarketingBudget: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.execute("SELECT SingerId, AlbumId, MarketingBudget, LastUpdateTime
                FROM Albums ORDER BY LastUpdateTime DESC").rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:MarketingBudget]} #{row[:LastUpdateTime]}"
end
```

## Provide your own value for the commit timestamp column

You can provide your own value for the commit timestamp column, instead of passing `  spanner.commit_timestamp()  ` (or client library constant) as the column value. The value must be a timestamp in the past. This restriction ensures that writing timestamps is an inexpensive and fast operation. The server returns a `  FailedPrecondition  ` error if a future timestamp is specified.

**Note:** The `  CURRENT_TIMESTAMP  ` value is not based on true time. So, the value it returns is not necessarily in the past and cannot be used to compare with a commit timestamp value.

## Create a changelog

Suppose that you want to create a changelog of every mutation that happens to a table and then use that changelog for auditing. An example would be a table that stores the history of changes to word processing documents. The commit timestamp makes creating the changelog easier, because the timestamps can enforce ordering of the changelog entries. You could build a changelog that stores the history of changes to a given document using a schema like the following example:

``` text
CREATE TABLE Documents (
  UserId     INT64 NOT NULL,
  DocumentId INT64 NOT NULL,
  Contents   STRING(MAX) NOT NULL,
) PRIMARY KEY (UserId, DocumentId);

CREATE TABLE DocumentHistory (
  UserId     INT64 NOT NULL,
  DocumentId INT64 NOT NULL,
  Ts         TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true),
  Delta      STRING(MAX),
) PRIMARY KEY (UserId, DocumentId, Ts),
  INTERLEAVE IN PARENT Documents ON DELETE NO ACTION;
```

To create a changelog, insert a new row in `  DocumentHistory  ` in the same transaction in which you insert or update a row in `  Document  ` . In the insertion of the new row in `  DocumentHistory  ` , use the placeholder `  spanner.commit_timestamp()  ` (or client library constant) to tell Spanner to write the commit timestamp into column `  Ts  ` . Interleaving the `  DocumentsHistory  ` table with the `  Documents  ` table will allow for data locality and more efficient inserts and updates. However, it also adds the constraint that the parent and child rows must be deleted together. To keep the rows in `  DocumentHistory  ` after rows in `  Documents  ` are deleted, do not interleave the tables.

## Optimize recent-data queries with commit timestamps

Commit timestamps enable a Spanner optimization that can reduce query I/O when retrieving data written after a particular time.

To activate this optimization, a query's `  WHERE  ` clause must include a comparison between a table's commit timestamp column and a specific time that you provide, with the following attributes:

  - Provide the specific time as a *constant expression* : a literal, a parameter, or a function whose own arguments evaluate to constants.

  - Compare whether the commit timestamp is more recent than the given time, through either the `  >  ` or `  >=  ` operators.

  - Optionally, add further restrictions to the `  WHERE  ` clause with `  AND  ` . Extending the clause with `  OR  ` disqualifies the query from this optimization.

For example, consider the following `  Performances  ` table, which includes a commit timestamp column:

``` text
CREATE TABLE Performances (
    SingerId INT64 NOT NULL,
    VenueId INT64 NOT NULL,
    EventDate DATE,
    Revenue INT64,
    LastUpdateTime TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
) PRIMARY KEY (SingerId, VenueId, EventDate);
```

This query benefits from the commit-timestamp optimization described earlier, because it has a greater-than-or-equal-to comparison between the table's commit timestamp column and a constant expression—in this case, a literal:

``` text
SELECT * FROM Performances WHERE LastUpdateTime >= "2022-05-01";
```

The following query also qualifies for the optimization, since it has a greater-than comparison between the commit timestamp and a function whose arguments all evaluate to constants during the query's execution:

``` text
SELECT * FROM Performances
  WHERE LastUpdateTime > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY);
```

## What's next

  - Use commit timestamps to [create a change log with Go](https://cloud.google.com/community/tutorials/cloud-spanner-commit-timestamp-change-log) .
