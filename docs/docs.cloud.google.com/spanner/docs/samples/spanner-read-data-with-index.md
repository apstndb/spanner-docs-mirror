Read data by using an index.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Getting started with Spanner in C++](/spanner/docs/getting-started/cpp)
  - [Getting started with Spanner in Go](/spanner/docs/getting-started/go)
  - [Getting started with Spanner in Java](/spanner/docs/getting-started/java)
  - [Getting started with Spanner in Node.js](/spanner/docs/getting-started/nodejs)
  - [Getting started with Spanner in PHP](/spanner/docs/getting-started/php)
  - [Getting started with Spanner in Python](/spanner/docs/getting-started/python)
  - [Getting started with Spanner in Ruby](/spanner/docs/getting-started/ruby)
  - [Reads outside of transactions](/spanner/docs/reads)
  - [Secondary indexes](/spanner/docs/secondary-indexes)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
