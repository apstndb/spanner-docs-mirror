This page describes how to write a commit timestamp for each insert and update operation that you perform with Spanner in PostgreSQL-dialect databases.

## Insert commit timestamps

The commit timestamp, based on [TrueTime](/spanner/docs/true-time-external-consistency) technology, is the time when a transaction is committed in the database. You can atomically store the commit timestamp of a transaction into a column. Using the commit timestamps stored in tables, you can determine the exact ordering of mutations and build features like changelogs.

To insert commit timestamps in your database, complete the following steps:

1.  Create a column of type `  SPANNER.COMMIT_TIMESTAMP  ` . For example:
    
    ``` text
    CREATE TABLE Performances (
        ...
        LastUpdateTime SPANNER.COMMIT_TIMESTAMP NOT NULL,
        ...
        PRIMARY KEY (...)
    ) ;
    ```

2.  If you are performing inserts or updates with DML, use the `  SPANNER.PENDING_COMMIT_TIMESTAMP()  ` function to write the commit timestamp.
    
    If you are performing inserts or updates with prepared statements or mutations, use the placeholder string `  SPANNER.COMMIT_TIMESTAMP()  ` for your commit timestamp column. You can also use the commit timestamp constant provided by the client library. For example, this constant in the Java client is `  Value.COMMIT_TIMESTAMP  ` .

When Spanner commits the transaction by using these placeholders as column values, the actual commit timestamp is written to the specified column. You can then use this column value to create a history of updates to the table.

Commit timestamp values are not guaranteed to be unique. Transactions that write to non-overlapping sets of fields might have the same timestamp. Transactions that write to overlapping sets of fields have unique timestamps.

Spanner commit timestamps have microsecond granularity, and they are converted to nanoseconds when stored in `  SPANNER.COMMIT_TIMESTAMP  ` columns.

### Keys and indexes

You can use a commit timestamp column as a primary key column or as a non-key column. Primary keys can be defined as `  ASC  ` or `  DESC  ` .

  - `  ASC  ` (default) - Ascending keys are ideal for answering queries from a specific time forward.
  - `  DESC  ` - Descending keys keep the latest rows at the top of the table. They provide quick access to the latest records.

### Avoid hotspots

