This page describes how to insert, update, and delete data using mutations. A mutation represents a sequence of inserts, updates, and deletes that Spanner applies atomically to different rows and tables in a database. Mutations are designed for writing data. They can't read data from your tables. Many update operations must read existing data before performing modifications. For these use cases, you must use a [read-write transaction](/spanner/docs/transactions#read-write_transactions) , which lets Spanner read rows and then apply mutations within the same atomic operation.

Although you can commit mutations by using [gRPC](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitRequest) or [REST](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit) , it is more common to access the APIs through the client libraries.

If you need to commit a large number of blind writes, but don't require an atomic transaction, you can bulk modify your Spanner tables using batch write. For more information, see [Modify data using batch writes](/spanner/docs/batch-write) .

## Insert new rows in a table

### C++

You write data using the `  InsertMutationBuilder()  ` function. `  Client::Commit()  ` adds new rows to a table. All inserts in a single batch are applied atomically.

This code shows how to write the data:

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

You can insert data using the [`  connection.CreateInsertCommand()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_CreateInsertCommand_System_String_Google_Cloud_Spanner_Data_SpannerParameterCollection_) method, which creates a new `  SpannerCommand  ` to insert rows into a table. The [`  SpannerCommand.ExecuteNonQueryAsync()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerCommand#Google_Cloud_Spanner_Data_SpannerCommand_ExecuteNonQueryAsync_System_Threading_CancellationToken_) method adds new rows to the table.

**Note:** You can run multiple transactions in parallel using a single `  SpannerConnection  ` object. When running additional transactions, you must ensure that the `  SpannerConnection  ` object is in the `  Open  ` state before you execute additional transaction commands by calling the [`  SpannerCommand.ExecuteNonQueryAsync()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_OpenAsync_System_Threading_CancellationToken_) method, as seen in the following example.

This code shows how to insert data:

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

You write data using a [`  Mutation  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Mutation) . A `  Mutation  ` is a container for mutation operations. A `  Mutation  ` represents a sequence of inserts, updates, or deletes that can be applied atomically to different rows and tables in a Spanner database.

Use [`  Mutation.InsertOrUpdate()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#InsertOrUpdate) to construct an `  INSERT_OR_UPDATE  ` mutation, which adds a new row or updates column values if the row already exists. Alternatively, use [`  Mutation.Insert()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Insert) method to construct an `  INSERT  ` mutation, which adds a new row.

[`  Client.Apply()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client.Apply) applies mutations atomically to a database.

This code shows how to write the data:

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

You write data using a [`  Mutation  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation) object. A `  Mutation  ` object is a container for mutation operations. A `  Mutation  ` represents a sequence of inserts, updates, and deletes that Spanner applies atomically to different rows and tables in a Spanner database.

The [`  newInsertBuilder()  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation#com_google_cloud_spanner_Mutation_newInsertBuilder_java_lang_String_) method in the `  Mutation  ` class constructs an `  INSERT  ` mutation, which inserts a new row in a table. If the row already exists, the write fails. Alternatively, you can use the [`  newInsertOrUpdateBuilder  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation#com_google_cloud_spanner_Mutation_newInsertOrUpdateBuilder_java_lang_String_) method to construct an `  INSERT_OR_UPDATE  ` mutation, which updates column values if the row already exists.

The [`  write()  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.DatabaseClient#com_google_cloud_spanner_DatabaseClient_write_java_lang_Iterable_com_google_cloud_spanner_Mutation__) method in the `  DatabaseClient  ` class writes the mutations. All mutations in a single batch are applied atomically.

This code shows how to write the data:

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

You write data using a [`  Table  `](https://googleapis.dev/nodejs/spanner/latest/Table.html) object. The [`  Table.insert()  `](https://googleapis.dev/nodejs/spanner/latest/Table.html#insert) method adds new rows to the table. All inserts in a single batch are applied atomically.

This code shows how to write the data:

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

You write data using the [`  Database::insertBatch  `](/php/docs/reference/cloud-spanner/latest/database?method=insertBatch) method. `  insertBatch  ` adds new rows to a table. All inserts in a single batch are applied atomically.

This code shows how to write the data:

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

You write data using a [`  Batch  `](/python/docs/reference/spanner/latest/batch-api) object. A `  Batch  ` object is a container for mutation operations. A mutation represents a sequence of inserts, updates, or deletes that can be applied atomically to different rows and tables in a Spanner database.

The [`  insert()  `](/python/docs/reference/spanner/latest/batch-usage#inserting-records-using-a-batch) method in the `  Batch  ` class is used to add one or more insert mutations to the batch. All mutations in a single batch are applied atomically.

This code shows how to write the data:

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

You write data using a [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) object. The [`  Client#commit  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#commit-instance_method) method creates and commits a transaction for writes that execute atomically at a single logical point in time across columns, rows, and tables in a database.

This code shows how to write the data:

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

## Update rows in a table

**Note:** Spanner needs to read the data in its tables to determine whether to write new values. You must use a [read-write transaction](/spanner/docs/transactions#read-write_transactions) to perform the reads and writes atomically.

Suppose that sales of `  Albums(1, 1)  ` are lower than expected. As a result, you want to move $200,000 from the marketing budget of `  Albums(2, 2)  ` to `  Albums(1, 1)  ` , but only if the money is available in the budget of `  Albums(2, 2)  ` .

### C++

Use the `  Transaction()  ` function to run a transaction for a client.

Here's the code to run the transaction:

``` cpp
void ReadWriteTransaction(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  using ::google::cloud::Status;
  using ::google::cloud::StatusCode;
  using ::google::cloud::StatusOr;

  // A helper to read a single album MarketingBudget.
  auto get_current_budget =
      [](spanner::Client client, spanner::Transaction txn,
         std::int64_t singer_id,
         std::int64_t album_id) -> StatusOr<std::int64_t> {
    auto key = spanner::KeySet().AddKey(spanner::MakeKey(singer_id, album_id));
    auto rows = client.Read(std::move(txn), "Albums", std::move(key),
                            {"MarketingBudget"});
    using RowType = std::tuple<std::int64_t>;
    auto row = spanner::GetSingularRow(spanner::StreamOf<RowType>(rows));
    if (!row) return std::move(row).status();
    return std::get<0>(*std::move(row));
  };

  auto constexpr kInsufficientFundsMessage =
      "The second album does not have enough funds to transfer";
  auto commit = client.Commit(
      [&](spanner::Transaction const& txn) -> StatusOr<spanner::Mutations> {
        auto b1 = get_current_budget(client, txn, 1, 1);
        if (!b1) return std::move(b1).status();
        auto b2 = get_current_budget(client, txn, 2, 2);
        if (!b2) return std::move(b2).status();
        std::int64_t transfer_amount = 200000;

        if (*b2 < transfer_amount) {
          return Status(StatusCode::kFailedPrecondition,
                        kInsufficientFundsMessage);
        }

        return spanner::Mutations{
            spanner::UpdateMutationBuilder(
                "Albums", {"SingerId", "AlbumId", "MarketingBudget"})
                .EmplaceRow(1, 1, *b1 + transfer_amount)
                .EmplaceRow(2, 2, *b2 - transfer_amount)
                .Build()};
      });

  if (!commit) throw std::move(commit).status();
  std::cout << "Transfer was successful [spanner_read_write_transaction]\n";
}
```

### C\#

For .NET Standard 2.0 (or .NET 4.5) and newer, you can use the .NET framework's [`  TransactionScope()  `](https://msdn.microsoft.com/en-us/library/system.transactions.transactionscope) to run a transaction. For all supported versions of .NET, you can create a transaction by setting the result of `  SpannerConnection.BeginTransactionAsync  ` as the `  Transaction  ` property of `  SpannerCommand  ` .

Here are the two ways to run the transaction:

### .NET Standard 2.0

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;
using System.Transactions;

public class ReadWriteWithTransactionAsyncSample
{
    public async Task<int> ReadWriteWithTransactionAsync(string projectId, string instanceId, string databaseId)
    {
        // This sample transfers 200,000 from the MarketingBudget
        // field of the second Album to the first Album. Make sure to run
        // the Add Column and Write Data To New Column samples first,
        // in that order.

        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using TransactionScope scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
        decimal transferAmount = 200000;
        decimal secondBudget = 0;
        decimal firstBudget = 0;

        using var connection = new SpannerConnection(connectionString);
        using var cmdLookup1 = connection.CreateSelectCommand("SELECT * FROM Albums WHERE SingerId = 2 AND AlbumId = 2");

        using (var reader = await cmdLookup1.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                // Read the second album's budget.
                secondBudget = reader.GetFieldValue<decimal>("MarketingBudget");
                // Confirm second Album's budget is sufficient and
                // if not raise an exception. Raising an exception
                // will automatically roll back the transaction.
                if (secondBudget < transferAmount)
                {
                    throw new Exception($"The second album's budget {secondBudget} is less than the amount to transfer.");
                }
            }
        }

        // Read the first album's budget.
        using var cmdLookup2 = connection.CreateSelectCommand("SELECT * FROM Albums WHERE SingerId = 1 and AlbumId = 1");
        using (var reader = await cmdLookup2.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                firstBudget = reader.GetFieldValue<decimal>("MarketingBudget");
            }
        }

        // Specify update command parameters.
        using var cmdUpdate = connection.CreateUpdateCommand("Albums", new SpannerParameterCollection
        {
            { "SingerId", SpannerDbType.Int64 },
            { "AlbumId", SpannerDbType.Int64 },
            { "MarketingBudget", SpannerDbType.Int64 },
        });

        // Update second album to remove the transfer amount.
        secondBudget -= transferAmount;
        cmdUpdate.Parameters["SingerId"].Value = 2;
        cmdUpdate.Parameters["AlbumId"].Value = 2;
        cmdUpdate.Parameters["MarketingBudget"].Value = secondBudget;
        var rowCount = await cmdUpdate.ExecuteNonQueryAsync();

        // Update first album to add the transfer amount.
        firstBudget += transferAmount;
        cmdUpdate.Parameters["SingerId"].Value = 1;
        cmdUpdate.Parameters["AlbumId"].Value = 1;
        cmdUpdate.Parameters["MarketingBudget"].Value = firstBudget;
        rowCount += await cmdUpdate.ExecuteNonQueryAsync();
        scope.Complete();
        Console.WriteLine("Transaction complete.");
        return rowCount;
    }
}
```

### .NET Standard 1.5

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class ReadWriteWithTransactionCoreAsyncSample
{
    public async Task<int> ReadWriteWithTransactionCoreAsync(string projectId, string instanceId, string databaseId)
    {
        // This sample transfers 200,000 from the MarketingBudget
        // field of the second Album to the first Album. Make sure to run
        // the Add Column and Write Data To New Column samples first,
        // in that order.
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        decimal transferAmount = 200000;
        decimal secondBudget = 0;
        decimal firstBudget = 0;

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var transaction = await connection.BeginTransactionAsync();

        using var cmdLookup1 = connection.CreateSelectCommand("SELECT * FROM Albums WHERE SingerId = 2 AND AlbumId = 2");
        cmdLookup1.Transaction = transaction;

        using (var reader = await cmdLookup1.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                // Read the second album's budget.
                secondBudget = reader.GetFieldValue<decimal>("MarketingBudget");
                // Confirm second Album's budget is sufficient and
                // if not raise an exception. Raising an exception
                // will automatically roll back the transaction.
                if (secondBudget < transferAmount)
                {
                    throw new Exception($"The second album's budget {secondBudget} contains less than the amount to transfer.");
                }
            }
        }
        // Read the first album's budget.
        using var cmdLookup2 = connection.CreateSelectCommand("SELECT * FROM Albums WHERE SingerId = 1 and AlbumId = 1");
        cmdLookup2.Transaction = transaction;
        using (var reader = await cmdLookup2.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                firstBudget = reader.GetFieldValue<decimal>("MarketingBudget");
            }
        }

        // Specify update command parameters.
        using var cmdUpdate = connection.CreateUpdateCommand("Albums", new SpannerParameterCollection
        {
            { "SingerId", SpannerDbType.Int64 },
            { "AlbumId", SpannerDbType.Int64 },
            { "MarketingBudget", SpannerDbType.Int64 },
        });
        cmdUpdate.Transaction = transaction;

        // Update second album to remove the transfer amount.
        secondBudget -= transferAmount;
        cmdUpdate.Parameters["SingerId"].Value = 2;
        cmdUpdate.Parameters["AlbumId"].Value = 2;
        cmdUpdate.Parameters["MarketingBudget"].Value = secondBudget;
        var rowCount = await cmdUpdate.ExecuteNonQueryAsync();

        // Update first album to add the transfer amount.
        firstBudget += transferAmount;
        cmdUpdate.Parameters["SingerId"].Value = 1;
        cmdUpdate.Parameters["AlbumId"].Value = 1;
        cmdUpdate.Parameters["MarketingBudget"].Value = firstBudget;
        rowCount += await cmdUpdate.ExecuteNonQueryAsync();

        await transaction.CommitAsync();
        Console.WriteLine("Transaction complete.");
        return rowCount;
    }
}
```

### Go

Use the [`  ReadWriteTransaction  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadWriteTransaction) type for executing a body of work in the context of a read-write transaction. [`  Client.ReadWriteTransaction()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client.ReadWriteTransaction) returns a `  ReadWriteTransaction  ` object.

The sample uses [`  ReadWriteTransaction.ReadRow()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadWriteTransaction.ReadRow) to retrieve a row of data.

The sample also uses [`  ReadWriteTransaction.BufferWrite()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadWriteTransaction.BufferWrite) , which adds a list of mutations to the set of updates that will be applied when the transaction is committed.

The sample also uses the [`  Key  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Key) type, which represents a row key in a Spanner table or index.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func writeWithTransaction(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     getBudget := func(key spanner.Key) (int64, error) {
         row, err := txn.ReadRow(ctx, "Albums", key, []string{"MarketingBudget"})
         if err != nil {
             return 0, err
         }
         var budget int64
         if err := row.Column(0, &budget); err != nil {
             return 0, err
         }
         return budget, nil
     }
     album2Budget, err := getBudget(spanner.Key{2, 2})
     if err != nil {
         return err
     }
     const transferAmt = 200000
     if album2Budget >= transferAmt {
         album1Budget, err := getBudget(spanner.Key{1, 1})
         if err != nil {
             return err
         }
         album1Budget += transferAmt
         album2Budget -= transferAmt
         cols := []string{"SingerId", "AlbumId", "MarketingBudget"}
         txn.BufferWrite([]*spanner.Mutation{
             spanner.Update("Albums", cols, []interface{}{1, 1, album1Budget}),
             spanner.Update("Albums", cols, []interface{}{2, 2, album2Budget}),
         })
         fmt.Fprintf(w, "Moved %d from Album2's MarketingBudget to Album1's.", transferAmt)
     }
     return nil
 })
 return err
}
```

### Java

Use the [`  TransactionRunner  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.TransactionRunner) interface for executing a body of work in the context of a read-write transaction. This interface contains the method [`  run()  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.TransactionRunner#com_google_cloud_spanner_TransactionRunner__T_run_com_google_cloud_spanner_TransactionRunner_TransactionCallable_T__) , which is used to execute a read- write transaction, with retries as necessary. The [`  readWriteTransaction  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.DatabaseClient#com_google_cloud_spanner_DatabaseClient_readWriteTransaction_) method of the `  DatabaseClient  ` class returns a `  TransactionRunner  ` object for executing a single logical transaction.

The [`  TransactionRunner.TransactionCallable  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.TransactionRunner.TransactionCallable) class contains a `  run()  ` method for performing a single attempt of a transaction. `  run()  ` takes a [`  TransactionContext  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.TransactionContext) object, which is a context for a transaction.

The sample uses the [`  Struct  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Struct) class, which is handy for storing the results of the `  readRow()  ` calls. The sample also uses the [`  Key  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Key) class, which represents a row key in a Spanner table or index.

Here's the code to run the transaction:

``` java
static void writeWithTransaction(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(transaction -> {
        // Transfer marketing budget from one album to another. We do it in a transaction to
        // ensure that the transfer is atomic.
        Struct row =
            transaction.readRow("Albums", Key.of(2, 2), Arrays.asList("MarketingBudget"));
        long album2Budget = row.getLong(0);
        // Transaction will only be committed if this condition still holds at the time of
        // commit. Otherwise it will be aborted and the callable will be rerun by the
        // client library.
        long transfer = 200000;
        if (album2Budget >= transfer) {
          long album1Budget =
              transaction
                  .readRow("Albums", Key.of(1, 1), Arrays.asList("MarketingBudget"))
                  .getLong(0);
          album1Budget += transfer;
          album2Budget -= transfer;
          transaction.buffer(
              Mutation.newUpdateBuilder("Albums")
                  .set("SingerId")
                  .to(1)
                  .set("AlbumId")
                  .to(1)
                  .set("MarketingBudget")
                  .to(album1Budget)
                  .build());
          transaction.buffer(
              Mutation.newUpdateBuilder("Albums")
                  .set("SingerId")
                  .to(2)
                  .set("AlbumId")
                  .to(2)
                  .set("MarketingBudget")
                  .to(album2Budget)
                  .build());
        }
        return null;
      });
}
```

### Node.js

Use [`  Database.runTransaction()  `](https://googleapis.dev/nodejs/spanner/latest/Database.html#runTransaction) to run a transaction.

Here's the code to run the transaction:

``` javascript
// This sample transfers 200,000 from the MarketingBudget field
// of the second Album to the first Album, as long as the second
// Album has enough money in its budget. Make sure to run the
// addColumn and updateData samples first (in that order).

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

const transferAmount = 200000;

// Note: the `runTransaction()` method is non blocking and returns "void".
// For sequential execution of the transaction use `runTransactionAsync()` method which returns a promise.
// For example: await database.runTransactionAsync(async (err, transaction) => { ... })
database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  let firstBudget, secondBudget;
  const queryOne = {
    columns: ['MarketingBudget'],
    keys: [[2, 2]], // SingerId: 2, AlbumId: 2
  };

  const queryTwo = {
    columns: ['MarketingBudget'],
    keys: [[1, 1]], // SingerId: 1, AlbumId: 1
  };

  Promise.all([
    // Reads the second album's budget
    transaction.read('Albums', queryOne).then(results => {
      // Gets second album's budget
      const rows = results[0].map(row => row.toJSON());
      secondBudget = rows[0].MarketingBudget;
      console.log(`The second album's marketing budget: ${secondBudget}`);

      // Makes sure the second album's budget is large enough
      if (secondBudget < transferAmount) {
        throw new Error(
          `The second album's budget (${secondBudget}) is less than the transfer amount (${transferAmount}).`,
        );
      }
    }),

    // Reads the first album's budget
    transaction.read('Albums', queryTwo).then(results => {
      // Gets first album's budget
      const rows = results[0].map(row => row.toJSON());
      firstBudget = rows[0].MarketingBudget;
      console.log(`The first album's marketing budget: ${firstBudget}`);
    }),
  ])
    .then(() => {
      console.log(firstBudget, secondBudget);
      // Transfers the budgets between the albums
      firstBudget += transferAmount;
      secondBudget -= transferAmount;

      console.log(firstBudget, secondBudget);

      // Updates the database
      // Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
      // must be converted (back) to strings before being inserted as INT64s.
      transaction.update('Albums', [
        {
          SingerId: '1',
          AlbumId: '1',
          MarketingBudget: firstBudget.toString(),
        },
        {
          SingerId: '2',
          AlbumId: '2',
          MarketingBudget: secondBudget.toString(),
        },
      ]);
    })
    .then(() => {
      // Commits the transaction and send the changes to the database
      return transaction.commit();
    })
    .then(() => {
      console.log(
        `Successfully executed read-write transaction to transfer ${transferAmount} from Album 2 to Album 1.`,
      );
    })
    .catch(err => {
      console.error('ERROR:', err);
    })
    .then(() => {
      transaction.end();
      // Closes the database when finished
      return database.close();
    });
});
```

### PHP

Use [`  Database::runTransaction  `](/php/docs/reference/cloud-spanner/latest/database?method=runTransaction) to run a transaction.

Here's the code to run the transaction:

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;
use UnexpectedValueException;

/**
 * Performs a read-write transaction to update two sample records in the
 * database.
 *
 * This will transfer 200,000 from the `MarketingBudget` field for the second
 * Album to the first Album. If the `MarketingBudget` for the second Album is
 * too low, it will raise an exception.
 *
 * Before running this sample, you will need to run the `update_data` sample
 * to populate the fields.
 * Example:
 * ```
 * read_write_transaction($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_write_transaction(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) use ($spanner) {
        $transferAmount = 200000;

        // Read the second album's budget.
        $secondAlbumKey = [2, 2];
        $secondAlbumKeySet = $spanner->keySet(['keys' => [$secondAlbumKey]]);
        $secondAlbumResult = $t->read(
            'Albums',
            $secondAlbumKeySet,
            ['MarketingBudget'],
            ['limit' => 1]
        );

        $firstRow = $secondAlbumResult->rows()->current();
        $secondAlbumBudget = $firstRow['MarketingBudget'];
        if ($secondAlbumBudget < $transferAmount) {
            // Throwing an exception will automatically roll back the transaction.
            throw new UnexpectedValueException(
                'The second album\'s budget is lower than the transfer amount: ' . $transferAmount
            );
        }

        $firstAlbumKey = [1, 1];
        $firstAlbumKeySet = $spanner->keySet(['keys' => [$firstAlbumKey]]);
        $firstAlbumResult = $t->read(
            'Albums',
            $firstAlbumKeySet,
            ['MarketingBudget'],
            ['limit' => 1]
        );

        // Read the first album's budget.
        $firstRow = $firstAlbumResult->rows()->current();
        $firstAlbumBudget = $firstRow['MarketingBudget'];

        // Update the budgets.
        $secondAlbumBudget -= $transferAmount;
        $firstAlbumBudget += $transferAmount;
        printf('Setting first album\'s budget to %s and the second album\'s ' .
            'budget to %s.' . PHP_EOL, $firstAlbumBudget, $secondAlbumBudget);

        // Update the rows.
        $t->updateBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'MarketingBudget' => $firstAlbumBudget],
            ['SingerId' => 2, 'AlbumId' => 2, 'MarketingBudget' => $secondAlbumBudget],
        ]);

        // Commit the transaction!
        $t->commit();

        print('Transaction complete.' . PHP_EOL);
    });
}
````

