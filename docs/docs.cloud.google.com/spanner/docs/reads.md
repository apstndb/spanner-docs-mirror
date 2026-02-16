This page describes how to perform reads in Spanner outside the context of read-only and read-write transactions. If either of the following applies, you should read the [Transactions](/spanner/docs/transactions) page instead:

  - **If you need to write, depending on the value of one or more reads** , you should execute the read as part of a read-write transaction. For more information, see [read-write transactions](/spanner/docs/transactions#read-write_transactions) .

  - **If you are making multiple read calls that require a consistent view of your data** , you should execute the reads as part of a read-only transaction. For more information, see [read-only transactions](/spanner/docs/transactions#read-only_transactions) .

## Read types

Spanner lets you determine how current the data should be when you read data by offering two types of reads:

  - A *strong read* is a read at a current timestamp and is guaranteed to see all data that has been committed up until the start of this read. Spanner defaults to using strong reads to serve read requests.
  - A *stale read* is read at a timestamp in the past. If your application is latency sensitive but tolerant of stale data, then stale reads can provide performance benefits.

To choose which type of read you want, set a [timestamp bound](/spanner/docs/timestamp-bounds) on the read request. Use the following best practices when choosing a timestamp bound:

  - **Choose strong reads whenever possible** . These are the default timestamp bound for Spanner reads, including read-only transactions. Strong reads are guaranteed to observe the effects of all transactions that committed before the start of the operation, independent of which replica receives the read. Because of this, strong reads make application code simpler and applications more trustworthy. Read more about Spanner's consistency properties in [TrueTime and External Consistency](/spanner/docs/true-time-external-consistency) .
    
    Optionally, to reduce read latency for read-only transactions that require strong consistency, you can grant non-leader, read-write, or read-only regions the [read lease region](/spanner/docs/read-lease) status. To do so, [use Spanner read leases](/spanner/docs/read-lease#use-read-leases) . Read lease regions help your database reduce [strong read](/spanner/docs/reads#read_types) latency in dual-region or multi-region instances. However, writes experience higher latency when you use read lease.

  - **If latency makes strong reads infeasible in some situations, then use stale reads** (bounded-staleness or exact-staleness) to improve performance in places where you don't need reads to be as recent as possible. As described on the [Replication](/spanner/docs/replication#read-only) page, 15 seconds is a reasonable staleness value to use for good performance.

## Read data with a database role

If you are a [fine-grained access control](/spanner/docs/fgac-about) user, you must select a database role to execute SQL statements and queries, and to perform row operations on a database. Your role selection persists throughout your session until you change the role.

For instructions on how to perform a read with a database role, see [Access a database with fine-grained access control](/spanner/docs/access-with-fgac) .

## Single read methods

Spanner supports single read methods (that is, a read outside the context of a transaction) on a database for:

  - Executing the read as a SQL query statement or using Spanner's read API.
  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

If you want to route single reads to a specific replica or region within a multi-region instance configuration or a custom regional configuration with optional read-only region(s), see [Directed reads](/spanner/docs/directed-reads) .

The following sections describe how to use read methods using Spanner client libraries.

### Execute a query

The following shows how to execute a SQL query statement against a database.

### GoogleSQL

### C++

Use `  ExecuteQuery()  ` to execute a SQL query statement against a database.

``` cpp
void QueryData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  spanner::SqlStatement select("SELECT SingerId, LastName FROM Singers");
  using RowType = std::tuple<std::int64_t, std::string>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\t";
    std::cout << "LastName: " << std::get<1>(*row) << "\n";
  }

  std::cout << "Query completed for [spanner_query_data]\n";
}
```

### C\#

Use `  ExecuteReaderAsync()  ` to query the database.

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QuerySampleDataAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> QuerySampleDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
            });
        }
        return albums;
    }
}
```

### Go

Use `  Client.Single().Query  ` to query the database.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func query(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
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
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

### Java

Use `  ReadContext.executeQuery  ` to query the database.

``` java
static void query(DatabaseClient dbClient) {
  try (ResultSet resultSet =
      dbClient
          .singleUse() // Execute a single read or query against Cloud Spanner.
          .executeQuery(Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"))) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s\n", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
    }
  }
}
```

### Node.js

Use `  Database.run  ` to query the database.

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

const query = {
  sql: 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums',
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  await database.close();
}
```

### PHP

