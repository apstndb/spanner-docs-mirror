Insert several rows of data into a table by using mutations.

## Explore further

For detailed documentation that includes this code sample, see the following:

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
  - [Insert, update, and delete data using mutations](/spanner/docs/modify-mutation-api)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void InsertData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto insert_singers = spanner::InsertMutationBuilder(
                            "Singers", {"SingerId", "FirstName", "LastName"})
                            .EmplaceRow(1, "Marc", "Richards")
                            .EmplaceRow(2, "Catalina", "Smith")
                            .EmplaceRow(3, "Alice", "Trentor")
                            .EmplaceRow(4, "Lea", "Martin")
                            .EmplaceRow(5, "David", "Lomond")
                            .Build();

  auto insert_albums = spanner::InsertMutationBuilder(
                           "Albums", {"SingerId", "AlbumId", "AlbumTitle"})
                           .EmplaceRow(1, 1, "Total Junk")
                           .EmplaceRow(1, 2, "Go, Go, Go")
                           .EmplaceRow(2, 1, "Green")
                           .EmplaceRow(2, 2, "Forever Hold Your Peace")
                           .EmplaceRow(2, 3, "Terrified")
                           .Build();

  auto commit_result =
      client.Commit(spanner::Mutations{insert_singers, insert_albums});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Insert was successful [spanner_insert_data]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class InsertDataAsyncSample
{
    public class Singer
    {
        public int SingerId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task InsertDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        List<Singer> singers = new List<Singer>
        {
            new Singer { SingerId = 1, FirstName = "Marc", LastName = "Richards" },
            new Singer { SingerId = 2, FirstName = "Catalina", LastName = "Smith" },
            new Singer { SingerId = 3, FirstName = "Alice", LastName = "Trentor" },
            new Singer { SingerId = 4, FirstName = "Lea", LastName = "Martin" },
            new Singer { SingerId = 5, FirstName = "David", LastName = "Lomond" },
        };
        List<Album> albums = new List<Album>
        {
            new Album { SingerId = 1, AlbumId = 1, AlbumTitle = "Total Junk" },
            new Album { SingerId = 1, AlbumId = 2, AlbumTitle = "Go, Go, Go" },
            new Album { SingerId = 2, AlbumId = 1, AlbumTitle = "Green" },
            new Album { SingerId = 2, AlbumId = 2, AlbumTitle = "Forever Hold your Peace" },
            new Album { SingerId = 2, AlbumId = 3, AlbumTitle = "Terrified" },
        };

        // Create connection to Cloud Spanner.
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        await connection.RunWithRetriableTransactionAsync(async transaction =>
        {
            await Task.WhenAll(singers.Select(singer =>
            {
                // Insert rows into the Singers table.
                using var cmd = connection.CreateInsertCommand("Singers", new SpannerParameterCollection
                {
                        { "SingerId", SpannerDbType.Int64, singer.SingerId },
                        { "FirstName", SpannerDbType.String, singer.FirstName },
                        { "LastName", SpannerDbType.String, singer.LastName }
                });
                cmd.Transaction = transaction;
                return cmd.ExecuteNonQueryAsync();
            }));

            await Task.WhenAll(albums.Select(album =>
            {
                // Insert rows into the Albums table.
                using var cmd = connection.CreateInsertCommand("Albums", new SpannerParameterCollection
                {
                        { "SingerId", SpannerDbType.Int64, album.SingerId },
                        { "AlbumId", SpannerDbType.Int64, album.AlbumId },
                        { "AlbumTitle", SpannerDbType.String,album.AlbumTitle }
                });
                cmd.Transaction = transaction;
                return cmd.ExecuteNonQueryAsync();
            }));
        });
        Console.WriteLine("Data inserted.");
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func write(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 singerColumns := []string{"SingerId", "FirstName", "LastName"}
 albumColumns := []string{"SingerId", "AlbumId", "AlbumTitle"}
 m := []*spanner.Mutation{
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{1, "Marc", "Richards"}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{2, "Catalina", "Smith"}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{3, "Alice", "Trentor"}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{4, "Lea", "Martin"}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{5, "David", "Lomond"}),
     spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{1, 1, "Total Junk"}),
     spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{1, 2, "Go, Go, Go"}),
     spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{2, 1, "Green"}),
     spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{2, 2, "Forever Hold Your Peace"}),
     spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{2, 3, "Terrified"}),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static final List<Singer> SINGERS =
    Arrays.asList(
        new Singer(1, "Marc", "Richards"),
        new Singer(2, "Catalina", "Smith"),
        new Singer(3, "Alice", "Trentor"),
        new Singer(4, "Lea", "Martin"),
        new Singer(5, "David", "Lomond"));

static final List<Album> ALBUMS =
    Arrays.asList(
        new Album(1, 1, "Total Junk"),
        new Album(1, 2, "Go, Go, Go"),
        new Album(2, 1, "Green"),
        new Album(2, 2, "Forever Hold Your Peace"),
        new Album(2, 3, "Terrified"));