### Python

Use the [`  run_in_transaction()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.run_in_transaction) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class to run a transaction.

Here's the code to run the transaction:

``` python
def read_write_transaction(instance_id, database_id):
    """Performs a read-write transaction to update two sample records in the
    database.

    This will transfer 200,000 from the `MarketingBudget` field for the second
    Album to the first Album. If the `MarketingBudget` is too low, it will
    raise an exception.

    Before running this sample, you will need to run the `update_data` sample
    to populate the fields.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    def update_albums(transaction):
        # Read the second album budget.
        second_album_keyset = spanner.KeySet(keys=[(2, 2)])
        second_album_result = transaction.read(
            table="Albums",
            columns=("MarketingBudget",),
            keyset=second_album_keyset,
            limit=1,
        )
        second_album_row = list(second_album_result)[0]
        second_album_budget = second_album_row[0]

        transfer_amount = 200000

        if second_album_budget < transfer_amount:
            # Raising an exception will automatically roll back the
            # transaction.
            raise ValueError("The second album doesn't have enough funds to transfer")

        # Read the first album's budget.
        first_album_keyset = spanner.KeySet(keys=[(1, 1)])
        first_album_result = transaction.read(
            table="Albums",
            columns=("MarketingBudget",),
            keyset=first_album_keyset,
            limit=1,
        )
        first_album_row = list(first_album_result)[0]
        first_album_budget = first_album_row[0]

        # Update the budgets.
        second_album_budget -= transfer_amount
        first_album_budget += transfer_amount
        print(
            "Setting first album's budget to {} and the second album's "
            "budget to {}.".format(first_album_budget, second_album_budget)
        )

        # Update the rows.
        transaction.update(
            table="Albums",
            columns=("SingerId", "AlbumId", "MarketingBudget"),
            values=[(1, 1, first_album_budget), (2, 2, second_album_budget)],
        )

    database.run_in_transaction(update_albums)

    print("Transaction complete.")
```

### Ruby

Use the [`  transaction  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#transaction-instance_method) method of the [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) class to run a transaction.

Here's the code to run the transaction:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner         = Google::Cloud::Spanner.new project: project_id
client          = spanner.client instance_id, database_id
transfer_amount = 200_000

client.transaction do |transaction|
  first_album  = transaction.read("Albums", [:MarketingBudget], keys: [[1, 1]]).rows.first
  second_album = transaction.read("Albums", [:MarketingBudget], keys: [[2, 2]]).rows.first

  raise "The second album does not have enough funds to transfer" if second_album[:MarketingBudget] < transfer_amount

  new_first_album_budget  = first_album[:MarketingBudget] + transfer_amount
  new_second_album_budget = second_album[:MarketingBudget] - transfer_amount

  transaction.update "Albums", [
    { SingerId: 1, AlbumId: 1, MarketingBudget: new_first_album_budget  },
    { SingerId: 2, AlbumId: 2, MarketingBudget: new_second_album_budget }
  ]
end

puts "Transaction complete"
```

## Delete rows in a table

Each client library provides multiple ways to delete rows:

  - Delete all the rows in a table.
  - Delete a single row by specifying the key column values for the row.
  - Delete a group of rows by creating a key range.
  - Delete rows in an interleaved table by deleting the parent rows, if the interleaved table includes `  ON DELETE CASCADE  ` in its schema definition.

**Note:** The limit of mutations per commit is 80,000. Each secondary index on a table is an additional mutation per row. For example, on a table with one secondary index, you can delete up to 40,000 rows in a commit. To delete a large amount of data, use [Partitioned DML](/spanner/docs/dml-partitioned) . Partitioned DML handles transaction limits and is optimized to handle large-scale deletions.

### C++

Delete rows using the `  DeleteMutationBuilder()  ` function for a client.

This code shows how to delete the data:

``` cpp
void DeleteData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  // Delete the albums with key (2,1) and (2,3).
  auto delete_albums = spanner::DeleteMutationBuilder(
                           "Albums", spanner::KeySet()
                                         .AddKey(spanner::MakeKey(2, 1))
                                         .AddKey(spanner::MakeKey(2, 3)))
                           .Build();

  // Delete some singers using the keys in the range [3, 5]
  auto delete_singers_range =
      spanner::DeleteMutationBuilder(
          "Singers", spanner::KeySet().AddRange(spanner::MakeKeyBoundClosed(3),
                                                spanner::MakeKeyBoundOpen(5)))
          .Build();

  // Deletes remaining rows from the Singers table and the Albums table, because
  // the Albums table is defined with ON DELETE CASCADE.
  auto delete_singers_all =
      spanner::MakeDeleteMutation("Singers", spanner::KeySet::All());

  auto commit_result = client.Commit(spanner::Mutations{
      delete_albums, delete_singers_range, delete_singers_all});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Delete was successful [spanner_delete_data]\n";
}
```

### C\#

Delete rows using the [`  connection.CreateDeleteCommand()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_CreateDeleteCommand_System_String_Google_Cloud_Spanner_Data_SpannerParameterCollection_) method, which creates a new `  SpannerCommand  ` to delete rows. The [`  SpannerCommand.ExecuteNonQueryAsync()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerCommand#Google_Cloud_Spanner_Data_SpannerCommand_ExecuteNonQueryAsync_System_Threading_CancellationToken_) method deletes the rows from the table.

This example deletes the rows in the `  Singers  ` table individually. The rows in the `  Albums  ` table are deleted because the `  Albums  ` table is interleaved in the `  Singers  ` table and is defined with `  ON DELETE CASCADE  ` .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class DeleteDataAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<int> DeleteDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>
        {
            new Album { SingerId = 2, AlbumId = 1, AlbumTitle = "Green" },
            new Album { SingerId = 2, AlbumId = 3, AlbumTitle = "Terrified" },
        };

        int rowCount = 0;
        using (var connection = new SpannerConnection(connectionString))
        {
            await connection.OpenAsync();

            // Delete individual rows from the Albums table.
            await Task.WhenAll(albums.Select(async album =>
            {
                var cmd = connection.CreateDeleteCommand("Albums", new SpannerParameterCollection
                {
                    { "SingerId", SpannerDbType.Int64, album.SingerId },
                    { "AlbumId", SpannerDbType.Int64, album.AlbumId }
                });
                rowCount += await cmd.ExecuteNonQueryAsync();
            }));
            Console.WriteLine("Deleted individual rows in Albums.");

            // Delete a range of rows from the Singers table where the column key is >=3 and <5.
            var cmd = connection.CreateDmlCommand("DELETE FROM Singers WHERE SingerId >= 3 AND SingerId < 5");
            rowCount += await cmd.ExecuteNonQueryAsync();
            Console.WriteLine($"{rowCount} row(s) deleted from Singers.");

            // Delete remaining Singers rows, which will also delete the remaining
            // Albums rows since it was defined with ON DELETE CASCADE.
            cmd = connection.CreateDmlCommand("DELETE FROM Singers WHERE true");
            rowCount += await cmd.ExecuteNonQueryAsync();
            Console.WriteLine($"{rowCount} row(s) deleted from Singers.");
        }
        return rowCount;
    }
}
```

### Go

Delete rows using a [`  Mutation  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Mutation) . Use the [`  Mutation.Delete()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Delete) method to construct a `  DELETE  ` mutation, which deletes a row. The [`  Client.Apply()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client.Apply) method applies mutations atomically to the database.

This example deletes the rows in the `  Albums  ` table individually, and then deletes all the rows in the `  Singers  ` table using a [KeyRange](https://pkg.go.dev/cloud.google.com/go/spanner/#KeyRange) .

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func delete(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 m := []*spanner.Mutation{
     // spanner.Key can be used to delete a specific set of rows.
     // Delete the Albums with the key values (2,1) and (2,3).
     spanner.Delete("Albums", spanner.Key{2, 1}),
     spanner.Delete("Albums", spanner.Key{2, 3}),
     // spanner.KeyRange can be used to delete rows with a key in a specific range.
     // Delete a range of rows where the column key is >=3 and <5
     spanner.Delete("Singers", spanner.KeyRange{Start: spanner.Key{3}, End: spanner.Key{5}, Kind: spanner.ClosedOpen}),
     // spanner.AllKeys can be used to delete all the rows in a table.
     // Delete remaining Singers rows, which will also delete the remaining Albums rows since it was
     // defined with ON DELETE CASCADE.
     spanner.Delete("Singers", spanner.AllKeys()),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

Delete rows using the [`  Mutation.delete()  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation#com_google_cloud_spanner_Mutation_delete_java_lang_String_com_google_cloud_spanner_Key_) method.

This examples uses the [`  KeySet.all()  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.KeySet#com_google_cloud_spanner_KeySet_all__) method to delete all the rows in the `  Albums  ` table. After deleting the rows in the `  Albums  ` table, the example deletes the rows in the `  Singers  ` table individually using keys created with the [`  KeySet.singleKey()  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.KeySet#com_google_cloud_spanner_KeySet_singleKey_com_google_cloud_spanner_Key_) method.

``` java
static void deleteExampleData(DatabaseClient dbClient) {
  List<Mutation> mutations = new ArrayList<>();

  // KeySet.Builder can be used to delete a specific set of rows.
  // Delete the Albums with the key values (2,1) and (2,3).
  mutations.add(
      Mutation.delete(
          "Albums", KeySet.newBuilder().addKey(Key.of(2, 1)).addKey(Key.of(2, 3)).build()));

  // KeyRange can be used to delete rows with a key in a specific range.
  // Delete a range of rows where the column key is >=3 and <5
  mutations.add(
      Mutation.delete("Singers", KeySet.range(KeyRange.closedOpen(Key.of(3), Key.of(5)))));

  // KeySet.all() can be used to delete all the rows in a table.
  // Delete remaining Singers rows, which will also delete the remaining Albums rows since it was
  // defined with ON DELETE CASCADE.
  mutations.add(Mutation.delete("Singers", KeySet.all()));

  dbClient.write(mutations);
  System.out.printf("Records deleted.\n");
}
```

### Node.js

Delete rows using the [`  table.deleteRows()  `](https://googleapis.dev/nodejs/spanner/latest/Table.html#deleteRows) method.

This example uses the `  table.deleteRows()  ` method to delete all the rows from the `  Singers  ` table. The rows in the `  Albums  ` table are deleted because the `  Albums  ` table is interleaved in `  Singers  ` table and is defined with `  ON DELETE CASCADE  ` .

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

// Instantiate Spanner table object
const albumsTable = database.table('Albums');

// Deletes individual rows from the Albums table.
try {
  const keys = [
    [2, 1],
    [2, 3],
  ];
  await albumsTable.deleteRows(keys);
  console.log('Deleted individual rows in Albums.');
} catch (err) {
  console.error('ERROR:', err);
}

// Delete a range of rows where the column key is >=3 and <5
database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  try {
    const [rowCount] = await transaction.runUpdate({
      sql: 'DELETE FROM Singers WHERE SingerId >= 3 AND SingerId < 5',
    });
    console.log(`${rowCount} records deleted from Singers.`);
  } catch (err) {
    console.error('ERROR:', err);
  }

  // Deletes remaining rows from the Singers table and the Albums table,
  // because Albums table is defined with ON DELETE CASCADE.
  try {
    // The WHERE clause is required for DELETE statements to prevent
    // accidentally deleting all rows in a table.
    // https://cloud.google.com/spanner/docs/dml-syntax#where_clause
    const [rowCount] = await transaction.runUpdate({
      sql: 'DELETE FROM Singers WHERE true',
    });
    console.log(`${rowCount} records deleted from Singers.`);
    await transaction.commit();
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    await database.close();
  }
});
```

### PHP

Delete rows using the [`  Database::delete() method  `](/php/docs/reference/cloud-spanner/latest/database?method=delete) . The [`  Database::delete()  ` method](/php/docs/reference/cloud-spanner/latest/database?method=delete) page includes an example.

### Python

Delete rows using the [`  Batch.delete()  `](/python/docs/reference/spanner/latest/batch-usage#delete-records-using-a-batch) method.

This example deletes all the rows in the `  Albums  ` and `  Singers  ` tables individually using a [`  KeySet  `](/python/docs/reference/spanner/latest/keyset-api) object.

``` python
def delete_data(instance_id, database_id):
    """Deletes sample data from the given database.

    The database, table, and data must already exist and can be created using
    `create_database` and `insert_data`.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    # Delete individual rows
    albums_to_delete = spanner.KeySet(keys=[[2, 1], [2, 3]])

    # Delete a range of rows where the column key is >=3 and <5
    singers_range = spanner.KeyRange(start_closed=[3], end_open=[5])
    singers_to_delete = spanner.KeySet(ranges=[singers_range])

    # Delete remaining Singers rows, which will also delete the remaining
    # Albums rows because Albums was defined with ON DELETE CASCADE
    remaining_singers = spanner.KeySet(all_=True)

    with database.batch() as batch:
        batch.delete("Albums", albums_to_delete)
        batch.delete("Singers", singers_to_delete)
        batch.delete("Singers", remaining_singers)

    print("Deleted data.")
```

### Ruby

Delete rows using the [`  Client#delete  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#delete-instance_method) method. The [`  Client#delete  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#delete-instance_method) page includes an example.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

# Delete individual rows
client.delete "Albums", [[2, 1], [2, 3]]

# Delete a range of rows where the column key is >=3 and <5
key_range = client.range 3, 5, exclude_end: true
client.delete "Singers", key_range

# Delete remaining Singers rows, which will also delete the remaining
# Albums rows because Albums was defined with ON DELETE CASCADE
client.delete "Singers"
```
