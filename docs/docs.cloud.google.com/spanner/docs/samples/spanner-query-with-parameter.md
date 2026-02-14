Query data by using a parameter.

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

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithParameter(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  spanner::SqlStatement select(
      "SELECT SingerId, FirstName, LastName FROM Singers"
      " WHERE LastName = @last_name",
      {{"last_name", spanner::Value("Garcia")}});
  using RowType = std::tuple<std::int64_t, std::string, std::string>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\t";
    std::cout << "FirstName: " << std::get<1>(*row) << "\t";
    std::cout << "LastName: " << std::get<2>(*row) << "\n";
  }

  std::cout << "Query completed for [spanner_query_with_parameter]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryWithParameterAsyncSample
{
    public class Singer
    {
        public int SingerId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public async Task<List<Singer>> QueryWithParameterAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            $"SELECT SingerId, FirstName, LastName FROM Singers WHERE LastName = @lastName",
            new SpannerParameterCollection { { "lastName", SpannerDbType.String, "Garcia" } });

        var singers = new List<Singer>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            singers.Add(new Singer
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                FirstName = reader.GetFieldValue<string>("FirstName"),
                LastName = reader.GetFieldValue<string>("LastName")
            });
        }
        return singers;
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

func queryWithParameter(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{
     SQL: `SELECT SingerId, FirstName, LastName FROM Singers
         WHERE LastName = @lastName`,
     Params: map[string]interface{}{
         "lastName": "Garcia",
     },
 }
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
     var singerID int64
     var firstName, lastName string
     if err := row.Columns(&singerID, &firstName, &lastName); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %s\n", singerID, firstName, lastName)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryWithParameter(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    try (PreparedStatement statement =
        connection.prepareStatement(
            "SELECT SingerId, FirstName, LastName "
                + "FROM Singers "
                + "WHERE LastName = ?")) {
      statement.setString(1, "Garcia");
      try (ResultSet resultSet = statement.executeQuery()) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %s %s\n",
              resultSet.getLong("SingerId"),
              resultSet.getString("FirstName"),
              resultSet.getString("LastName"));
        }
      }
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

const query = {
  sql: `SELECT SingerId, FirstName, LastName
        FROM Singers WHERE LastName = @lastName`,
  params: {
    lastName: 'Garcia',
  },
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `SingerId: ${json.SingerId}, FirstName: ${json.FirstName}, LastName: ${json.LastName}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL with a parameter.
 * Example:
 * ```
 * query_data_with_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT SingerId, FirstName, LastName FROM Singers ' .
        'WHERE LastName = @lastName',
        ['parameters' => ['lastName' => 'Garcia']]
    );

    foreach ($results as $row) {
        printf('SingerId: %s, FirstName: %s, LastName: %s' . PHP_EOL,
            $row['SingerId'], $row['FirstName'], $row['LastName']);
    }
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT SingerId, FirstName, LastName FROM Singers "
        "WHERE LastName = @lastName",
        params={"lastName": "Garcia"},
        param_types={"lastName": spanner.param_types.STRING},
    )

    for row in results:
        print("SingerId: {}, FirstName: {}, LastName: {}".format(*row))
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

sql_query = "SELECT SingerId, FirstName, LastName
             FROM Singers
             WHERE LastName = @lastName"

params      = { lastName: "Garcia" }
param_types = { lastName: :STRING }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:SingerId]} #{row[:FirstName]} #{row[:LastName]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