Use `  Database::execute  ` to query the database.

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL.
 * Example:
 * ```
 * query_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
    );

    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

### Python

Use `  Database.execute_sql  ` to query the database.

``` python
def query_data(instance_id, database_id):
    """Queries sample data from the database using SQL."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
        )

        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

### Ruby

Use `  Client#execute  ` to query the database.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.execute("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:AlbumTitle]}"
end
```

Consult the SQL [Query Syntax](/spanner/docs/reference/standard-sql/query-syntax) and [Functions and Operators](/spanner/docs/reference/standard-sql/functions-and-operators) references when constructing a SQL statement.

### Perform a strong read

The following shows how to perform a strong read of zero or more rows from a database.

### GoogleSQL

### C++

The code to read data is the same as the previous sample for querying Spanner by [executing a SQL query](#execute_a_query) .

``` cpp
void QueryData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  spanner::SqlStatement select("SELECT SingerId, LastName FROM Singers");
  using RowType = std::tuple<std::int64_t, std::string>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\t";
    std::cout << "LastName: " << std::get<1>(*row) << "\n";
  }

  std::cout << "Query completed for [spanner_query_data]\n";
}
```

### C\#

The code to read data is the same as the previous sample for querying Spanner by [executing a SQL query](#execute_a_query) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QuerySampleDataAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> QuerySampleDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
            });
        }
        return albums;
    }
}
```

### Go

Use `  Client.Single().Read  ` to read rows from the database.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func read(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 iter := client.Single().Read(ctx, "Albums", spanner.AllKeys(),
     []string{"SingerId", "AlbumId", "AlbumTitle"})
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
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

The example uses `  AllKeys  ` to define a collection of keys or key ranges to read.

### Java

Use `  ReadContext.read  ` to read rows from the database.

``` java
static void read(DatabaseClient dbClient) {
  try (ResultSet resultSet =
      dbClient
          .singleUse()
          .read(
              "Albums",
              KeySet.all(), // Read all rows in a table.
              Arrays.asList("SingerId", "AlbumId", "AlbumTitle"))) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s\n", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
    }
  }
}
```

The example uses `  KeySet  ` to define a collection of keys or key ranges to read.

### Node.js

Use `  Table.read  ` to read rows from the database.

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

// Reads rows from the Albums table
const albumsTable = database.table('Albums');

const query = {
  columns: ['SingerId', 'AlbumId', 'AlbumTitle'],
  keySet: {
    all: true,
  },
};

try {
  const [rows] = await albumsTable.read(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  await database.close();
}
```

The example uses `  keySet  ` to define a collection of keys or key ranges to read.

### PHP

Use `  Database::read  ` to read rows from the database.

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads sample data from the database.
 * Example:
 * ```
 * read_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['SingerId', 'AlbumId', 'AlbumTitle']
    );

    foreach ($results->rows() as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

The example uses `  keySet  ` to define a collection of keys or key ranges to read.

### Python

Use `  Database.read  ` to read rows from the database.

``` python
def read_data(instance_id, database_id):
    """Reads sample data from the database."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums", columns=("SingerId", "AlbumId", "AlbumTitle"), keyset=keyset
        )

        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

The example uses `  KeySet  ` to define a collection of keys or key ranges to read.

### Ruby

Use `  Client#read  ` to read rows from the database.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.read("Albums", [:SingerId, :AlbumId, :AlbumTitle]).rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:AlbumTitle]}"
end
```

### Perform a stale read

The following sample code shows how to perform a stale read of zero or more rows from a database using an **exact-staleness** timestamp bound. For instructions on how to perform a stale read using a **bounded-staleness** timestamp bound, see the note after the sample code. See [Timestamp bounds](/spanner/docs/timestamp-bounds) for more information on the different types of timestamp bounds that are available.

### GoogleSQL

### C++

Use `  ExecuteQuery()  ` with `  MakeReadOnlyTransaction()  ` and `  Transaction::ReadOnlyOptions()  ` to perform a stale read.

``` cpp
void ReadStaleData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  // The timestamp chosen using the `exact_staleness` parameter is bounded
  // below by the creation time of the database, so the visible state may only
  // include that generated by the `extra_statements` executed atomically with
  // the creation of the database. Here we at least know `Albums` exists.
  auto opts = spanner::Transaction::ReadOnlyOptions(std::chrono::seconds(15));
  auto read_only = spanner::MakeReadOnlyTransaction(std::move(opts));

  spanner::SqlStatement select(
      "SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
  using RowType = std::tuple<std::int64_t, std::int64_t, std::string>;

  auto rows = client.ExecuteQuery(std::move(read_only), std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " AlbumId: " << std::get<1>(*row)
              << " AlbumTitle: " << std::get<2>(*row) << "\n";
  }
}
```

### C\#

Use the `  BeginReadOnlyTransactionAsync  ` method on a `  connection  ` with a specified `  TimestampBound.OfExactStaleness()  ` value to query the database.

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class ReadStaleDataAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public long? MarketingBudget { get; set; }
    }

    public async Task<List<Album>> ReadStaleDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        var staleness = TimestampBound.OfExactStaleness(TimeSpan.FromSeconds(15));
        using var transaction = await connection.BeginTransactionAsync(
            SpannerTransactionCreationOptions.ForTimestampBoundReadOnly(staleness),
            transactionOptions: null,
            cancellationToken: default);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, MarketingBudget FROM Albums");
        cmd.Transaction = transaction;

        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                MarketingBudget = reader.IsDBNull(reader.GetOrdinal("MarketingBudget")) ? 0 : reader.GetFieldValue<long>("MarketingBudget")
            });
        }
        return albums;
    }
}
```

