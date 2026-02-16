Write data by using DML.

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
void DmlGettingStartedInsert(google::cloud::spanner::Client client) {
  using ::google::cloud::StatusOr;
  namespace spanner = ::google::cloud::spanner;

  auto commit_result = client.Commit(
      [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto insert = client.ExecuteDml(
            std::move(txn),
            spanner::SqlStatement(
                "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES"
                " (12, 'Melissa', 'Garcia'),"
                " (13, 'Russell', 'Morales'),"
                " (14, 'Jacqueline', 'Long'),"
                " (15, 'Dylan', 'Shaw')"));
        if (!insert) return std::move(insert).status();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Insert was successful [spanner_dml_getting_started_insert]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class WriteUsingDmlCoreAsyncSample
{
    public async Task<int> WriteUsingDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        SpannerCommand cmd = connection.CreateDmlCommand(
            "INSERT Singers (SingerId, FirstName, LastName) VALUES "
               + "(12, 'Melissa', 'Garcia'), "
               + "(13, 'Russell', 'Morales'), "
               + "(14, 'Jacqueline', 'Long'), "
               + "(15, 'Dylan', 'Shaw')");
        int rowCount = await cmd.ExecuteNonQueryAsync();

        Console.WriteLine($"{rowCount} row(s) inserted...");
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
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func writeUsingDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT Singers (SingerId, FirstName, LastName) VALUES
             (12, 'Melissa', 'Garcia'),
             (13, 'Russell', 'Morales'),
             (14, 'Jacqueline', 'Long'),
             (15, 'Dylan', 'Shaw')`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
     return err
 })
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void writeDataWithDml(
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
    // Add 4 rows in one statement.
    // JDBC always uses '?' as a parameter placeholder.
    try (PreparedStatement preparedStatement =
        connection.prepareStatement(
            "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES "
                + "(?, ?, ?), "
                + "(?, ?, ?), "
                + "(?, ?, ?), "
                + "(?, ?, ?)")) {

      final ImmutableList<Singer> singers =
          ImmutableList.of(
              new Singer(/* SingerId = */ 12L, "Melissa", "Garcia"),
              new Singer(/* SingerId = */ 13L, "Russel", "Morales"),
              new Singer(/* SingerId = */ 14L, "Jacqueline", "Long"),
              new Singer(/* SingerId = */ 15L, "Dylan", "Shaw"));

      // Note that JDBC parameters start at index 1.
      int paramIndex = 0;
      for (Singer singer : singers) {
        preparedStatement.setLong(++paramIndex, singer.singerId);
        preparedStatement.setString(++paramIndex, singer.firstName);
        preparedStatement.setString(++paramIndex, singer.lastName);
      }

      int updateCount = preparedStatement.executeUpdate();
      System.out.printf("%d records inserted.\n", updateCount);
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

database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  try {
    const [rowCount] = await transaction.runUpdate({
      sql: `INSERT Singers (SingerId, FirstName, LastName) VALUES
      (12, 'Melissa', 'Garcia'),
      (13, 'Russell', 'Morales'),
      (14, 'Jacqueline', 'Long'),
      (15, 'Dylan', 'Shaw')`,
    });
    console.log(`${rowCount} records inserted.`);
    await transaction.commit();
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
});
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;

/**
 * Inserts sample data into the given database with a DML statement.
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
function write_data_with_dml(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) {
        $rowCount = $t->executeUpdate(
            'INSERT Singers (SingerId, FirstName, LastName) VALUES '
            . "(12, 'Melissa', 'Garcia'), "
            . "(13, 'Russell', 'Morales'), "
            . "(14, 'Jacqueline', 'Long'), "
            . "(15, 'Dylan', 'Shaw')");
        $t->commit();
        printf('Inserted %d row(s).' . PHP_EOL, $rowCount);
    });
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

def insert_singers(transaction):
    row_ct = transaction.execute_update(
        "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES "
        "(12, 'Melissa', 'Garcia'), "
        "(13, 'Russell', 'Morales'), "
        "(14, 'Jacqueline', 'Long'), "
        "(15, 'Dylan', 'Shaw')"
    )
    print("{} record(s) inserted.".format(row_ct))

database.run_in_transaction(insert_singers)
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
row_count = 0

client.transaction do |transaction|
  row_count = transaction.execute_update(
    "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES
     (12, 'Melissa', 'Garcia'),
     (13, 'Russell', 'Morales'),
     (14, 'Jacqueline', 'Long'),
     (15, 'Dylan', 'Shaw'),
     (16, 'Billie', 'Eillish'),
     (17, 'Judy', 'Garland'),
     (18, 'Taylor', 'Swift'),
     (19, 'Miley', 'Cyrus'),
     (20, 'Michael', 'Jackson'),
     (21, 'Ariana', 'Grande'),
     (22, 'Elvis', 'Presley'),
     (23, 'Kanye', 'West'),
     (24, 'Lady', 'Gaga'),
     (25, 'Nick', 'Jonas')"
  )
end

puts "#{row_count} records inserted."
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
