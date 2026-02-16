Query data by using an index.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Secondary indexes](/spanner/docs/secondary-indexes)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryUsingIndex(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  spanner::SqlStatement select(
      "SELECT AlbumId, AlbumTitle, MarketingBudget"
      " FROM Albums@{FORCE_INDEX=AlbumsByAlbumTitle}"
      " WHERE AlbumTitle >= @start_title AND AlbumTitle < @end_title",
      {{"start_title", spanner::Value("Aardvark")},
       {"end_title", spanner::Value("Goo")}});
  using RowType =
      std::tuple<std::int64_t, std::string, absl::optional<std::int64_t>>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "AlbumId: " << std::get<0>(*row) << "\t";
    std::cout << "AlbumTitle: " << std::get<1>(*row) << "\t";
    auto marketing_budget = std::get<2>(*row);
    if (marketing_budget) {
      std::cout << "MarketingBudget: " << *marketing_budget << "\n";
    } else {
      std::cout << "MarketingBudget: NULL\n";
    }
  }
  std::cout << "Read completed for [spanner_query_data_with_index]\n";
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
 "strconv"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func queryUsingIndex(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{
     SQL: `SELECT AlbumId, AlbumTitle, MarketingBudget
         FROM Albums@{FORCE_INDEX=AlbumsByAlbumTitle}
         WHERE AlbumTitle >= @start_title AND AlbumTitle < @end_title`,
     Params: map[string]interface{}{
         "start_title": "Aardvark",
         "end_title":   "Goo",
     },
 }
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     var albumID int64
     var marketingBudget spanner.NullInt64
     var albumTitle string
     if err := row.ColumnByName("AlbumId", &albumID); err != nil {
         return err
     }
     if err := row.ColumnByName("AlbumTitle", &albumTitle); err != nil {
         return err
     }
     if err := row.ColumnByName("MarketingBudget", &marketingBudget); err != nil {
         return err
     }
     budget := "NULL"
     if marketingBudget.Valid {
         budget = strconv.FormatInt(marketingBudget.Int64, 10)
     }
     fmt.Fprintf(w, "%d %s %s\n", albumID, albumTitle, budget)
 }
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryUsingIndex(DatabaseClient dbClient) {
  Statement statement =
      Statement
          // We use FORCE_INDEX hint to specify which index to use. For more details see
          // https://cloud.google.com/spanner/docs/query-syntax#from-clause
          .newBuilder(
              "SELECT AlbumId, AlbumTitle, MarketingBudget "
                  + "FROM Albums@{FORCE_INDEX=AlbumsByAlbumTitle} "
                  + "WHERE AlbumTitle >= @StartTitle AND AlbumTitle < @EndTitle")
          // We use @BoundParameters to help speed up frequently executed queries.
          //  For more details see https://cloud.google.com/spanner/docs/sql-best-practices
          .bind("StartTitle")
          .to("Aardvark")
          .bind("EndTitle")
          .to("Goo")
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(statement)) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %s %s\n",
          resultSet.getLong("AlbumId"),
          resultSet.getString("AlbumTitle"),
          resultSet.isNull("MarketingBudget") ? "NULL" : resultSet.getLong("MarketingBudget"));
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
// const startTitle = 'Ardvark';
// const endTitle = 'Goo';

// Imports the Google Cloud Spanner client library
const {Spanner} = require('@google-cloud/spanner');

// Instantiates a client
const spanner = new Spanner({
  projectId: projectId,
});

async function queryDataWithIndex() {
  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  const query = {
    sql: `SELECT AlbumId, AlbumTitle, MarketingBudget
                FROM Albums@{FORCE_INDEX=AlbumsByAlbumTitle}
                WHERE AlbumTitle >= @startTitle AND AlbumTitle <= @endTitle`,
    params: {
      startTitle: startTitle,
      endTitle: endTitle,
    },
  };

  // Queries rows from the Albums table
  try {
    const [rows] = await database.run(query);

    rows.forEach(row => {
      const json = row.toJSON();
      const marketingBudget = json.MarketingBudget
        ? json.MarketingBudget
        : null; // This value is nullable
      console.log(
        `AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}, MarketingBudget: ${marketingBudget}`,
      );
    });
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
}
queryDataWithIndex();
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL and an index.
 *
 * The index must exist before running this sample. You can add the index
 * by running the `add_index` sample or by running this DDL statement against
 * your database:
 *
 *     CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)
 *
 * Example:
 * ```
 * query_data_with_index($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $startTitle The start of the title index.
 * @param string $endTitle   The end of the title index.
 */
function query_data_with_index(
    string $instanceId,
    string $databaseId,
    string $startTitle = 'Aardvark',
    string $endTitle = 'Goo'
): void {
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $parameters = [
        'startTitle' => $startTitle,
        'endTitle' => $endTitle
    ];

    $results = $database->execute(
        'SELECT AlbumId, AlbumTitle, MarketingBudget ' .
        'FROM Albums@{FORCE_INDEX=AlbumsByAlbumTitle} ' .
        'WHERE AlbumTitle >= @startTitle AND AlbumTitle < @endTitle',
        ['parameters' => $parameters]
    );

    foreach ($results as $row) {
        printf('AlbumId: %s, AlbumTitle: %s, MarketingBudget: %d' . PHP_EOL,
            $row['AlbumId'], $row['AlbumTitle'], $row['MarketingBudget']);
    }
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def query_data_with_index(
    instance_id, database_id, start_title="Aardvark", end_title="Goo"
):
    """Queries sample data from the database using SQL and an index.

    The index must exist before running this sample. You can add the index
    by running the `add_index` sample or by running this DDL statement against
    your database:

        CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)

    This sample also uses the `MarketingBudget` column. You can add the column
    by running the `add_column` sample or by running this DDL statement against
    your database:

        ALTER TABLE Albums ADD COLUMN MarketingBudget INT64

    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    params = {"start_title": start_title, "end_title": end_title}
    param_types = {
        "start_title": spanner.param_types.STRING,
        "end_title": spanner.param_types.STRING,
    }

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT AlbumId, AlbumTitle, MarketingBudget "
            "FROM Albums@{FORCE_INDEX=AlbumsByAlbumTitle} "
            "WHERE AlbumTitle >= @start_title AND AlbumTitle < @end_title",
            params=params,
            param_types=param_types,
        )

        for row in results:
            print("AlbumId: {}, AlbumTitle: {}, " "MarketingBudget: {}".format(*row))
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"
# start_title = "An album title to start with such as 'Ardvark'"
# end_title   = "An album title to end with such as 'Goo'"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

sql_query = "SELECT AlbumId, AlbumTitle, MarketingBudget
             FROM Albums@{FORCE_INDEX=AlbumsByAlbumTitle}
             WHERE AlbumTitle >= @start_title AND AlbumTitle < @end_title"

params      = { start_title: start_title, end_title: end_title }
param_types = { start_title: :STRING,     end_title: :STRING }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:MarketingBudget]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
