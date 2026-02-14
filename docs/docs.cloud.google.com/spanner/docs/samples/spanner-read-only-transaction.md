Use a read-only transaction.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Getting started with Spanner and PGAdapter](/spanner/docs/getting-started/pgadapter)
  - [Getting started with Spanner in C\#](/spanner/docs/getting-started/csharp)
  - [Getting started with Spanner in C++](/spanner/docs/getting-started/cpp)
  - [Getting started with Spanner in Go](/spanner/docs/getting-started/go)
  - [Getting started with Spanner in Go database/sql](/spanner/docs/getting-started/database_sql)
  - [Getting started with Spanner in Java](/spanner/docs/getting-started/java)
  - [Getting started with Spanner in JDBC](/spanner/docs/getting-started/jdbc)
  - [Getting started with Spanner in Node.js](/spanner/docs/getting-started/nodejs)
  - [Getting started with Spanner in PHP](/spanner/docs/getting-started/php)
  - [Getting started with Spanner in Python](/spanner/docs/getting-started/python)
  - [Getting started with Spanner in Ruby](/spanner/docs/getting-started/ruby)
  - [Transactions overview](/spanner/docs/transactions)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void ReadOnlyTransaction(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto read_only = spanner::MakeReadOnlyTransaction();

  spanner::SqlStatement select(
      "SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
  using RowType = std::tuple<std::int64_t, std::int64_t, std::string>;

  // Read#1.
  auto rows1 = client.ExecuteQuery(read_only, select);
  std::cout << "Read 1 results\n";
  for (auto& row : spanner::StreamOf<RowType>(rows1)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " AlbumId: " << std::get<1>(*row)
              << " AlbumTitle: " << std::get<2>(*row) << "\n";
  }
  // Read#2. Even if changes occur in-between the reads the transaction ensures
  // that Read #1 and Read #2 return the same data.
  auto rows2 = client.ExecuteQuery(read_only, select);
  std::cout << "Read 2 results\n";
  for (auto& row : spanner::StreamOf<RowType>(rows2)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " AlbumId: " << std::get<1>(*row)
              << " AlbumTitle: " << std::get<2>(*row) << "\n";
  }
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Transactions;

public class QueryDataWithTransactionAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> QueryDataWithTransactionAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();
        using TransactionScope scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
        using var connection = new SpannerConnection(connectionString);

        // Opens the connection so that the Spanner transaction included in the TransactionScope
        // is read-only TimestampBound.Strong.
        await connection.OpenAsync(SpannerTransactionCreationOptions.ReadOnly, options: null, cancellationToken: default);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

        // Read #1.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine("SingerId : " + reader.GetFieldValue<string>("SingerId")
                    + " AlbumId : " + reader.GetFieldValue<string>("AlbumId")
                    + " AlbumTitle : " + reader.GetFieldValue<string>("AlbumTitle"));
            }
        }

        // Read #2. Even if changes occur in-between the reads,
        // the transaction ensures that Read #1 and Read #2
        // return the same data.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                albums.Add(new Album
                {
                    AlbumId = reader.GetFieldValue<int>("AlbumId"),
                    SingerId = reader.GetFieldValue<int>("SingerId"),
                    AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
                });
            }
        }
        scope.Complete();
        Console.WriteLine("Transaction complete.");
        return albums;
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readOnlyTransaction(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 ro := client.ReadOnlyTransaction()
 defer ro.Close()
 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := ro.Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         break
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

 iter = ro.Read(ctx, "Albums", spanner.AllKeys(), []string{"SingerId", "AlbumId", "AlbumTitle"})
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

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void readOnlyTransaction(DatabaseClient dbClient) {
  // ReadOnlyTransaction must be closed by calling close() on it to release resources held by it.
  // We use a try-with-resource block to automatically do so.
  try (ReadOnlyTransaction transaction = dbClient.readOnlyTransaction()) {
    try (ResultSet queryResultSet =
        transaction.executeQuery(
            Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"))) {
      while (queryResultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            queryResultSet.getLong(0), queryResultSet.getLong(1), queryResultSet.getString(2));
      }
    } // queryResultSet.close() is automatically called here
    try (ResultSet readResultSet =
        transaction.read(
          "Albums", KeySet.all(), Arrays.asList("SingerId", "AlbumId", "AlbumTitle"))) {
      while (readResultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            readResultSet.getLong(0), readResultSet.getLong(1), readResultSet.getString(2));
      }
    } // readResultSet.close() is automatically called here
  } // transaction.close() is automatically called here
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

// Gets a transaction object that captures the database state
// at a specific point in time
database.getSnapshot(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  const queryOne = 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums';

  try {
    // Read #1, using SQL
    const [qOneRows] = await transaction.run(queryOne);

    qOneRows.forEach(row => {
      const json = row.toJSON();
      console.log(
        `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
      );
    });

    const queryTwo = {
      columns: ['SingerId', 'AlbumId', 'AlbumTitle'],
    };

    // Read #2, using the `read` method. Even if changes occur
    // in-between the reads, the transaction ensures that both
    // return the same data.
    const [qTwoRows] = await transaction.read('Albums', queryTwo);

    qTwoRows.forEach(row => {
      const json = row.toJSON();
      console.log(
        `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
      );
    });

    console.log('Successfully executed read-only transaction.');
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    transaction.end();
    // Close the database when finished.
    await database.close();
  }
});
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads data inside of a read-only transaction.
 *
 * Within the read-only transaction, or "snapshot", the application sees
 * consistent view of the database at a particular timestamp.
 * Example:
 * ```
 * read_only_transaction($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_only_transaction(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $snapshot = $database->snapshot();
    $results = $snapshot->execute(
        'SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
    );
    print('Results from the first read:' . PHP_EOL);
    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }

    // Perform another read using the `read` method. Even if the data
    // is updated in-between the reads, the snapshot ensures that both
    // return the same data.
    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['SingerId', 'AlbumId', 'AlbumTitle']
    );

    print('Results from the second read:' . PHP_EOL);
    foreach ($results->rows() as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def read_only_transaction(instance_id, database_id):
    """Reads data inside of a read-only transaction.

    Within the read-only transaction, or "snapshot", the application sees
    consistent view of the database at a particular timestamp.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot(multi_use=True) as snapshot:
        # Read using SQL.
        results = snapshot.execute_sql(
            "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
        )

        print("Results from first read:")
        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))

        # Perform another read using the `read` method. Even if the data
        # is updated in-between the reads, the snapshot ensures that both
        # return the same data.
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums", columns=("SingerId", "AlbumId", "AlbumTitle"), keyset=keyset
        )

        print("Results from second read:")
        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.snapshot do |snapshot|
  snapshot.execute("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").rows.each do |row|
    puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
  end

  # Even if changes occur in-between the reads, the transaction ensures that
  # both return the same data.
  snapshot.read("Albums", [:AlbumId, :AlbumTitle, :SingerId]).rows.each do |row|
    puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
  end
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
