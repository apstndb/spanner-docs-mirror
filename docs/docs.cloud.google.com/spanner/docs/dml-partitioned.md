Partitioned [Data Manipulation Language](/spanner/docs/reference/standard-sql/dml-syntax) (partitioned DML) is designed for the following types of bulk updates and deletes:

  - Periodic cleanup and garbage collection. Examples are deleting old rows or setting columns to `  NULL  ` .
  - Backfilling new columns with default values. An example is using an `  UPDATE  ` statement to set a new column's value to `  False  ` where it is currently `  NULL  ` .

Partitioned DML isn't suitable for small-scale transaction processing. If you want to run a statement on a few rows, use transactional DMLs with identifiable primary keys. For more information, see [Using DML](/spanner/docs/dml-tasks#using-dml) .

If you need to commit a large number of blind writes, but don't require an atomic transaction, you can bulk modify your Spanner tables using batch write. For more information, see [Modify data using batch writes](/spanner/docs/batch-write) .

You can get insights on active partitioned DML queries and their progress from statistics tables in your Spanner database. For more information, see [Active partitioned DMLs statistics](/spanner/docs/introspection/active-partitioned-dmls) .

## DML and partitioned DML

Spanner supports two execution modes for DML statements:

  - DML, which is suitable for transaction processing. For more information, see [Using DML](/spanner/docs/dml-tasks#using-dml) .

  - Partitioned DML, which enables large-scale, database-wide operations with minimal impact on concurrent transaction processing by partitioning the key space and running the statement over partitions in separate, smaller-scoped transactions. For more information, see [Using partitioned DML](/spanner/docs/dml-tasks#partitioned-dml) .

The following table highlights some of the differences between the two execution modes.

<table>
<thead>
<tr class="header">
<th>DML</th>
<th>Partitioned DML</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Rows that don't match the <code dir="ltr" translate="no">       WHERE      </code> clause might be locked.</td>
<td>Only rows that match the <code dir="ltr" translate="no">       WHERE      </code> clause are locked.</td>
</tr>
<tr class="even">
<td>Transaction size limits apply.</td>
<td>Spanner handles the transaction limits and per-transaction concurrency limits.</td>
</tr>
<tr class="odd">
<td>Statements don't need to be idempotent.</td>
<td>A DML statement must be <a href="#partitionable-idempotent">idempotent</a> to ensure consistent results.</td>
</tr>
<tr class="even">
<td>A transaction can include multiple DML and SQL statements.</td>
<td>A partitioned transaction can include only one DML statement.</td>
</tr>
<tr class="odd">
<td>There are no restrictions on complexity of statements.</td>
<td>Statements must be <a href="#partitionable-idempotent">fully partitionable</a> .</td>
</tr>
<tr class="even">
<td>You create read-write transactions in your client code.</td>
<td>Spanner creates the transactions.</td>
</tr>
</tbody>
</table>

## Partitionable and idempotent

When a partitioned DML statement runs, rows in one partition don't have access to rows in other partitions, and you cannot choose how Spanner creates the partitions. Partitioning ensures scalability, but it also means that partitioned DML statements must be *fully partitionable* . That is, the partitioned DML statement must be expressible as the union of a set of statements, where each statement accesses a single row of the table and each statement accesses no other tables. For example, a DML statement that accesses multiple tables or performs a self-join is not partitionable. If the DML statement is not partitionable, Spanner returns the error `  BadUsage  ` .

These DML statements are fully partitionable, because each statement can be applied to a single row in the table:

``` text
UPDATE Singers SET LastName = NULL WHERE LastName = '';

DELETE FROM Albums WHERE MarketingBudget > 10000;
```

This DML statement is not fully partitionable, because it accesses multiple tables:

``` text
# Not fully partitionable
DELETE FROM Singers WHERE
SingerId NOT IN (SELECT SingerId FROM Concerts);
```

Spanner might execute a partitioned DML statement multiple times against some partitions due to network-level retries. As a result, a statement might be executed more than once against a row. The statement must therefore be *idempotent* to yield consistent results. A statement is idempotent if executing it multiple times against a single row leads to the same result.

This DML statement is idempotent:

``` text
UPDATE Singers SET MarketingBudget = 1000 WHERE true;
```

This DML statement is not idempotent:

``` text
UPDATE Singers SET MarketingBudget = 1.5 * MarketingBudget WHERE true;
```

## Delete rows from parent tables with indexed child tables

When you use a partitioned DML statement to delete rows in a parent table, the operation might fail with the error: `  The transaction contains too many mutations  ` . This occurs if the parent table has interleaved child tables that contain a global index. Mutations to the child table rows themselves aren't counted against the transaction's [mutation limit](/spanner/quotas#limits-for) . However, the corresponding mutations to the index entries are counted. If a large number of child table index entries are affected, the transaction might exceed the mutation limit.

To avoid this error, delete the rows in two separate partitioned DML statements:

1.  Run a partitioned delete on the child tables.
2.  Run a partitioned delete on the parent table.

This two-step process helps keep the mutation count within the allowed limits for each transaction. Alternatively, you can drop the global index on the child table before deleting the parent rows.

## Row locking

Spanner acquires a lock only if a row is a candidate for update or deletion. This behavior is different from [DML execution](/spanner/docs/dml-tasks#locking) , which might read-lock rows that don't match the `  WHERE  ` clause.

## Execution and transactions

Whether a DML statement is partitioned or not depends on the client library method that you choose for execution. Each client library provides separate methods for [DML execution](/spanner/docs/dml-tasks#client-library-dml) and [Partitioned DML execution](/spanner/docs/dml-tasks#client-library-partitioned) .

You can execute only one partitioned DML statement in a call to the client library method.

Spanner does not apply the partitioned DML statements atomically across the entire table. Spanner does, however, apply partitioned DML statements atomically across each partition.

Partitioned DML does not support commit or rollback. Spanner executes and applies the DML statement immediately.

  - If you cancel the operation, Spanner cancels the executing partitions and doesn't start the remaining partitions. Spanner does not roll back any partitions that have already executed.
  - If the execution of the statement causes an error, then execution stops across all partitions and Spanner returns that error for the entire operation. Some examples of errors are violations of data type constraints, violations of `  UNIQUE INDEX  ` , and violations of `  ON DELETE NO ACTION  ` . Depending on the point in time when the execution failed, the statement might have successfully run against some partitions, and might never have been run against other partitions.

If the partitioned DML statement succeeds, then Spanner ran the statement at least once against each partition of the key range.

## Count of modified rows

A partitioned DML statement returns a lower bound on the number of modified rows. It might not be an exact count of the number of rows modified, because there is no guarantee that Spanner counts all the modified rows.

## Transaction limits

Spanner creates the partitions and transactions that it needs to execute a partitioned DML statement. Transaction limits or per-transaction concurrency limits apply, but Spanner attempts to keep the transactions within the limits.

Spanner allows a maximum of 20,000 concurrent partitioned DML statements per database.

## Unsupported features

Spanner does not support some features for partitioned DML:

  - `  INSERT  ` is not supported.
  - Google Cloud console: You can't execute partitioned DML statements in the Google Cloud console.
  - Query plans and profiling: The Google Cloud CLI and the client libraries don't support query plans and profiling.
  - Subqueries that read from another table, or a different row of the same table.

For complex scenarios, such as moving a table or transformations that require joins across tables, consider [using the Dataflow connector](/spanner/docs/dataflow-connector) .

## Best practices

Apply the following best practices to improve the performance of your partitioned DML statements:

  - **Avoid high concurrency:** Running a large number of partitioned DML statements concurrently (for example, more than 100) might lead to lock contention on internal system tables, degrading performance. Instead of running a high number of concurrent statements, use a single partitioned DML statement.
  - **Utilize [`  PDML_MAX_PARALLELISM  `](/spanner/docs/reference/standard-sql/dml-syntax#statement_hints) :** To increase the throughput of a single partitioned DML statement, especially on tables with many splits, set a higher value for the `  PDML_MAX_PARALLELISM  ` statement hint. This allows the single statement to use more parallelism internally. Setting a higher value for `  PDML_MAX_PARALLELISM  ` results in more compute usage, so you should try to balance compute usage and increased processing speed.
  - **Let Spanner handle partitioning:** Avoid manually sharding your data (for example, using primary key ranges) and running separate partitioned DML statements on each shard. Partitioned DML is designed to efficiently partition the work across the entire table. Custom sharding often increases overhead and might worsen contention.
  - **Understand partitioning scope:** Partitioned DML operations are parallelized across all splits in the entire database, not just the splits containing data for the table being modified. This means that for databases with a large number of splits, there might be overhead even if the target table is small or the modified data is localized. Partitioned DML might not be the most efficient choice for modifying a very small portion of a large database.
  - **Consider alternatives for frequent, small deletions:** For use cases involving frequent deletions of a small number of known rows, using DML statements within transactions or the [BatchWrite API](/spanner/docs/batch-write) might offer better performance and lower overhead than using partitioned DML.

## Examples

The following code example updates the `  MarketingBudget  ` column of the `  Albums  ` table.

### C++

You use the `  ExecutePartitionedDml()  ` function to execute a partitioned DML statement.

``` cpp
void DmlPartitionedUpdate(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto result = client.ExecutePartitionedDml(
      spanner::SqlStatement("UPDATE Albums SET MarketingBudget = 100000"
                            "  WHERE SingerId > 1"));
  if (!result) throw std::move(result).status();
  std::cout << "Updated at least " << result->row_count_lower_bound
            << " row(s) [spanner_dml_partitioned_update]\n";
}
```

### C\#

You use the `  ExecutePartitionedUpdateAsync()  ` method to execute a partitioned DML statement.

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateUsingPartitionedDmlCoreAsyncSample
{
    public async Task<long> UpdateUsingPartitionedDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1");
        long rowCount = await cmd.ExecutePartitionedUpdateAsync();

        Console.WriteLine($"{rowCount} row(s) updated...");
        return rowCount;
    }
}
```

### Go

You use the `  PartitionedUpdate()  ` method to execute a partitioned DML statement.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func updateUsingPartitionedDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1"}
 rowCount, err := client.PartitionedUpdate(ctx, stmt)
 if err != nil {
     return err
 }
 fmt.Fprintf(w, "%d record(s) updated.\n", rowCount)
 return nil
}
```

### Java

You use the `  executePartitionedUpdate()  ` method to execute a partitioned DML statement.

``` java
static void updateUsingPartitionedDml(DatabaseClient dbClient) {
  String sql = "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1";
  long rowCount = dbClient.executePartitionedUpdate(Statement.of(sql));
  System.out.printf("%d records updated.\n", rowCount);
}
```

### Node.js

You use the `  runPartitionedUpdate()  ` method to execute a partitioned DML statement.

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

try {
  const [rowCount] = await database.runPartitionedUpdate({
    sql: 'UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1',
  });
  console.log(`Successfully updated ${rowCount} records.`);
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

You use the `  executePartitionedUpdate()  ` method to execute a partitioned DML statement.

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Updates sample data in the database by partition with a DML statement.
 *
 * This updates the `MarketingBudget` column which must be created before
 * running this sample. You can add the column by running the `add_column`
 * sample or by running this DDL statement against your database:
 *
 *     ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * Example:
 * ```
 * update_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_partitioned_dml(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $rowCount = $database->executePartitionedUpdate(
        'UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1'
    );

    printf('Updated %d row(s).' . PHP_EOL, $rowCount);
}
````

### Python

You use the `  execute_partitioned_dml()  ` method to execute a partitioned DML statement.

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

row_ct = database.execute_partitioned_dml(
    "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1"
)

print("{} records updated.".format(row_ct))
```

### Ruby

You use the `  execute_partitioned_update()  ` method to execute a partitioned DML statement.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

row_count = client.execute_partition_update(
  "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1"
)

puts "#{row_count} records updated."
```

The following code example deletes rows from the `  Singers  ` table, based on the `  SingerId  ` column.

### C++

``` cpp
void DmlPartitionedDelete(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto result = client.ExecutePartitionedDml(
      spanner::SqlStatement("DELETE FROM Singers WHERE SingerId > 10"));
  if (!result) throw std::move(result).status();
  std::cout << "Deleted at least " << result->row_count_lower_bound
            << " row(s) [spanner_dml_partitioned_delete]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class DeleteUsingPartitionedDmlCoreAsyncSample
{
    public async Task<long> DeleteUsingPartitionedDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("DELETE FROM Singers WHERE SingerId > 10");
        long rowCount = await cmd.ExecutePartitionedUpdateAsync();

        Console.WriteLine($"{rowCount} row(s) deleted...");
        return rowCount;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func deleteUsingPartitionedDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: "DELETE FROM Singers WHERE SingerId > 10"}
 rowCount, err := client.PartitionedUpdate(ctx, stmt)
 if err != nil {
     return err

 }
 fmt.Fprintf(w, "%d record(s) deleted.", rowCount)
 return nil
}
```

### Java

``` java
static void deleteUsingPartitionedDml(DatabaseClient dbClient) {
  String sql = "DELETE FROM Singers WHERE SingerId > 10";
  long rowCount = dbClient.executePartitionedUpdate(Statement.of(sql));
  System.out.printf("%d records deleted.\n", rowCount);
}
```

### Node.js

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

try {
  const [rowCount] = await database.runPartitionedUpdate({
    sql: 'DELETE FROM Singers WHERE SingerId > 10',
  });
  console.log(`Successfully deleted ${rowCount} records.`);
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Delete sample data in the database by partition with a DML statement.
 *
 * This updates the `MarketingBudget` column which must be created before
 * running this sample. You can add the column by running the `add_column`
 * sample or by running this DDL statement against your database:
 *
 *     ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * Example:
 * ```
 * update_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function delete_data_with_partitioned_dml(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $rowCount = $database->executePartitionedUpdate(
        'DELETE FROM Singers WHERE SingerId > 10'
    );

    printf('Deleted %d row(s).' . PHP_EOL, $rowCount);
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

row_ct = database.execute_partitioned_dml("DELETE FROM Singers WHERE SingerId > 10")

print("{} record(s) deleted.".format(row_ct))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

row_count = client.execute_partition_update(
  "DELETE FROM Singers WHERE SingerId > 10"
)

puts "#{row_count} records deleted."
```

## What's next?

  - Learn how to modify data [Using DML](/spanner/docs/dml-tasks#partitioned-dml) .

  - Learn about [Data Manipulation Language (DML) best practices](/spanner/docs/dml-best-practices) .

  - To learn about the differences between DML and mutations, see [Compare DML and Mutations](/spanner/docs/dml-versus-mutations)

  - Consider [using the Dataflow connector](/spanner/docs/dataflow-connector) for other data transformation scenarios.