Using commit timestamps under the following scenarios creates [hotspots](/spanner/docs/schema-design#primary-key-prevent-hotspots) , which reduce data performance:

  - Commit timestamp column as the first part of the primary key of a table.
    
    ``` text
    CREATE TABLE Users (
      LastAccess SPANNER.COMMIT_TIMESTAMP NOT NULL,
      UserId     bigint NOT NULL,
      ...
      PRIMARY KEY (LastAccess, UserId)
    ) ;
    ```

  - Commit timestamp primary key column as the first part of a secondary index.
    
    ``` text
    CREATE INDEX UsersByLastAccess ON Users(LastAccess)
    ```
    
    or
    
    ``` text
    CREATE INDEX UsersByLastAccessAndName ON Users(LastAccess, FirstName)
    ```

Hotspots reduce data performance, even with low write rates. There is no performance overhead if commit timestamps are enabled on non-key columns that are not indexed.

### Add a commit timestamp column to an existing table

To add a commit timestamp column to an existing table, use the `  ALTER TABLE  ` statement. For example to add a `  LastUpdateTime  ` column to the `  Performances  ` table, use the following statement:

``` text
ALTER TABLE Performances ADD COLUMN LastUpdateTime SPANNER.COMMIT_TIMESTAMP;
```

## Write a commit timestamp using a DML statement

You use the `  SPANNER.PENDING_COMMIT_TIMESTAMP()  ` function to write the commit timestamp in a DML statement. Spanner selects the commit timestamp when the transaction commits.

**Note:** After you call the `  SPANNER.PENDING_COMMIT_TIMESTAMP()  ` function, the table and any derived index is unreadable to any subsequent SQL statements in the transaction. You must write commit timestamps as the last statement in a transaction to prevent the possibility of trying to read the table. If you try to read the table, then Spanner returns an error.

The following DML statement updates the `  LastUpdateTime  ` column in the `  Performances  ` table with the commit timestamp:

``` text
UPDATE Performances SET LastUpdateTime = SPANNER.PENDING_COMMIT_TIMESTAMP()
   WHERE SingerId=1 AND VenueId=2 AND EventDate="2015-10-21"
```

## Insert a row using a mutation

When inserting a row, Spanner writes the commit timestamp value only if you include the column in the column list and pass the `  spanner.commit_timestamp()  ` placeholder string (or client library constant) as its value. For example:

### C++

``` cpp
void InsertDataWithTimestamp(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(spanner::Mutations{
      spanner::InsertOrUpdateMutationBuilder(
          "Performances",
          {"SingerId", "VenueId", "EventDate", "Revenue", "LastUpdateTime"})
          .EmplaceRow(1, 4, absl::CivilDay(2017, 10, 5), 11000,
                      spanner::CommitTimestamp{})
          .EmplaceRow(1, 19, absl::CivilDay(2017, 11, 2), 15000,
                      spanner::CommitTimestamp{})
          .EmplaceRow(2, 42, absl::CivilDay(2017, 12, 23), 7000,
                      spanner::CommitTimestamp{})
          .Build()});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout
      << "Update was successful [spanner_insert_data_with_timestamp_column]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class WriteDataWithTimestampAsyncSample
{
    public class Performance
    {
        public int SingerId { get; set; }
        public int VenueId { get; set; }
        public DateTime EventDate { get; set; }
        public long Revenue { get; set; }
    }

    public async Task<int> WriteDataWithTimestampAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        List<Performance> performances = new List<Performance>
        {
            new Performance { SingerId = 1, VenueId = 4, EventDate = DateTime.Parse("2017-10-05"), Revenue = 11000 },
            new Performance { SingerId = 1, VenueId = 19, EventDate = DateTime.Parse("2017-11-02"), Revenue = 15000 },
            new Performance { SingerId = 2, VenueId = 42, EventDate = DateTime.Parse("2017-12-23"), Revenue = 7000 },
        };
        // Create connection to Cloud Spanner.
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        // Insert rows into the Performances table.
        var rowCountAarray = await Task.WhenAll(performances.Select(performance =>
        {
            var cmd = connection.CreateInsertCommand("Performances", new SpannerParameterCollection
            {
                { "SingerId", SpannerDbType.Int64, performance.SingerId },
                { "VenueId", SpannerDbType.Int64, performance.VenueId },
                { "EventDate", SpannerDbType.Date, performance.EventDate },
                { "Revenue", SpannerDbType.Int64, performance.Revenue },
                { "LastUpdateTime", SpannerDbType.Timestamp, SpannerParameter.CommitTimestamp },
            });
            return cmd.ExecuteNonQueryAsync();
        }));
        return rowCountAarray.Sum();
    }
}
```

### Go

``` go
import (
 "context"

 "cloud.google.com/go/spanner"
)

func writeWithTimestamp(db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 performanceColumns := []string{"SingerId", "VenueId", "EventDate", "Revenue", "LastUpdateTime"}
 m := []*spanner.Mutation{
     spanner.InsertOrUpdate("Performances", performanceColumns, []interface{}{1, 4, "2017-10-05", 11000, spanner.CommitTimestamp}),
     spanner.InsertOrUpdate("Performances", performanceColumns, []interface{}{1, 19, "2017-11-02", 15000, spanner.CommitTimestamp}),
     spanner.InsertOrUpdate("Performances", performanceColumns, []interface{}{2, 42, "2017-12-23", 7000, spanner.CommitTimestamp}),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

``` java
static final List<Performance> PERFORMANCES =
    Arrays.asList(
        new Performance(1, 4, "2017-10-05", 11000),
        new Performance(1, 19, "2017-11-02", 15000),
        new Performance(2, 42, "2017-12-23", 7000));
static void writeExampleDataWithTimestamp(DatabaseClient dbClient) {
  List<Mutation> mutations = new ArrayList<>();
  for (Performance performance : PERFORMANCES) {
    mutations.add(
        Mutation.newInsertBuilder("Performances")
            .set("SingerId")
            .to(performance.singerId)
            .set("VenueId")
            .to(performance.venueId)
            .set("EventDate")
            .to(performance.eventDate)
            .set("Revenue")
            .to(performance.revenue)
            .set("LastUpdateTime")
            .to(Value.COMMIT_TIMESTAMP)
            .build());
  }
  dbClient.write(mutations);
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

// Instantiate Spanner table objects
const performancesTable = database.table('Performances');

const data = [
  {
    SingerId: '1',
    VenueId: '4',
    EventDate: '2017-10-05',
    Revenue: '11000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    SingerId: '1',
    VenueId: '19',
    EventDate: '2017-11-02',
    Revenue: '15000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    SingerId: '2',
    VenueId: '42',
    EventDate: '2017-12-23',
    Revenue: '7000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
];

// Inserts rows into the Singers table
// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so
// they must be converted to strings before being inserted as INT64s
try {
  await performancesTable.insert(data);
  console.log('Inserted data.');
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
 * Inserts sample data into a table with a commit timestamp column.
 *
 * The database and table must already exist and can be created using
 * `create_table_with_timestamp_column`.
 * Example:
 * ```
 * insert_data_with_timestamp_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function insert_data_with_timestamp_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->insertBatch('Performances', [
            ['SingerId' => 1, 'VenueId' => 4, 'EventDate' => '2017-10-05', 'Revenue' => 11000, 'LastUpdateTime' => $spanner->commitTimestamp()],
            ['SingerId' => 1, 'VenueId' => 19, 'EventDate' => '2017-11-02', 'Revenue' => 15000, 'LastUpdateTime' => $spanner->commitTimestamp()],
            ['SingerId' => 2, 'VenueId' => 42, 'EventDate' => '2017-12-23', 'Revenue' => 7000, 'LastUpdateTime' => $spanner->commitTimestamp()],
        ])
        ->commit();

    print('Inserted data.' . PHP_EOL);
}
````

### Python

``` python
def insert_data_with_timestamp(instance_id, database_id):
    """Inserts data with a COMMIT_TIMESTAMP field into a table."""

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.batch() as batch:
        batch.insert(
            table="Performances",
            columns=("SingerId", "VenueId", "EventDate", "Revenue", "LastUpdateTime"),
            values=[
                (1, 4, "2017-10-05", 11000, spanner.COMMIT_TIMESTAMP),
                (1, 19, "2017-11-02", 15000, spanner.COMMIT_TIMESTAMP),
                (2, 42, "2017-12-23", 7000, spanner.COMMIT_TIMESTAMP),
            ],
        )

    print("Inserted data.")
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

# Get commit_timestamp
commit_timestamp = client.commit_timestamp

client.commit do |c|
  c.insert "Performances", [
    { SingerId: 1, VenueId: 4, EventDate: "2017-10-05", Revenue: 11_000, LastUpdateTime: commit_timestamp },
    { SingerId: 1, VenueId: 19, EventDate: "2017-11-02", Revenue: 15_000, LastUpdateTime: commit_timestamp },
    { SingerId: 2, VenueId: 42, EventDate: "2017-12-23", Revenue: 7000, LastUpdateTime: commit_timestamp }
  ]
end

puts "Inserted data"
```

If you have mutations on rows in multiple tables, you must specify `  spanner.commit_timestamp()  ` (or client library constant) for the commit timestamp column in each table.

## Update a row using a mutation

When updating a row, Spanner writes the commit timestamp value only if you include the column in the column list and pass the `  spanner.commit_timestamp()  ` placeholder string (or client library constant) as its value. You cannot update the primary key of a row. To update the primary key, delete the existing row and create a new row.

For example, to update a commit timestamp column named `  LastUpdateTime  ` :

### C++

``` cpp
void UpdateDataWithTimestamp(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(spanner::Mutations{
      spanner::UpdateMutationBuilder(
          "Albums",
          {"SingerId", "AlbumId", "MarketingBudget", "LastUpdateTime"})
          .EmplaceRow(1, 1, 1000000, spanner::CommitTimestamp{})
          .EmplaceRow(2, 2, 750000, spanner::CommitTimestamp{})
          .Build()});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout
      << "Update was successful [spanner_update_data_with_timestamp_column]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateDataWithTimestampColumnAsyncSample
{
    public async Task<int> UpdateDataWithTimestampColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);

        var rowCount = 0;
        using var updateCmd1 = connection.CreateUpdateCommand("Albums", new SpannerParameterCollection
        {
            { "SingerId", SpannerDbType.Int64, 1 },
            { "AlbumId", SpannerDbType.Int64, 1 },
            { "MarketingBudget", SpannerDbType.Int64, 1000000 },
            { "LastUpdateTime", SpannerDbType.Timestamp, SpannerParameter.CommitTimestamp },
        });
        rowCount += await updateCmd1.ExecuteNonQueryAsync();

        using var updateCmd2 = connection.CreateUpdateCommand("Albums", new SpannerParameterCollection
        {
            { "SingerId", SpannerDbType.Int64, 2 },
            { "AlbumId", SpannerDbType.Int64, 2 },
            { "MarketingBudget", SpannerDbType.Int64, 750000 },
            { "LastUpdateTime", SpannerDbType.Timestamp, SpannerParameter.CommitTimestamp },
        });
        rowCount += await updateCmd2.ExecuteNonQueryAsync();

        Console.WriteLine("Updated data.");
        return rowCount;
    }
}
```

### Go

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func updateWithTimestamp(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 cols := []string{"SingerId", "AlbumId", "MarketingBudget", "LastUpdateTime"}
 _, err = client.Apply(ctx, []*spanner.Mutation{
     spanner.Update("Albums", cols, []interface{}{1, 1, 1000000, spanner.CommitTimestamp}),
     spanner.Update("Albums", cols, []interface{}{2, 2, 750000, spanner.CommitTimestamp}),
 })
 return err
}
```

### Java

``` java
static void updateWithTimestamp(DatabaseClient dbClient) {
  // Mutation can be used to update/insert/delete a single row in a table. Here we use
  // newUpdateBuilder to create update mutations.
  List<Mutation> mutations =
      Arrays.asList(
          Mutation.newUpdateBuilder("Albums")
              .set("SingerId")
              .to(1)
              .set("AlbumId")
              .to(1)
              .set("MarketingBudget")
              .to(1000000)
              .set("LastUpdateTime")
              .to(Value.COMMIT_TIMESTAMP)
              .build(),
          Mutation.newUpdateBuilder("Albums")
              .set("SingerId")
              .to(2)
              .set("AlbumId")
              .to(2)
              .set("MarketingBudget")
              .to(750000)
              .set("LastUpdateTime")
              .to(Value.COMMIT_TIMESTAMP)
              .build());
  // This writes all the mutations to Cloud Spanner atomically.
  dbClient.write(mutations);
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

// Update a row in the Albums table
// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
// must be converted to strings before being inserted as INT64s
const albumsTable = database.table('Albums');

const data = [
  {
    SingerId: '1',
    AlbumId: '1',
    MarketingBudget: '1000000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    SingerId: '2',
    AlbumId: '2',
    MarketingBudget: '750000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
];

try {
  await albumsTable.update(data);
  console.log('Updated data.');
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
 * Updates sample data in a table with a commit timestamp column.
 *
 * Before executing this method, a new column MarketingBudget has to be added to the Albums
 * table by applying the DDL statement "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64".
 *
 * In addition this update expects the LastUpdateTime column added by applying the DDL statement
 * "ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP OPTIONS (allow_commit_timestamp=true)"
 *
 * Example:
 * ```
 * update_data_with_timestamp_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_timestamp_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->updateBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'MarketingBudget' => 1000000, 'LastUpdateTime' => $spanner->commitTimestamp()],
            ['SingerId' => 2, 'AlbumId' => 2, 'MarketingBudget' => 750000, 'LastUpdateTime' => $spanner->commitTimestamp()],
        ])
        ->commit();

    print('Updated data.' . PHP_EOL);
}
````