static void writeExampleData(DatabaseClient dbClient) {
  List<Mutation> mutations = new ArrayList<>();
  for (Singer singer : SINGERS) {
    mutations.add(
        Mutation.newInsertBuilder("Singers")
            .set("SingerId")
            .to(singer.singerId)
            .set("FirstName")
            .to(singer.firstName)
            .set("LastName")
            .to(singer.lastName)
            .build());
  }
  for (Album album : ALBUMS) {
    mutations.add(
        Mutation.newInsertBuilder("Albums")
            .set("SingerId")
            .to(album.singerId)
            .set("AlbumId")
            .to(album.albumId)
            .set("AlbumTitle")
            .to(album.albumTitle)
            .build());
  }
  dbClient.write(mutations);
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

// Instantiate Spanner table objects
const singersTable = database.table('Singers');
const albumsTable = database.table('Albums');

// Inserts rows into the Singers table
// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so
// they must be converted to strings before being inserted as INT64s
try {
  await singersTable.insert([
    {SingerId: '1', FirstName: 'Marc', LastName: 'Richards'},
    {SingerId: '2', FirstName: 'Catalina', LastName: 'Smith'},
    {SingerId: '3', FirstName: 'Alice', LastName: 'Trentor'},
    {SingerId: '4', FirstName: 'Lea', LastName: 'Martin'},
    {SingerId: '5', FirstName: 'David', LastName: 'Lomond'},
  ]);

  await albumsTable.insert([
    {SingerId: '1', AlbumId: '1', AlbumTitle: 'Total Junk'},
    {SingerId: '1', AlbumId: '2', AlbumTitle: 'Go, Go, Go'},
    {SingerId: '2', AlbumId: '1', AlbumTitle: 'Green'},
    {SingerId: '2', AlbumId: '2', AlbumTitle: 'Forever Hold your Peace'},
    {SingerId: '2', AlbumId: '3', AlbumTitle: 'Terrified'},
  ]);

  console.log('Inserted data.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  await database.close();
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Inserts sample data into the given database.
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
function insert_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->insertBatch('Singers', [
            ['SingerId' => 1, 'FirstName' => 'Marc', 'LastName' => 'Richards'],
            ['SingerId' => 2, 'FirstName' => 'Catalina', 'LastName' => 'Smith'],
            ['SingerId' => 3, 'FirstName' => 'Alice', 'LastName' => 'Trentor'],
            ['SingerId' => 4, 'FirstName' => 'Lea', 'LastName' => 'Martin'],
            ['SingerId' => 5, 'FirstName' => 'David', 'LastName' => 'Lomond'],
        ])
        ->insertBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'AlbumTitle' => 'Total Junk'],
            ['SingerId' => 1, 'AlbumId' => 2, 'AlbumTitle' => 'Go, Go, Go'],
            ['SingerId' => 2, 'AlbumId' => 1, 'AlbumTitle' => 'Green'],
            ['SingerId' => 2, 'AlbumId' => 2, 'AlbumTitle' => 'Forever Hold Your Peace'],
            ['SingerId' => 2, 'AlbumId' => 3, 'AlbumTitle' => 'Terrified']
        ])
        ->commit();

    print('Inserted data.' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def insert_data(instance_id, database_id):
    """Inserts sample data into the given database.

    The database and table must already exist and can be created using
    `create_database`.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.batch() as batch:
        batch.insert(
            table="Singers",
            columns=("SingerId", "FirstName", "LastName"),
            values=[
                (1, "Marc", "Richards"),
                (2, "Catalina", "Smith"),
                (3, "Alice", "Trentor"),
                (4, "Lea", "Martin"),
                (5, "David", "Lomond"),
            ],
        )

        batch.insert(
            table="Albums",
            columns=("SingerId", "AlbumId", "AlbumTitle"),
            values=[
                (1, 1, "Total Junk"),
                (1, 2, "Go, Go, Go"),
                (2, 1, "Green"),
                (2, 2, "Forever Hold Your Peace"),
                (2, 3, "Terrified"),
            ],
        )

    print("Inserted data.")
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

client.commit do |c|
  c.insert "Singers", [
    { SingerId: 1, FirstName: "Marc",     LastName: "Richards" },
    { SingerId: 2, FirstName: "Catalina", LastName: "Smith"    },
    { SingerId: 3, FirstName: "Alice",    LastName: "Trentor"  },
    { SingerId: 4, FirstName: "Lea",      LastName: "Martin"   },
    { SingerId: 5, FirstName: "David",    LastName: "Lomond"   }
  ]
  c.insert "Albums", [
    { SingerId: 1, AlbumId: 1, AlbumTitle: "Total Junk" },
    { SingerId: 1, AlbumId: 2, AlbumTitle: "Go, Go, Go" },
    { SingerId: 2, AlbumId: 1, AlbumTitle: "Green" },
    { SingerId: 2, AlbumId: 2, AlbumTitle: "Forever Hold Your Peace" },
    { SingerId: 2, AlbumId: 3, AlbumTitle: "Terrified" }
  ]
end

puts "Inserted data"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
