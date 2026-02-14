Call multiple SQL statements in a single transaction by using batch DML.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Insert, update, and delete data using data manipulation language (DML)](/spanner/docs/dml-tasks)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void DmlBatchUpdate(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  auto commit_result =
      client.Commit([&client](spanner::Transaction const& txn)
                        -> google::cloud::StatusOr<spanner::Mutations> {
        std::vector<spanner::SqlStatement> statements = {
            spanner::SqlStatement("INSERT INTO Albums"
                                  " (SingerId, AlbumId, AlbumTitle,"
                                  " MarketingBudget)"
                                  " VALUES (1, 3, 'Test Album Title', 10000)"),
            spanner::SqlStatement("UPDATE Albums"
                                  " SET MarketingBudget = MarketingBudget * 2"
                                  "  WHERE SingerId = 1 and AlbumId = 3")};
        auto result = client.ExecuteBatchDml(txn, statements);
        if (!result) return std::move(result).status();
        // Note: This mutator might be re-run, or its effects discarded, so
        // changing non-transactional state (e.g., by producing output) is,
        // in general, not something to be imitated.
        for (std::size_t i = 0; i < result->stats.size(); ++i) {
          std::cout << result->stats[i].row_count << " rows affected"
                    << " for the statement " << (i + 1) << ".\n";
        }
        // Batch operations may have partial failures, in which case
        // ExecuteBatchDml returns with success, but the application should
        // verify that all statements completed successfully
        if (!result->status.ok()) return result->status;
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Update was successful [spanner_dml_batch_update]\n";
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

public class UpdateUsingBatchDmlCoreAsyncSample
{
    public async Task<int> UpdateUsingBatchDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        SpannerBatchCommand cmd = connection.CreateBatchDmlCommand();

        cmd.Add("INSERT INTO Albums (SingerId, AlbumId, AlbumTitle, MarketingBudget) VALUES (1, 3, 'Test Album Title', 10000)");

        cmd.Add("UPDATE Albums SET MarketingBudget = MarketingBudget * 2 WHERE SingerId = 1 and AlbumId = 3");

        IEnumerable<long> affectedRows = await cmd.ExecuteNonQueryAsync();

        Console.WriteLine($"Executed {affectedRows.Count()} " + "SQL statements using Batch DML.");
        return affectedRows.Count();
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

func updateUsingBatchDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmts := []spanner.Statement{
         {SQL: `INSERT INTO Albums
             (SingerId, AlbumId, AlbumTitle, MarketingBudget)
             VALUES (1, 3, 'Test Album Title', 10000)`},
         {SQL: `UPDATE Albums
             SET MarketingBudget = MarketingBudget * 2
             WHERE SingerId = 1 and AlbumId = 3`},
     }
     rowCounts, err := txn.BatchUpdate(ctx, stmts)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "Executed %d SQL statements using Batch DML.\n", len(rowCounts))
     return nil
 })
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void updateUsingBatchDml(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(transaction -> {
        List<Statement> stmts = new ArrayList<Statement>();
        String sql =
            "INSERT INTO Albums "
                + "(SingerId, AlbumId, AlbumTitle, MarketingBudget) "
                + "VALUES (1, 3, 'Test Album Title', 10000) ";
        stmts.add(Statement.of(sql));
        sql =
            "UPDATE Albums "
                + "SET MarketingBudget = MarketingBudget * 2 "
                + "WHERE SingerId = 1 and AlbumId = 3";
        stmts.add(Statement.of(sql));
        long[] rowCounts;
        try {
          rowCounts = transaction.batchUpdate(stmts);
        } catch (SpannerBatchUpdateException e) {
          rowCounts = e.getUpdateCounts();
        }
        for (int i = 0; i < rowCounts.length; i++) {
          System.out.printf("%d record updated by stmt %d.\n", rowCounts[i], i);
        }
        return null;
      });
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

const insert = {
  sql: `INSERT INTO Albums (SingerId, AlbumId, AlbumTitle, MarketingBudget)
    VALUES (1, 3, "Test Album Title", 10000)`,
};

const update = {
  sql: `UPDATE Albums SET MarketingBudget = MarketingBudget * 2
    WHERE SingerId = 1 and AlbumId = 3`,
};

const dmlStatements = [insert, update];

try {
  await database.runTransactionAsync(async transaction => {
    const [rowCounts] = await transaction.batchUpdate(dmlStatements);
    await transaction.commit();
    console.log(
      `Successfully executed ${rowCounts.length} SQL statements using Batch DML.`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
  throw err;
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
use Google\Cloud\Spanner\Transaction;

/**
 * Updates sample data in the database with Batch DML.
 *
 * This requires the `MarketingBudget` column which must be created before
 * running this sample. You can add the column by running the `add_column`
 * sample or by running this DDL statement against your database:
 *
 *     ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * Example:
 * ```
 * update_data_with_batch_dml($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_batch_dml(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $batchDmlResult = $database->runTransaction(function (Transaction $t) {
        $result = $t->executeUpdateBatch([
            [
                'sql' => 'INSERT INTO Albums '
                . '(SingerId, AlbumId, AlbumTitle, MarketingBudget) '
                . "VALUES (1, 3, 'Test Album Title', 10000)"
            ],
            [
                'sql' => 'UPDATE Albums '
                . 'SET MarketingBudget = MarketingBudget * 2 '
                . 'WHERE SingerId = 1 and AlbumId = 3'
            ],
        ]);
        $t->commit();
        $rowCounts = count($result->rowCounts());
        printf('Executed %s SQL statements using Batch DML.' . PHP_EOL,
            $rowCounts);
    });
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
from google.rpc.code_pb2 import OK

# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

insert_statement = (
    "INSERT INTO Albums "
    "(SingerId, AlbumId, AlbumTitle, MarketingBudget) "
    "VALUES (1, 3, 'Test Album Title', 10000)"
)

update_statement = (
    "UPDATE Albums "
    "SET MarketingBudget = MarketingBudget * 2 "
    "WHERE SingerId = 1 and AlbumId = 3"
)

def update_albums(transaction):
    status, row_cts = transaction.batch_update([insert_statement, update_statement])

    if status.code != OK:
        # Do handling here.
        # Note: the exception will still be raised when
        # `commit` is called by `run_in_transaction`.
        return

    print("Executed {} SQL statements using Batch DML.".format(len(row_cts)))

database.run_in_transaction(update_albums)
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

row_counts = nil
client.transaction do |transaction|
  row_counts = transaction.batch_update do |b|
    b.batch_update(
      "INSERT INTO Albums " \
      "(SingerId, AlbumId, AlbumTitle, MarketingBudget) " \
      "VALUES (1, 3, 'Test Album Title', 10000)"
    )
    b.batch_update(
      "UPDATE Albums " \
      "SET MarketingBudget = MarketingBudget * 2 " \
      "WHERE SingerId = 1 and AlbumId = 3"
    )
  end
end

statement_count = row_counts.count

puts "Executed #{statement_count} SQL statements using Batch DML."
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
