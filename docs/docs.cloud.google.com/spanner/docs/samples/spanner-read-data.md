Read data.

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

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void ReadData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  auto rows = client.Read("Albums", google::cloud::spanner::KeySet::All(),
                          {"SingerId", "AlbumId", "AlbumTitle"});
  using RowType = std::tuple<std::int64_t, std::int64_t, std::string>;
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\t";
    std::cout << "AlbumId: " << std::get<1>(*row) << "\t";
    std::cout << "AlbumTitle: " << std::get<2>(*row) << "\n";
  }

  std::cout << "Read completed for [spanner_read_data]\n";
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

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

client.read("Albums", [:SingerId, :AlbumId, :AlbumTitle]).rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:AlbumTitle]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