### Python

``` python
def update_data_with_timestamp(instance_id, database_id):
    """Updates Performances tables in the database with the COMMIT_TIMESTAMP
    column.

    This updates the `MarketingBudget` column which must be created before
    running this sample. You can add the column by running the `add_column`
    sample or by running this DDL statement against your database:

        ALTER TABLE Albums ADD COLUMN MarketingBudget INT64

    In addition this update expects the LastUpdateTime column added by
    applying this DDL statement against your database:

        ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP
        OPTIONS(allow_commit_timestamp=true)
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.batch() as batch:
        batch.update(
            table="Albums",
            columns=("SingerId", "AlbumId", "MarketingBudget", "LastUpdateTime"),
            values=[
                (1, 1, 1000000, spanner.COMMIT_TIMESTAMP),
                (2, 2, 750000, spanner.COMMIT_TIMESTAMP),
            ],
        )

    print("Updated data.")
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

In your code, you can provide your own value for the commit timestamp column instead of passing `  spanner.commit_timestamp()  ` (or the available client library constant) as the column value. The value must be a timestamp in the past. This restriction ensures that writing timestamps is an inexpensive and fast operation. One way to confirm that a value is in the past is to compare it to the value returned by the `  CURRENT_TIMESTAMP  ` SQL function. The server returns a `  FailedPrecondition  ` error if a future timestamp is specified.

## Create a changelog

Suppose that you want to create a changelog of every mutation that happens to a table and then use that changelog for auditing. An example would be a table that stores the history of changes to word processing documents. The commit timestamp makes creating the changelog easier, because the timestamps can enforce ordering of the changelog entries. You could build a changelog that stores the history of changes to a given document using a schema like the following example:

``` text
CREATE TABLE Documents (
  UserId     int8 NOT NULL,
  DocumentId int8 NOT NULL,
  Contents   text NOT NULL,
  PRIMARY KEY (UserId, DocumentId)
);