**Note:** To use a *bounded-staleness* timestamp bound, specify a `  TimestampBound.OfMaxStaleness()  ` value instead of a `  TimestampBound.OfExactStaleness()  ` value.

### Go

Use `  Client.ReadOnlyTransaction().WithTimestampBound()  ` and specify an `  ExactStaleness  ` value to perform a read of rows from the database using an exact-staleness timestamp bound.

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readStaleData(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 ro := client.ReadOnlyTransaction().WithTimestampBound(spanner.ExactStaleness(15 * time.Second))
 defer ro.Close()

 iter := ro.Read(ctx, "Albums", spanner.AllKeys(), []string{"SingerId", "AlbumId", "AlbumTitle"})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID int64
     var albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

The example uses `  AllKeys  ` to define a collection of keys or key ranges to read.

**Note:** To use a *bounded-staleness* timestamp bound, specify a `  MaxStaleness  ` value instead of an `  ExactStaleness  ` value.

### Java

Use the `  read  ` method of a `  ReadContext  ` that has a specified `  TimestampBound.ofExactStaleness()  ` to perform a read of rows from the database using an exact-staleness timestamp bound.

``` java
static void readStaleData(DatabaseClient dbClient) {
  try (ResultSet resultSet =
      dbClient
          .singleUse(TimestampBound.ofExactStaleness(15, TimeUnit.SECONDS))
          .read(
              "Albums", KeySet.all(), Arrays.asList("SingerId", "AlbumId", "MarketingBudget"))) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s\n",
          resultSet.getLong(0),
          resultSet.getLong(1),
          resultSet.isNull(2) ? "NULL" : resultSet.getLong("MarketingBudget"));
    }
  }
}
```

The example uses `  KeySet  ` to define a collection of keys or key ranges to read.

**Note:** To use a *bounded-staleness* timestamp bound, specify a `  TimestampBound.ofMaxStaleness  ` instead of a `  TimestampBound.ofExactStaleness()  ` .

### Node.js

Use `  Table.read  ` with the `  exactStaleness  ` option to perform a read of rows from the database using an exact-staleness timestamp bound.

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

// Reads rows from the Albums table
const albumsTable = database.table('Albums');

const query = {
  columns: ['SingerId', 'AlbumId', 'AlbumTitle', 'MarketingBudget'],
  keySet: {
    all: true,
  },
};

const options = {
  // Guarantees that all writes committed more than 15000 milliseconds ago are visible
  exactStaleness: 15000,
};

try {
  const [rows] = await albumsTable.read(query, options);

  rows.forEach(row => {
    const json = row.toJSON();
    const id = json.SingerId;
    const album = json.AlbumId;
    const title = json.AlbumTitle;
    const budget = json.MarketingBudget ? json.MarketingBudget : '';
    console.log(
      `SingerId: ${id}, AlbumId: ${album}, AlbumTitle: ${title}, MarketingBudget: ${budget}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  await database.close();
}
```

The example uses `  keySet  ` to define a collection of keys or key ranges to read.

**Note:** To use a *bounded-staleness* timestamp bound, use the `  maxStaleness  ` option instead of the `  exactStaleness  ` option.

### PHP

Use `  Database::read  ` with a `  exactStaleness  ` value specified to perform a read of rows from the database using an exact-staleness timestamp bound.

```` php
use Google\Protobuf\Duration;
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads sample data from the database.  The data is exactly 15 seconds stale.
 * Guarantees that all writes committed more than 15 seconds ago are visible.
 * Example:
 * ```
 * read_stale_data
 *($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_stale_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);
    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['SingerId', 'AlbumId', 'AlbumTitle'],
        ['exactStaleness' => new Duration(['seconds' => 15])]
    );

    foreach ($results->rows() as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

The example uses `  keySet  ` to define a collection of keys or key ranges to read.

**Note:** To use a *bounded-staleness* timestamp bound, specify a `  maxStaleness  ` value instead of an `  exactStaleness  ` value.

### Python

Use the `  read  ` method of a `  Database  ` `  snapshot  ` that has a specified `  exact_staleness  ` value to perform a read of rows from the database using an exact-staleness timestamp bound.

``` python
def read_stale_data(instance_id, database_id):
    """Reads sample data from the database. The data is exactly 15 seconds
    stale."""
    import datetime

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)
    staleness = datetime.timedelta(seconds=15)

    with database.snapshot(exact_staleness=staleness) as snapshot:
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums",
            columns=("SingerId", "AlbumId", "MarketingBudget"),
            keyset=keyset,
        )

        for row in results:
            print("SingerId: {}, AlbumId: {}, MarketingBudget: {}".format(*row))
```

The example uses `  KeySet  ` to define a collection of keys or key ranges to read.

**Note:** To use a *bounded-staleness* timestamp bound, specify a `  max_staleness  ` value instead of the `  exact_staleness  ` value.

### Ruby

Use the `  read  ` method of a snapshot `  Client  ` that has a specified `  staleness  ` value (in seconds) to perform a read of rows from the database using an exact-staleness timestamp bound.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"
require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

# Perform a read with a data staleness of 15 seconds
client.snapshot staleness: 15 do |snapshot|
  snapshot.read("Albums", [:SingerId, :AlbumId, :AlbumTitle]).rows.each do |row|
    puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:AlbumTitle]}"
  end
end
```

**Note:** To use a *bounded-staleness* timestamp bound, use the sample code to [perform a strong read](#perform-strong-read) , but additionally specify a `  single_use:  ` parameter with a value of `  {max_staleness: [n seconds] }  ` to read with a staleness of \[n seconds\] .

### Perform a read using an index

The following shows how to read zero or more rows from a database using an [index](/spanner/docs/secondary-indexes) :

### GoogleSQL

### C++

Use the `  Read()  ` function to perform a read using an index.

``` cpp
void ReadDataWithIndex(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  auto rows =
      client.Read("Albums", google::cloud::spanner::KeySet::All(),
                  {"AlbumId", "AlbumTitle"},
                  google::cloud::Options{}.set<spanner::ReadIndexNameOption>(
                      "AlbumsByAlbumTitle"));
  using RowType = std::tuple<std::int64_t, std::string>;
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "AlbumId: " << std::get<0>(*row) << "\t";
    std::cout << "AlbumTitle: " << std::get<1>(*row) << "\n";
  }
  std::cout << "Read completed for [spanner_read_data_with_index]\n";
}
```

### C\#

Read data using the index by executing a query that explicitly specifies the index:

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithIndexAsyncSample
{
    public class Album
    {
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
        public long MarketingBudget { get; set; }
    }

    public async Task<List<Album>> QueryDataWithIndexAsync(string projectId, string instanceId, string databaseId,
        string startTitle, string endTitle)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            "SELECT AlbumId, AlbumTitle, MarketingBudget FROM Albums@ "
            + "{FORCE_INDEX=AlbumsByAlbumTitle} "
            + $"WHERE AlbumTitle >= @startTitle "
            + $"AND AlbumTitle < @endTitle",
            new SpannerParameterCollection
            {
                { "startTitle", SpannerDbType.String, startTitle },
                { "endTitle", SpannerDbType.String, endTitle }
            });

        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                AlbumTitle = reader.GetFieldValue<string>("AlbumTitle"),
                MarketingBudget = reader.IsDBNull(reader.GetOrdinal("MarketingBudget")) ? 0 : reader.GetFieldValue<long>("MarketingBudget")
            });
        }
        return albums;
    }
}
```

### Go

Use `  Client.Single().ReadUsingIndex  ` to read rows from the database using an index.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readUsingIndex(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 iter := client.Single().ReadUsingIndex(ctx, "Albums", "AlbumsByAlbumTitle", spanner.AllKeys(),
     []string{"AlbumId", "AlbumTitle"})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var albumID int64
     var albumTitle string
     if err := row.Columns(&albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s\n", albumID, albumTitle)
 }
}
```

