Query data from a table containing a commit TIMESTAMP column.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Commit timestamps in GoogleSQL-dialect databases](/spanner/docs/commit-timestamp)
  - [Commit timestamps in PostgreSQL-dialect databases](/spanner/docs/commit-timestamp-postgresql)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