CREATE TABLE DocumentHistory (
  UserId     int8 NOT NULL,
  DocumentId int8 NOT NULL,
  Ts         SPANNER.COMMIT_TIMESTAMP NOT NULL,
  Delta      text,
  PRIMARY KEY (UserId, DocumentId, Ts)
) INTERLEAVE IN PARENT Documents;
```

To create a changelog, insert a new row in `  DocumentHistory  ` in the same transaction in which you insert or update a row in `  Document  ` . In the insertion of the new row in `  DocumentHistory  ` , use the placeholder `  spanner.commit_timestamp()  ` (or client library constant) to tell Spanner to write the commit timestamp into column `  Ts  ` . Interleaving the `  DocumentsHistory  ` table with the `  Documents  ` table permits data locality and more efficient inserts and updates. However, it also adds the constraint that the parent and child rows must be deleted together. To keep the rows in `  DocumentHistory  ` after rows in `  Documents  ` are deleted, don't interleave the tables.

## Optimize recent-data queries with commit timestamps

Commit timestamps optimize your Spanner database and can reduce query I/O when retrieving data written after a particular time.

To activate this optimization, a query's `  WHERE  ` clause must include a comparison between a table's commit timestamp column and a specific time that you provide, with the following attributes:

  - Provide the specific time as a *constant expression* : a literal, a parameter, or a function whose own arguments evaluate to constants.

  - Compare whether the commit timestamp is more recent than the given time, through either the `  >  ` or `  >=  ` operators.

  - Optionally, add further restrictions to the `  WHERE  ` clause with `  AND  ` . Extending the clause with `  OR  ` disqualifies the query from this optimization.

For example, consider the following `  Performances  ` table, which includes a commit timestamp column:

``` text
CREATE TABLE Performances (
  SingerId bigint NOT NULL,
  VenueId bigint NOT NULL,
  EventDate timestamp with time zone NOT NULL,
  Revenue bigint,
  LastUpdateTime spanner.commit_timestamp,
  PRIMARY KEY(SingerId, VenueId, EventDate)
);
```

This query benefits from the commit-timestamp optimization described earlier, because it has a greater-than-or-equal-to comparison between the table's commit timestamp column and a constant expression—in this case, a literal:

``` text
SELECT * FROM Performances WHERE LastUpdateTime >= '2022-01-01';
```