### Java

Use `  ReadContext.readUsingIndex  ` to read rows from the database using an index.

``` java
static void readUsingIndex(DatabaseClient dbClient) {
  try (ResultSet resultSet =
      dbClient
          .singleUse()
          .readUsingIndex(
              "Albums",
              "AlbumsByAlbumTitle",
              KeySet.all(),
              Arrays.asList("AlbumId", "AlbumTitle"))) {
    while (resultSet.next()) {
      System.out.printf("%d %s\n", resultSet.getLong(0), resultSet.getString(1));
    }
  }
}
```

### Node.js

Use `  Table.read  ` and specify the index in the query to read rows from the database using an index.

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud Spanner client library
const {Spanner} = require('@google-cloud/spanner');

// Instantiates a client
const spanner = new Spanner({
  projectId: projectId,
});

async function readDataWithIndex() {
  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  const albumsTable = database.table('Albums');

  const query = {
    columns: ['AlbumId', 'AlbumTitle'],
    keySet: {
      all: true,
    },
    index: 'AlbumsByAlbumTitle',
  };

  // Reads the Albums table using an index
  try {
    const [rows] = await albumsTable.read(query);

    rows.forEach(row => {
      const json = row.toJSON();
      console.log(`AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`);
    });
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
}
readDataWithIndex();
```

### PHP

Use `  Database::read  ` and specify the index to read rows from the database using an index.

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads sample data from the database using an index.
 *
 * The index must exist before running this sample. You can add the index
 * by running the `add_index` sample or by running this DDL statement against
 * your database:
 *
 *     CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)
 *
 * Example:
 * ```
 * read_data_with_index($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_data_with_index(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['AlbumId', 'AlbumTitle'],
        ['index' => 'AlbumsByAlbumTitle']
    );

    foreach ($results->rows() as $row) {
        printf('AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

### Python

Use `  Database.read  ` and specify the index to read rows from the database using an index.

``` python
def read_data_with_index(instance_id, database_id):
    """Reads sample data from the database using an index.

    The index must exist before running this sample. You can add the index
    by running the `add_index` sample or by running this DDL statement against
    your database:

        CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)

    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums",
            columns=("AlbumId", "AlbumTitle"),
            keyset=keyset,
            index="AlbumsByAlbumTitle",
        )

        for row in results:
            print("AlbumId: {}, AlbumTitle: {}".format(*row))
```

### Ruby

Use `  Client#read  ` and specify the index to read rows from the database using an index.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

result = client.read "Albums", [:AlbumId, :AlbumTitle],
                     index: "AlbumsByAlbumTitle"

result.rows.each do |row|
  puts "#{row[:AlbumId]} #{row[:AlbumTitle]}"
end
```

## Read data in parallel

When performing bulk read or query operations involving very large amounts of data from Spanner, you can use the [`  PartitionQuery  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery) API for faster results. The API divides the query into batches, or *partitions* , by using multiple machines to fetch the partitions in parallel. Be mindful that using the `  PartitionQuery  ` API cause higher latency because it is only intended for bulk operations such as exporting or scanning the whole database.

You can perform any read API operation in parallel using the Spanner client libraries. However, you can only partition SQL queries when queries are root-partitionable. For a query to be root-partitionable, the query plan must satisfy one of the following conditions:

  - The first operator in the query execution plan is a [**distributed union**](/spanner/docs/query-execution-operators#distributed_union) and the query execution plan only contains one [**distributed union**](/spanner/docs/query-execution-operators#distributed_union) (excluding Local Distribution Unions). Your query plan can't contain any other distributed operators, such as [**distributed cross apply**](/spanner/docs/query-execution-operators#distributed-cross-apply) .

  - There are no distributed operators in the query plan.

The `  PartitionQuery  ` API runs the queries in batch mode. Spanner might choose a query execution plan that makes the queries root-partitionable when run in batch mode. As a result, the `  PartitionQuery  ` API and Spanner Studio might use different query execution plans for the same query. You might not be able to get the query execution plan used by the `  PartitionQuery  ` API on Spanner Studio.

For partitioned queries like this, you can choose to enable Spanner Data Boost. Data Boost lets you run large analytic queries with near-zero impact to existing workloads on the provisioned Spanner instance. The C++, Go, Java, Node.js, and Python code examples on this page show how to enable Data Boost.

For more information about Data Boost, see [Data Boost overview](/spanner/docs/databoost/databoost-overview) .

**Note:** The following samples work with the DDL table definition found in the [Quickstart](/spanner/docs/create-query-database-console#short-schema) .

### GoogleSQL

### C++

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner batch transaction.
  - Generating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from multiple parallel tasks within a single process. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

``` cpp
void UsePartitionQuery(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto txn = spanner::MakeReadOnlyTransaction();

  spanner::SqlStatement select(
      "SELECT SingerId, FirstName, LastName FROM Singers");
  using RowType = std::tuple<std::int64_t, std::string, std::string>;

  auto partitions = client.PartitionQuery(
      std::move(txn), std::move(select),
      google::cloud::Options{}.set<spanner::PartitionDataBoostOption>(true));
  if (!partitions) throw std::move(partitions).status();

  // You would probably choose to execute these partitioned queries in
  // separate threads/processes, or on a different machine.
  int number_of_rows = 0;
  for (auto const& partition : *partitions) {
    auto rows = client.ExecuteQuery(partition);
    for (auto& row : spanner::StreamOf<RowType>(rows)) {
      if (!row) throw std::move(row).status();
      number_of_rows++;
    }
  }
  std::cout << "Number of partitions: " << partitions->size() << "\n"
            << "Number of rows: " << number_of_rows << "\n";
  std::cout << "Read completed for [spanner_batch_client]\n";
}
```

### C\#

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner batch transaction.
  - Generating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from multiple parallel tasks within a single process. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

public class BatchReadRecordsAsyncSample
{
    private int _rowsRead;
    private int _partitionCount;
    public async Task<(int RowsRead, int Partitions)> BatchReadRecordsAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var transaction = await connection.BeginTransactionAsync(
            SpannerTransactionCreationOptions.ReadOnly.WithIsDetached(true),
            new SpannerTransactionOptions { DisposeBehavior = DisposeBehavior.CloseResources },
            cancellationToken: default);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, FirstName, LastName FROM Singers");
        cmd.Transaction = transaction;

        // A CommandPartition object is serializable and can be used from a different process.
        // If data boost is enabled, partitioned read and query requests will be executed
        // using Spanner independent compute resources.
        var partitions = await cmd.GetReaderPartitionsAsync(PartitionOptions.Default.WithDataBoostEnabled(true));

        var transactionId = transaction.TransactionId;
        await Task.WhenAll(partitions.Select(x => DistributedReadWorkerAsync(x, transactionId)));
        Console.WriteLine($"Done reading!  Total rows read: {_rowsRead:N0} with {_partitionCount} partition(s)");
        return (RowsRead: _rowsRead, Partitions: _partitionCount);
    }

    private async Task DistributedReadWorkerAsync(CommandPartition readPartition, TransactionId id)
    {
        var localId = Interlocked.Increment(ref _partitionCount);
        using var connection = new SpannerConnection(id.ConnectionString);
        using var transaction = await connection.BeginTransactionAsync(
            SpannerTransactionCreationOptions.FromReadOnlyTransactionId(id),
            transactionOptions: null,
            cancellationToken: default);
        using var cmd = connection.CreateCommandWithPartition(readPartition, transaction);
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Interlocked.Increment(ref _rowsRead);
            Console.WriteLine($"Partition ({localId}) "
                + $"{reader.GetFieldValue<int>("SingerId")}"
                + $" {reader.GetFieldValue<string>("FirstName")}"
                + $" {reader.GetFieldValue<string>("LastName")}");
        }
        Console.WriteLine($"Done with single reader {localId}.");
    }
}
```

### Go

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner client and a transaction.
  - Generating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from a single machine and executes the partitions to fetch the results. If you have more than one processor available, the executions can also be run in parallel. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readBatchData(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 txn, err := client.BatchReadOnlyTransaction(ctx, spanner.StrongRead())
 if err != nil {
     return err
 }
 defer txn.Close()

 // Singer represents a row in the Singers table.
 type Singer struct {
     SingerID   int64
     FirstName  string
     LastName   string
     SingerInfo []byte
 }
 stmt := spanner.Statement{SQL: "SELECT SingerId, FirstName, LastName FROM Singers;"}
 // A Partition object is serializable and can be used from a different process.
 // DataBoost option is an optional parameter which can also be used for partition read
 // and query to execute the request via spanner independent compute resources.
 partitions, err := txn.PartitionQueryWithOptions(ctx, stmt, spanner.PartitionOptions{}, spanner.QueryOptions{DataBoostEnabled: true})
 if err != nil {
     return err
 }
 recordCount := 0
 for i, p := range partitions {
     iter := txn.Execute(ctx, p)
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             break
         } else if err != nil {
             return err
         }
         var s Singer
         if err := row.ToStruct(&s); err != nil {
             return err
         }
         fmt.Fprintf(w, "Partition (%d) %v\n", i, s)
         recordCount++
     }
 }
 fmt.Fprintf(w, "Total partition count: %v\n", len(partitions))
 fmt.Fprintf(w, "Total record count: %v\n", recordCount)
 return nil
}
```

### Java

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner batch client and a transaction.
  - Generating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from a single machine, running multiple jobs in parallel if you have more than one processor available. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

``` java
int numThreads = Runtime.getRuntime().availableProcessors();
ExecutorService executor = Executors.newFixedThreadPool(numThreads);

// Statistics
int totalPartitions;
AtomicInteger totalRecords = new AtomicInteger(0);

try {
  BatchClient batchClient =
      spanner.getBatchClient(DatabaseId.of(options.getProjectId(), instanceId, databaseId));

  final BatchReadOnlyTransaction txn =
      batchClient.batchReadOnlyTransaction(TimestampBound.strong());

  // A Partition object is serializable and can be used from a different process.
  // DataBoost option is an optional parameter which can be used for partition read
  // and query to execute the request via spanner independent compute resources.

  List<Partition> partitions =
      txn.partitionQuery(
          PartitionOptions.getDefaultInstance(),
          Statement.of("SELECT SingerId, FirstName, LastName FROM Singers"),
          // Option to enable data boost for a given request
          Options.dataBoostEnabled(true));

  totalPartitions = partitions.size();

  for (final Partition p : partitions) {
    executor.execute(
        () -> {
          try (ResultSet results = txn.execute(p)) {
            while (results.next()) {
              long singerId = results.getLong(0);
              String firstName = results.getString(1);
              String lastName = results.getString(2);
              System.out.println("[" + singerId + "] " + firstName + " " + lastName);
              totalRecords.getAndIncrement();
            }
          }
        });
  }
} finally {
  executor.shutdown();
  executor.awaitTermination(1, TimeUnit.HOURS);
  spanner.close();
}

double avgRecordsPerPartition = 0.0;
if (totalPartitions != 0) {
  avgRecordsPerPartition = (double) totalRecords.get() / totalPartitions;
}
System.out.println("totalPartitions=" + totalPartitions);
System.out.println("totalRecords=" + totalRecords);
System.out.println("avgRecordsPerPartition=" + avgRecordsPerPartition);
```

### Node.js

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner client and a batch.
  - Generating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from a single machine, running multiple jobs in parallel if you have more than one processor available. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

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
const [transaction] = await database.createBatchTransaction();

const query = {
  sql: 'SELECT * FROM Singers',
  // DataBoost option is an optional parameter which can also be used for partition read
  // and query to execute the request via spanner independent compute resources.
  dataBoostEnabled: true,
};

// A Partition object is serializable and can be used from a different process.
const [partitions] = await transaction.createQueryPartitions(query);
console.log(`Successfully created ${partitions.length} query partitions.`);

let row_count = 0;
const promises = [];
partitions.forEach(partition => {
  promises.push(
    transaction.execute(partition).then(results => {
      const rows = results[0].map(row => row.toJSON());
      row_count += rows.length;
    }),
  );
});
Promise.all(promises)
  .then(() => {
    console.log(
      `Successfully received ${row_count} from executed partitions.`,
    );
    transaction.close();
  })
  .then(() => {
    database.close();
  });
```

### PHP

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner client and a batch.
  - Generating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from a single machine and executes the partitions to fetch the results. If you have more than one processor available, the executions can also be run in parallel. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL.
 * Example:
 * ```
 * batch_query_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function batch_query_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $batch = $spanner->batch($instanceId, $databaseId);
    $snapshot = $batch->snapshot();
    $queryString = 'SELECT SingerId, FirstName, LastName FROM Singers';
    $partitions = $snapshot->partitionQuery($queryString, [
        // This is an optional parameter which can be used for partition
        // read and query to execute the request via spanner independent
        // compute resources.
        'dataBoostEnabled' => true
    ]);
    $totalPartitions = count($partitions);
    $totalRecords = 0;
    foreach ($partitions as $partition) {
        $result = $snapshot->executePartition($partition);
        $rows = $result->rows();
        foreach ($rows as $row) {
            $singerId = $row['SingerId'];
            $firstName = $row['FirstName'];
            $lastName = $row['LastName'];
            printf('SingerId: %s, FirstName: %s, LastName: %s' . PHP_EOL, $singerId, $firstName, $lastName);
            $totalRecords++;
        }
    }
    printf('Total Partitions: %d' . PHP_EOL, $totalPartitions);
    printf('Total Records: %d' . PHP_EOL, $totalRecords);
    $averageRecordsPerPartition = $totalRecords / $totalPartitions;
    printf('Average Records Per Partition: %f' . PHP_EOL, $averageRecordsPerPartition);
}
````

### Python

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner client and a batch transaction.
  - Generating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from a single machine, running multiple jobs in parallel if you have more than one processor available. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

``` python
def run_batch_query(instance_id, database_id):
    """Runs an example batch query."""

    # Expected Table Format:
    # CREATE TABLE Singers (
    #   SingerId   INT64 NOT NULL,
    #   FirstName  STRING(1024),
    #   LastName   STRING(1024),
    #   SingerInfo BYTES(MAX),
    # ) PRIMARY KEY (SingerId);

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    # Create the batch transaction and generate partitions
    snapshot = database.batch_snapshot()
    partitions = snapshot.generate_read_batches(
        table="Singers",
        columns=("SingerId", "FirstName", "LastName"),
        keyset=spanner.KeySet(all_=True),
        # A Partition object is serializable and can be used from a different process.
        # DataBoost option is an optional parameter which can also be used for partition read
        # and query to execute the request via spanner independent compute resources.
        data_boost_enabled=True,
    )

    # Create a pool of workers for the tasks
    start = time.time()
    with concurrent.futures.ThreadPoolExecutor() as executor:
        futures = [executor.submit(process, snapshot, p) for p in partitions]

        for future in concurrent.futures.as_completed(futures, timeout=3600):
            finish, row_ct = future.result()
            elapsed = finish - start
            print("Completed {} rows in {} seconds".format(row_ct, elapsed))

    # Clean up
    snapshot.close()


def process(snapshot, partition):
    """Processes the requests of a query in an separate process."""
    print("Started processing partition.")
    row_ct = 0
    for row in snapshot.process_read_batch(partition):
        print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
        row_ct += 1
    return time.time(), row_ct
```

### Ruby

This example fetches partitions of a SQL query of the `  Singers  ` table and executes the query over each partition through the following steps:

  - Creating a Spanner batch client.
  - Creating partitions for the query, so that the partitions can be distributed to multiple workers.
  - Retrieving the query results for each partition.

**Note:** For the sake of simplicity, the example fetches all of the partitions from a single machine, running multiple jobs in parallel if you have more than one processor available. For real-world use cases, you should distribute partitions to workers on different machines to improve efficiency.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

# Prepare a thread pool with number of processors
processor_count  = Concurrent.processor_count
thread_pool      = Concurrent::FixedThreadPool.new processor_count

# Prepare AtomicFixnum to count total records using multiple threads
total_records = Concurrent::AtomicFixnum.new

# Create a new Spanner batch client
spanner        = Google::Cloud::Spanner.new project: project_id
batch_client   = spanner.batch_client instance_id, database_id

# Get a strong timestamp bound batch_snapshot
batch_snapshot = batch_client.batch_snapshot strong: true

# Get partitions for specified query
# data_boost_enabled option is an optional parameter which can be used for partition read
# and query to execute the request via spanner independent compute resources.
partitions       = batch_snapshot.partition_query "SELECT SingerId, FirstName, LastName FROM Singers", data_boost_enabled: true
total_partitions = partitions.size

# Enqueue a new thread pool job
partitions.each_with_index do |partition, _partition_index|
  thread_pool.post do
    # Increment total_records per new row
    batch_snapshot.execute_partition(partition).rows.each do |_row|
      total_records.increment
    end
  end
end

# Wait for queued jobs to complete
thread_pool.shutdown
thread_pool.wait_for_termination

# Close the client connection and release resources.
batch_snapshot.close

# Collect statistics for batch query
average_records_per_partition = 0.0
if total_partitions != 0
  average_records_per_partition = total_records.value / total_partitions.to_f
end

puts "Total Partitions: #{total_partitions}"
puts "Total Records: #{total_records.value}"
puts "Average records per Partition: #{average_records_per_partition}"
```
