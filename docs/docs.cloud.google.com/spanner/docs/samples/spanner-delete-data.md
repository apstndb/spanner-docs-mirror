Delete individual rows from a table.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Insert, update, and delete data using mutations](/spanner/docs/modify-mutation-api)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void DeleteData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  // Delete the albums with key (2,1) and (2,3).
  //! [make-key] [keyset-add-key]
  auto delete_albums = spanner::DeleteMutationBuilder(
                           "Albums", spanner::KeySet()
                                         .AddKey(spanner::MakeKey(2, 1))
                                         .AddKey(spanner::MakeKey(2, 3)))
                           .Build();
  //! [make-key] [keyset-add-key]

  // Delete some singers using the keys in the range [3, 5]
  //! [make-keybound-closed]
  auto delete_singers_range =
      spanner::DeleteMutationBuilder(
          "Singers", spanner::KeySet().AddRange(spanner::MakeKeyBoundClosed(3),
                                                spanner::MakeKeyBoundOpen(5)))
          .Build();
  //! [make-keybound-closed]

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\KeyRange;
use Google\Cloud\Core\Exception\GoogleException;

/**
 * Deletes sample data from the given database.
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @throws GoogleException
 */
function delete_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    // Delete individual rows
    $albumsToDelete = $spanner->keySet([
        'keys' => [[2, 1], [2, 3]]
    ]);
    $database->delete('Albums', $albumsToDelete);

    // Delete a range of rows where the column key is >=3 and <5
    // NOTE: A KeyRange must include a start and end.
    // NOTE: startType and endType both default to KeyRange::TYPE_OPEN.
    $singersRange = $spanner->keyRange([
        'startType' => KeyRange::TYPE_CLOSED,
        'start' => [3],
        'endType' => KeyRange::TYPE_OPEN,
        'end' => [5]
    ]);
    $singersToDelete = $spanner->keySet([
        'ranges' => [$singersRange]
    ]);
    $database->delete('Singers', $singersToDelete);

    // Delete remaining Singers rows, which will also delete the remaining
    // Albums rows because Albums was defined with ON DELETE CASCADE
    $remainingSingers = $spanner->keySet([
        'all' => true
    ]);
    $database->delete('Singers', $remainingSingers);

    print('Deleted data.' . PHP_EOL);
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
