This page describes how to create, alter, and drop a sequence in Spanner using Data Definition Language (DDL) statements. You can also see how to use a sequence in a default value to populate a primary key column.

See the complete sequence DDL syntax reference for ( [GoogleSQL-dialect databases](/spanner/docs/reference/standard-sql/data-definition-language#sequence_statements) and [PostgreSQL-dialect databases](/spanner/docs/reference/postgresql/data-definition-language#sequence_statements) ).

**Note:** Spanner now supports [automatically generated primary key values](/spanner/docs/primary-key-default-value) . For more information, see [Simplifying best practices at scale with auto-generated keys in Spanner](https://cloud.google.com/blog/products/databases/announcing-support-for-auto-generated-keys-in-spanner) .

## Create a sequence

The following code example creates a sequence `  Seq  ` , uses it in the primary key default value of the table `  Customers  ` , and inserts three new rows into the `  Customers  ` table.

### GoogleSQL

### C++

``` cpp
void CreateSequence(
    google::cloud::spanner_admin::DatabaseAdminClient admin_client,
    google::cloud::spanner::Client client, std::string const& project_id,
    std::string const& instance_id, std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  std::vector<std::string> statements;
  statements.emplace_back(R"""(
      CREATE SEQUENCE Seq
          OPTIONS (sequence_kind = 'bit_reversed_positive')
  )""");
  statements.emplace_back(R"""(
      CREATE TABLE Customers (
          CustomerId INT64 DEFAULT (GET_NEXT_SEQUENCE_VALUE(SEQUENCE Seq)),
          CustomerName STRING(1024)
      ) PRIMARY KEY (CustomerId)
  )""");
  auto metadata =
      admin_client.UpdateDatabaseDdl(database.FullName(), std::move(statements))
          .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Created `Seq` sequence and `Customers` table,"
            << " where the key column `CustomerId`"
            << " uses the sequence as a default value,"
            << " new DDL:\n"
            << metadata->DebugString();
  auto commit = client.Commit(
      [&client](google::cloud::spanner::Transaction txn)
          -> google::cloud::StatusOr<google::cloud::spanner::Mutations> {
        auto sql = google::cloud::spanner::SqlStatement(R"""(
            INSERT INTO Customers (CustomerName)
              VALUES ('Alice'),
                     ('David'),
                     ('Marc')
              THEN RETURN CustomerId
        )""");
        using RowType = std::tuple<std::int64_t>;
        auto rows = client.ExecuteQuery(std::move(txn), std::move(sql));
        // Note: This mutator might be re-run, or its effects discarded, so
        // changing non-transactional state (e.g., by producing output) is,
        // in general, not something to be imitated.
        for (auto& row : google::cloud::spanner::StreamOf<RowType>(rows)) {
          if (!row) return std::move(row).status();
          std::cout << "Inserted customer record with CustomerId: "
                    << std::get<0>(*row) << "\n";
        }
        std::cout << "Number of customer records inserted is: "
                  << rows.RowsModified() << "\n";
        return google::cloud::spanner::Mutations{};
      });
  if (!commit) throw std::move(commit).status();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class CreateSequenceSample
{
    public async Task<List<long>> CreateSequenceAsync(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        string[] statements =
        {
            "CREATE SEQUENCE Seq OPTIONS (sequence_kind = 'bit_reversed_positive')",
            "CREATE TABLE Customers (CustomerId INT64 DEFAULT (GET_NEXT_SEQUENCE_VALUE(SEQUENCE Seq)), CustomerName STRING(1024)) PRIMARY KEY (CustomerId)"
        };
        var operation = await databaseAdminClient.UpdateDatabaseDdlAsync(databaseName, statements);

        var completedResponse = await operation.PollUntilCompletedAsync();

        if (completedResponse.IsFaulted)
        {
            throw completedResponse.Exception;
        }

        Console.WriteLine("Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value");

        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand(
            @"INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') THEN RETURN CustomerId");

        var reader = await cmd.ExecuteReaderAsync();
        var customerIds = new List<long>();
        while (await reader.ReadAsync())
        {
            var customerId = reader.GetFieldValue<long>("CustomerId");
            Console.WriteLine($"Inserted customer record with CustomerId: {customerId}");
            customerIds.Add(customerId);
        }
        Console.WriteLine($"Number of customer records inserted is: {customerIds.Count}");
        return customerIds;
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
 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/api/iterator"
)

func createSequence(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // List of DDL statements to be applied to the database.
 // Create a sequence, and then use the sequence as auto generated primary key in Customers table.
 ddl := []string{
     "CREATE SEQUENCE Seq OPTIONS (sequence_kind = 'bit_reversed_positive')",
     "CREATE TABLE Customers (CustomerId INT64 DEFAULT (GET_NEXT_SEQUENCE_VALUE(Sequence Seq)), CustomerName STRING(1024)) PRIMARY KEY (CustomerId)",
 }
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database:   db,
     Statements: ddl,
 })
 if err != nil {
     return err
 }
 // Wait for the UpdateDatabaseDdl operation to finish.
 if err := op.Wait(ctx); err != nil {
     return fmt.Errorf("waiting for bit reverse sequence creation to finish failed: %w", err)
 }
 fmt.Fprintf(w, "Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value\n")

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Inserts records into the Customers table.
 // The ReadWriteTransaction function returns the commit timestamp and an error.
 // The commit timestamp is ignored in this case.
 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') THEN RETURN CustomerId`,
     }
     iter := txn.Query(ctx, stmt)
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             break
         }
         if err != nil {
             return err
         }
         var customerId int64
         if err := row.Columns(&customerId); err != nil {
             return err
         }
         fmt.Fprintf(w, "Inserted customer record with CustomerId: %d\n", customerId)
     }
     fmt.Fprintf(w, "Number of customer records inserted is: %d\n", iter.RowCount)
     return nil
 })
 return err
}
```

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.Objects;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class CreateSequenceSample {

  static void createSequence() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    createSequence(projectId, instanceId, databaseId);
  }

  static void createSequence(String projectId, String instanceId, String databaseId) {

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      databaseAdminClient
          .updateDatabaseDdlAsync(
              DatabaseName.of(projectId, instanceId, databaseId),
              ImmutableList.of(
                  "CREATE SEQUENCE Seq OPTIONS (sequence_kind = 'bit_reversed_positive')",
                  "CREATE TABLE Customers (CustomerId INT64 DEFAULT "
                      + "(GET_NEXT_SEQUENCE_VALUE(SEQUENCE Seq)), CustomerName STRING(1024)) "
                      + "PRIMARY KEY (CustomerId)"))
          .get(5, TimeUnit.MINUTES);

      System.out.println(
          "Created Seq sequence and Customers table, where the key column CustomerId "
              + "uses the sequence as a default value");

      final DatabaseClient dbClient =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));

      Long insertCount =
          dbClient
              .readWriteTransaction()
              .run(
                  transaction -> {
                    try (ResultSet rs =
                        transaction.executeQuery(
                            Statement.of(
                                "INSERT INTO Customers (CustomerName) VALUES "
                                    + "('Alice'), ('David'), ('Marc') THEN RETURN CustomerId"))) {
                      while (rs.next()) {
                        System.out.printf(
                            "Inserted customer record with CustomerId: %d\n", rs.getLong(0));
                      }
                      return Objects.requireNonNull(rs.getStats()).getRowCountExact();
                    }
                  });
      System.out.printf("Number of customer records inserted is: %d\n", insertCount);
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw SpannerExceptionFactory.asSpannerException(e.getCause());
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    } catch (TimeoutException e) {
      // If the operation timed out propagate the timeout
      throw SpannerExceptionFactory.propagateTimeout(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library.
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

async function createSequence(instanceId, databaseId) {
  // Gets a reference to a Cloud Spanner Database Admin Client object
  const databaseAdminClient = spanner.getDatabaseAdminClient();

  const request = [
    "CREATE SEQUENCE Seq OPTIONS (sequence_kind = 'bit_reversed_positive')",
    'CREATE TABLE Customers (CustomerId INT64 DEFAULT (GET_NEXT_SEQUENCE_VALUE(Sequence Seq)), CustomerName STRING(1024)) PRIMARY KEY (CustomerId)',
  ];

  // Creates a new table with sequence
  try {
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log(
      'Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value.',
    );
  } catch (err) {
    console.error('ERROR:', err);
  }

  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  database.runTransaction(async (err, transaction) => {
    if (err) {
      console.error(err);
      return;
    }
    try {
      const [rows, stats] = await transaction.run({
        sql: "INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') THEN RETURN CustomerId",
      });

      rows.forEach(row => {
        console.log(
          `Inserted customer record with CustomerId: ${
            row.toJSON({wrapNumbers: true}).CustomerId.value
          }`,
        );
      });

      const rowCount = Math.floor(stats[stats.rowCount]);
      console.log(`Number of customer records inserted is: ${rowCount}`);

      await transaction.commit();
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the database when finished.
      await database.close();
    }
  });
}
await createSequence(instanceId, databaseId);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Result;

/**
 * Creates a sequence.
 *
 * Example:
 * ```
 * create_sequence($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function create_sequence(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $spanner = new SpannerClient();

    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [
            "CREATE SEQUENCE Seq OPTIONS (sequence_kind = 'bit_reversed_positive')",
            'CREATE TABLE Customers (CustomerId INT64 DEFAULT (GET_NEXT_SEQUENCE_VALUE(' .
            'Sequence Seq)), CustomerName STRING(1024)) PRIMARY KEY (CustomerId)'
        ]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf(
        'Created Seq sequence and Customers table, where ' .
        'the key column CustomerId uses the sequence as a default value' .
        PHP_EOL
    );

    $transaction = $database->transaction();
    $res = $transaction->execute(
        'INSERT INTO Customers (CustomerName) VALUES ' .
        "('Alice'), ('David'), ('Marc') THEN RETURN CustomerId"
    );
    $rows = $res->rows(Result::RETURN_ASSOCIATIVE);

    foreach ($rows as $row) {
        printf('Inserted customer record with CustomerId: %d %s',
            $row['CustomerId'],
            PHP_EOL
        );
    }
    $transaction->commit();

    printf(sprintf(
        'Number of customer records inserted is: %d %s',
        $res->stats()['rowCountExact'],
        PHP_EOL
    ));
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def create_sequence(instance_id, database_id):
    """Creates the Sequence and insert data"""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "CREATE SEQUENCE Seq OPTIONS (sequence_kind = 'bit_reversed_positive')",
            """CREATE TABLE Customers (
            CustomerId     INT64 DEFAULT (GET_NEXT_SEQUENCE_VALUE(Sequence Seq)),
            CustomerName      STRING(1024)
            ) PRIMARY KEY (CustomerId)""",
        ],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value on database {} on instance {}".format(
            database_id, instance_id
        )
    )

    def insert_customers(transaction):
        results = transaction.execute_sql(
            "INSERT INTO Customers (CustomerName) VALUES "
            "('Alice'), "
            "('David'), "
            "('Marc') "
            "THEN RETURN CustomerId"
        )
        for result in results:
            print("Inserted customer record with Customer Id: {}".format(*result))
        print(
            "Number of customer records inserted is {}".format(
                results.stats.row_count_exact
            )
        )

    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    database.run_in_transaction(insert_customers)
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to create a sequence.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_create_sequence project_id:, instance_id:, database_id:
  db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

  database_path = db_admin_client.database_path project: project_id,
                                                instance: instance_id,
                                                database: database_id


  job = db_admin_client.update_database_ddl database: database_path, statements: [
    "CREATE SEQUENCE Seq OPTIONS (sequence_kind = 'bit_reversed_positive')",
    "CREATE TABLE Customers (CustomerId INT64 DEFAULT (GET_NEXT_SEQUENCE_VALUE(Sequence Seq)), CustomerName STRING(1024)) PRIMARY KEY (CustomerId)"
  ]

  puts "Waiting for operation to complete..."
  job.wait_until_done!
  puts "Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value"
end
```

### PostgreSQL

### C++

``` cpp
void CreateSequence(
    google::cloud::spanner_admin::DatabaseAdminClient admin_client,
    google::cloud::spanner::Database const& database,
    google::cloud::spanner::Client client) {
  std::vector<std::string> statements;
  statements.emplace_back(R"""(
      CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE
  )""");
  statements.emplace_back(R"""(
      CREATE TABLE Customers (
          CustomerId    BIGINT DEFAULT NEXTVAL('Seq'),
          CustomerName  CHARACTER VARYING(1024),
          PRIMARY KEY (CustomerId)
      )
  )""");
  auto metadata =
      admin_client.UpdateDatabaseDdl(database.FullName(), std::move(statements))
          .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Created `Seq` sequence and `Customers` table,"
            << " where the key column `CustomerId`"
            << " uses the sequence as a default value,"
            << " new DDL:\n"
            << metadata->DebugString();
  auto commit = client.Commit(
      [&client](google::cloud::spanner::Transaction txn)
          -> google::cloud::StatusOr<google::cloud::spanner::Mutations> {
        auto sql = google::cloud::spanner::SqlStatement(R"""(
            INSERT INTO Customers (CustomerName)
                VALUES ('Alice'),
                       ('David'),
                       ('Marc')
                RETURNING CustomerId
        )""");
        using RowType = std::tuple<std::int64_t>;
        auto rows = client.ExecuteQuery(std::move(txn), std::move(sql));
        // Note: This mutator might be re-run, or its effects discarded, so
        // changing non-transactional state (e.g., by producing output) is,
        // in general, not something to be imitated.
        for (auto& row : google::cloud::spanner::StreamOf<RowType>(rows)) {
          if (!row) return std::move(row).status();
          std::cout << "Inserted customer record with CustomerId: "
                    << std::get<0>(*row) << "\n";
        }
        std::cout << "Number of customer records inserted is: "
                  << rows.RowsModified() << "\n";
        return google::cloud::spanner::Mutations{};
      });
  if (!commit) throw std::move(commit).status();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Common.V1;
using System.Threading.Tasks;
using System;
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;

public class CreateSequencePostgresqlSample
{
    public async Task<List<long>> CreateSequencePostgresqlSampleAsync(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        string[] statements =
        {
            "CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE ;",
            "CREATE TABLE Customers (CustomerId BIGINT DEFAULT nextval('Seq'), CustomerName character varying(1024), PRIMARY KEY (CustomerId))"
        };
        var operation = await databaseAdminClient.UpdateDatabaseDdlAsync(databaseName, statements);

        var completedResponse = await operation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            throw completedResponse.Exception;
        }

        Console.WriteLine("Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value");

        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand(
            @"INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') RETURNING CustomerId");

        var reader = await cmd.ExecuteReaderAsync();
        var customerIds = new List<long>();
        while (await reader.ReadAsync())
        {
            var customerId = reader.GetFieldValue<long>("customerid");
            Console.WriteLine($"Inserted customer record with CustomerId: {customerId}");
            customerIds.Add(customerId);
        }
        Console.WriteLine($"Number of customer records inserted is: {customerIds.Count}");
        return customerIds;
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
 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/api/iterator"
)

func pgCreateSequence(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // List of DDL statements to be applied to the database.
 // Create a sequence, and then use the sequence as auto generated primary key in Customers table.
 ddl := []string{
     "CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE",
     "CREATE TABLE Customers (CustomerId BIGINT DEFAULT nextval('Seq'), CustomerName character varying(1024), PRIMARY KEY (CustomerId))",
 }
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database:   db,
     Statements: ddl,
 })
 if err != nil {
     return err
 }
 // Wait for the UpdateDatabaseDdl operation to finish.
 if err := op.Wait(ctx); err != nil {
     return fmt.Errorf("waiting for bit reverse sequence creation to finish failed: %w", err)
 }
 fmt.Fprintf(w, "Created Seq sequence and Customers table, where its key column CustomerId uses the sequence as a default value\n")

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Inserts records into the Customers table.
 // The ReadWriteTransaction function returns the commit timestamp and an error.
 // The commit timestamp is ignored in this case.
 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') RETURNING CustomerId`,
     }
     iter := txn.Query(ctx, stmt)
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             break
         }
         if err != nil {
             return err
         }
         var customerId int64
         if err := row.Columns(&customerId); err != nil {
             return err
         }
         fmt.Fprintf(w, "Inserted customer record with CustomerId: %d\n", customerId)
     }
     fmt.Fprintf(w, "Number of customer records inserted is: %d\n", iter.RowCount)
     return nil
 })
 return err
}
```

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.Objects;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class PgCreateSequenceSample {

  static void pgCreateSequence() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    pgCreateSequence(projectId, instanceId, databaseId);
  }

  static void pgCreateSequence(String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      databaseAdminClient
          .updateDatabaseDdlAsync(DatabaseName.of(projectId, instanceId, databaseId).toString(),
              ImmutableList.of(
                  "CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE;",
                  "CREATE TABLE Customers (CustomerId BIGINT DEFAULT nextval('Seq'), "
                      + "CustomerName character varying(1024), PRIMARY KEY (CustomerId))"))
          .get(5, TimeUnit.MINUTES);

      System.out.println(
          "Created Seq sequence and Customers table, where the key column "
              + "CustomerId uses the sequence as a default value");

      final DatabaseClient dbClient =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));

      Long insertCount =
          dbClient
              .readWriteTransaction()
              .run(
                  transaction -> {
                    try (ResultSet rs =
                        transaction.executeQuery(
                            Statement.of(
                                "INSERT INTO Customers (CustomerName) VALUES "
                                    + "('Alice'), ('David'), ('Marc') RETURNING CustomerId"))) {
                      while (rs.next()) {
                        System.out.printf(
                            "Inserted customer record with CustomerId: %d\n", rs.getLong(0));
                      }
                      return Objects.requireNonNull(rs.getStats()).getRowCountExact();
                    }
                  });
      System.out.printf("Number of customer records inserted is: %d\n", insertCount);
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw SpannerExceptionFactory.asSpannerException(e.getCause());
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    } catch (TimeoutException e) {
      // If the operation timed out propagate the timeout
      throw SpannerExceptionFactory.propagateTimeout(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library.
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

async function createSequence(instanceId, databaseId) {
  // Gets a reference to a Cloud Spanner Database Admin Client object
  const databaseAdminClient = spanner.getDatabaseAdminClient();

  const request = [
    'CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE',
    "CREATE TABLE Customers (CustomerId BIGINT DEFAULT nextval('Seq'), CustomerName character varying(1024), PRIMARY KEY (CustomerId))",
  ];

  // Creates a new table with sequence
  try {
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log(
      'Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value',
    );
  } catch (err) {
    console.error('ERROR:', err);
  }

  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  database.runTransaction(async (err, transaction) => {
    if (err) {
      console.error(err);
      return;
    }
    try {
      const [rows, stats] = await transaction.run({
        sql: "INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') RETURNING CustomerId",
      });

      rows.forEach(row => {
        console.log(
          `Inserted customer record with CustomerId: ${
            row.toJSON({wrapNumbers: true}).customerid.value
          }`,
        );
      });

      const rowCount = Math.floor(stats[stats.rowCount]);
      console.log(`Number of customer records inserted is: ${rowCount}`);

      await transaction.commit();
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the spanner client when finished.
      // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
      spanner.close();
    }
  });
}
await createSequence(instanceId, databaseId);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Result;

/**
 * Creates a sequence.
 * Example:
 * ```
 * pg_create_sequence($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud Project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function pg_create_sequence(
    string $projectId,
    string $instanceId,
    string $databaseId
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);
    $transaction = $database->transaction();
    $operation = $databaseAdminClient->updateDatabaseDdl(new UpdateDatabaseDdlRequest([
        'database' => DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId),
        'statements' => [
            'CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE',
            "CREATE TABLE Customers (
            CustomerId           BIGINT DEFAULT nextval('Seq'), 
            CustomerName         CHARACTER VARYING(1024), 
            PRIMARY KEY (CustomerId))"
        ]
    ]));

    print('Waiting for operation to complete ...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf(
        'Created Seq sequence and Customers table, where ' .
        'the key column CustomerId uses the sequence as a default value' .
        PHP_EOL
    );

    $res = $transaction->execute(
        'INSERT INTO Customers (CustomerName) VALUES ' .
        "('Alice'), ('David'), ('Marc') RETURNING CustomerId"
    );
    $rows = $res->rows(Result::RETURN_ASSOCIATIVE);

    foreach ($rows as $row) {
        printf('Inserted customer record with CustomerId: %d %s',
            $row['customerid'],
            PHP_EOL
        );
    }
    $transaction->commit();

    printf(sprintf(
        'Number of customer records inserted is: %d %s',
        $res->stats()['rowCountExact'],
        PHP_EOL
    ));
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def create_sequence(instance_id, database_id):
    """Creates the Sequence and insert data"""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE",
            """CREATE TABLE Customers (
        CustomerId  BIGINT DEFAULT nextval('Seq'),
        CustomerName  character varying(1024),
        PRIMARY KEY (CustomerId)
        )""",
        ],
    )
    operation = database_admin_api.update_database_ddl(request)
    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Created Seq sequence and Customers table, where the key column CustomerId uses the sequence as a default value on database {} on instance {}".format(
            database_id, instance_id
        )
    )

    def insert_customers(transaction):
        results = transaction.execute_sql(
            "INSERT INTO Customers (CustomerName) VALUES "
            "('Alice'), "
            "('David'), "
            "('Marc') "
            "RETURNING CustomerId"
        )
        for result in results:
            print("Inserted customer record with Customer Id: {}".format(*result))
        print(
            "Number of customer records inserted is {}".format(
                results.stats.row_count_exact
            )
        )

    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    database.run_in_transaction(insert_customers)
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to create a sequence using postgresql.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_postgresql_create_sequence project_id:, instance_id:, database_id:
  db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

  database_path = db_admin_client.database_path project: project_id,
                                                instance: instance_id,
                                                database: database_id


  job = db_admin_client.update_database_ddl database: database_path, statements: [
    "CREATE SEQUENCE Seq BIT_REVERSED_POSITIVE",
    "CREATE TABLE Customers (CustomerId BIGINT DEFAULT nextval('Seq'), CustomerName character varying(1024), PRIMARY KEY (CustomerId))"
  ]

  puts "Waiting for operation to complete..."
  job.wait_until_done!
  puts "Created Seq sequence and Customers table, where its key column CustomerId uses the sequence as a default value"
end
```

## Alter a sequence

The following code example alters the sequence `  Seq  ` to skip a value range from 1,000 to 5 million. It then inserts three new rows into the `  Customers  ` table.

### GoogleSQL

### C++

``` cpp
void AlterSequence(
    google::cloud::spanner_admin::DatabaseAdminClient admin_client,
    google::cloud::spanner::Client client, std::string const& project_id,
    std::string const& instance_id, std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  std::vector<std::string> statements;
  statements.emplace_back(R"""(
      ALTER SEQUENCE Seq
          SET OPTIONS (skip_range_min = 1000, skip_range_max = 5000000)
  )""");
  auto metadata =
      admin_client.UpdateDatabaseDdl(database.FullName(), std::move(statements))
          .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Altered `Seq` sequence"
            << "  to skip an inclusive range between 1000 and 5000000,"
            << " new DDL:\n"
            << metadata->DebugString();
  auto commit = client.Commit(
      [&client](google::cloud::spanner::Transaction txn)
          -> google::cloud::StatusOr<google::cloud::spanner::Mutations> {
        auto sql = google::cloud::spanner::SqlStatement(R"""(
            INSERT INTO Customers (CustomerName)
              VALUES ('Lea'),
                     ('Catalina'),
                     ('Smith')
              THEN RETURN CustomerId
        )""");
        using RowType = std::tuple<std::int64_t>;
        auto rows = client.ExecuteQuery(std::move(txn), std::move(sql));
        // Note: This mutator might be re-run, or its effects discarded, so
        // changing non-transactional state (e.g., by producing output) is,
        // in general, not something to be imitated.
        for (auto& row : google::cloud::spanner::StreamOf<RowType>(rows)) {
          if (!row) return std::move(row).status();
          std::cout << "Inserted customer record with CustomerId: "
                    << std::get<0>(*row) << "\n";
        }
        std::cout << "Number of customer records inserted is: "
                  << rows.RowsModified() << "\n";
        return google::cloud::spanner::Mutations{};
      });
  if (!commit) throw std::move(commit).status();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class AlterSequenceSample
{
    public async Task<List<long>> AlterSequenceSampleAsync(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        string[] statements =
        {
            "ALTER SEQUENCE Seq SET OPTIONS (skip_range_min = 1000, skip_range_max = 5000000)"
        };
        var operation = await databaseAdminClient.UpdateDatabaseDdlAsync(databaseName, statements);

        var completedResponse = await operation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            throw completedResponse.Exception;
        }
        Console.WriteLine("Altered Seq sequence to skip an inclusive range between 1000 and 5000000");

        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand(
            @"INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') THEN RETURN CustomerId");

        var reader = await cmd.ExecuteReaderAsync();
        var customerIds = new List<long>();
        while (await reader.ReadAsync())
        {
            long customerId = reader.GetFieldValue<long>("CustomerId");
            Console.WriteLine($"Inserted customer record with CustomerId: {customerId}");
            customerIds.Add(customerId);
        }
        Console.WriteLine($"Number of customer records inserted is: {customerIds.Count}");

        return customerIds;
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
 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/api/iterator"
)

func alterSequence(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // List of DDL statements to be applied to the database.
 // Alter the sequence to skip range [1000-5000000] for new keys.
 ddl := []string{
     "ALTER SEQUENCE Seq SET OPTIONS (skip_range_min = 1000, skip_range_max = 5000000)",
 }
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database:   db,
     Statements: ddl,
 })
 if err != nil {
     return err
 }
 // Wait for the UpdateDatabaseDdl operation to finish.
 if err := op.Wait(ctx); err != nil {
     return fmt.Errorf("waiting for bit reverse sequence skip range to finish failed: %w", err)
 }
 fmt.Fprintf(w, "Altered Seq sequence to skip an inclusive range between 1000 and 5000000\n")

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Inserts records into the Customers table.
 // The ReadWriteTransaction function returns the commit timestamp and an error.
 // The commit timestamp is ignored in this case.
 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT INTO Customers (CustomerName) VALUES ('Lea'), ('Catalina'), ('Smith') THEN RETURN CustomerId`,
     }
     iter := txn.Query(ctx, stmt)
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             break
         }
         if err != nil {
             return err
         }
         var customerId int64
         if err := row.Columns(&customerId); err != nil {
             return err
         }
         fmt.Fprintf(w, "Inserted customer record with CustomerId: %d\n", customerId)
     }
     fmt.Fprintf(w, "Number of customer records inserted is: %d\n", iter.RowCount)
     return nil
 })
 return err
}
```

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.Objects;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class AlterSequenceSample {

  static void alterSequence() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    alterSequence(projectId, instanceId, databaseId);
  }

  static void alterSequence(String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {

      databaseAdminClient
          .updateDatabaseDdlAsync(DatabaseName.of(projectId, instanceId, databaseId),
              ImmutableList.of(
                  "ALTER SEQUENCE Seq SET OPTIONS "
                      + "(skip_range_min = 1000, skip_range_max = 5000000)"))
          .get(5, TimeUnit.MINUTES);

      System.out.println(
          "Altered Seq sequence to skip an inclusive range between 1000 and 5000000");

      final DatabaseClient dbClient =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));

      Long insertCount =
          dbClient
              .readWriteTransaction()
              .run(
                  transaction -> {
                    try (ResultSet rs =
                        transaction.executeQuery(
                            Statement.of(
                                "INSERT INTO Customers (CustomerName) VALUES "
                                    + "('Lea'), ('Catalina'), ('Smith') "
                                    + "THEN RETURN CustomerId"))) {
                      while (rs.next()) {
                        System.out.printf(
                            "Inserted customer record with CustomerId: %d\n", rs.getLong(0));
                      }
                      return Objects.requireNonNull(rs.getStats()).getRowCountExact();
                    }
                  });
      System.out.printf("Number of customer records inserted is: %d\n", insertCount);
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw SpannerExceptionFactory.asSpannerException(e.getCause());
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    } catch (TimeoutException e) {
      // If the operation timed out propagate the timeout
      throw SpannerExceptionFactory.propagateTimeout(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library.
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

async function alterSequence(instanceId, databaseId) {
  // Gets a reference to a Cloud Spanner Database Admin Client object
  const databaseAdminClient = spanner.getDatabaseAdminClient();

  const request = [
    'ALTER SEQUENCE Seq SET OPTIONS (skip_range_min = 1000, skip_range_max = 5000000)',
  ];

  try {
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log(
      'Altered Seq sequence to skip an inclusive range between 1000 and 5000000.',
    );
  } catch (err) {
    console.error('ERROR:', err);
  }

  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  database.runTransaction(async (err, transaction) => {
    if (err) {
      console.error(err);
      return;
    }
    try {
      const [rows, stats] = await transaction.run({
        sql: "INSERT INTO Customers (CustomerName) VALUES ('Lea'), ('Catalina'), ('Smith') THEN RETURN CustomerId",
      });

      rows.forEach(row => {
        console.log(
          `Inserted customer record with CustomerId: ${
            row.toJSON({wrapNumbers: true}).CustomerId.value
          }`,
        );
      });

      const rowCount = Math.floor(stats[stats.rowCount]);
      console.log(`Number of customer records inserted is: ${rowCount}`);

      await transaction.commit();
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the database when finished.
      await database.close();
    }
  });
}
await alterSequence(instanceId, databaseId);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;
use Google\Cloud\Spanner\Result;
use Google\Cloud\Spanner\SpannerClient;

/**
 * Alters a sequence.
 * Example:
 * ```
 * alter_sequence($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function alter_sequence(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $spanner = new SpannerClient();

    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);
    $transaction = $database->transaction();

    $statements = [
         'ALTER SEQUENCE Seq SET OPTIONS ' .
        '(skip_range_min = 1000, skip_range_max = 5000000)'
    ];
    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => $statements
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf(
        'Altered Seq sequence to skip an inclusive range between 1000 and 5000000' .
        PHP_EOL
    );

    $res = $transaction->execute(
        'INSERT INTO Customers (CustomerName) VALUES ' .
        "('Lea'), ('Catalina'), ('Smith') THEN RETURN CustomerId"
    );
    $rows = $res->rows(Result::RETURN_ASSOCIATIVE);

    foreach ($rows as $row) {
        printf('Inserted customer record with CustomerId: %d %s',
            $row['CustomerId'],
            PHP_EOL
        );
    }
    $transaction->commit();

    printf(sprintf(
        'Number of customer records inserted is: %d %s',
        $res->stats()['rowCountExact'],
        PHP_EOL
    ));
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def alter_sequence(instance_id, database_id):
    """Alters the Sequence and insert data"""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "ALTER SEQUENCE Seq SET OPTIONS (skip_range_min = 1000, skip_range_max = 5000000)",
        ],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Altered Seq sequence to skip an inclusive range between 1000 and 5000000 on database {} on instance {}".format(
            database_id, instance_id
        )
    )

    def insert_customers(transaction):
        results = transaction.execute_sql(
            "INSERT INTO Customers (CustomerName) VALUES "
            "('Lea'), "
            "('Cataline'), "
            "('Smith') "
            "THEN RETURN CustomerId"
        )
        for result in results:
            print("Inserted customer record with Customer Id: {}".format(*result))
        print(
            "Number of customer records inserted is {}".format(
                results.stats.row_count_exact
            )
        )

    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    database.run_in_transaction(insert_customers)
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to alter a sequence.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_alter_sequence project_id:, instance_id:, database_id:
  db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

  database_path = db_admin_client.database_path project: project_id,
                                                instance: instance_id,
                                                database: database_id


  job = db_admin_client.update_database_ddl database: database_path, statements: [
    "ALTER SEQUENCE Seq SET OPTIONS (skip_range_min = 1000, skip_range_max = 5000000)"
  ]

  puts "Waiting for operation to complete..."
  job.wait_until_done!
  puts "Altered Seq sequence to skip an inclusive range between 1000 and 5000000"
end
```

### PostgreSQL

### C++

``` cpp
void AlterSequence(
    google::cloud::spanner_admin::DatabaseAdminClient admin_client,
    google::cloud::spanner::Database const& database,
    google::cloud::spanner::Client client) {
  std::vector<std::string> statements;
  statements.emplace_back(R"""(
      ALTER SEQUENCE Seq SKIP RANGE 1000 5000000
  )""");
  auto metadata =
      admin_client.UpdateDatabaseDdl(database.FullName(), std::move(statements))
          .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Altered `Seq` sequence"
            << "  to skip an inclusive range between 1000 and 5000000,"
            << " new DDL:\n"
            << metadata->DebugString();
  auto commit = client.Commit(
      [&client](google::cloud::spanner::Transaction txn)
          -> google::cloud::StatusOr<google::cloud::spanner::Mutations> {
        auto sql = google::cloud::spanner::SqlStatement(R"""(
            INSERT INTO Customers (CustomerName)
                VALUES ('Lea'),
                       ('Catalina'),
                       ('Smith')
                RETURNING CustomerId
        )""");
        using RowType = std::tuple<std::int64_t>;
        auto rows = client.ExecuteQuery(std::move(txn), std::move(sql));
        // Note: This mutator might be re-run, or its effects discarded, so
        // changing non-transactional state (e.g., by producing output) is,
        // in general, not something to be imitated.
        for (auto& row : google::cloud::spanner::StreamOf<RowType>(rows)) {
          if (!row) return std::move(row).status();
          std::cout << "Inserted customer record with CustomerId: "
                    << std::get<0>(*row) << "\n";
        }
        std::cout << "Number of customer records inserted is: "
                  << rows.RowsModified() << "\n";
        return google::cloud::spanner::Mutations{};
      });
  if (!commit) throw std::move(commit).status();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.Cloud.Spanner.Data;
using Google.LongRunning;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class AlterSequencePostgresqlSample
{
    public async Task<List<long>> AlterSequencePostgresqlSampleAsync(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();
        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        string[] statements =
        {
            "ALTER SEQUENCE Seq SKIP RANGE 1000 5000000;"
        };
        var operation = await databaseAdminClient.UpdateDatabaseDdlAsync(databaseName, statements);
        var completedResponse = await operation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            throw completedResponse.Exception;
        }
        Console.WriteLine("Altered Seq sequence to skip an inclusive range between 1000 and 5000000");

        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand(
            @"INSERT INTO Customers (CustomerName) VALUES ('Alice'), ('David'), ('Marc') RETURNING CustomerId");

        var reader = await cmd.ExecuteReaderAsync();
        var customerIds = new List<long>();
        while (await reader.ReadAsync())
        {
            var customerId = reader.GetFieldValue<long>("customerid");
            Console.WriteLine($"Inserted customer record with CustomerId: {customerId}");
            customerIds.Add(customerId);
        }
        Console.WriteLine($"Number of customer records inserted is: {customerIds.Count}");
        return customerIds;
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
 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/api/iterator"
)

func pgAlterSequence(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // List of DDL statements to be applied to the database.
 // Alter the sequence to skip range [1000-5000000] for new keys.
 ddl := []string{
     "ALTER SEQUENCE Seq SKIP RANGE 1000 5000000",
 }
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database:   db,
     Statements: ddl,
 })
 if err != nil {
     return err
 }
 // Wait for the UpdateDatabaseDdl operation to finish.
 if err := op.Wait(ctx); err != nil {
     return fmt.Errorf("waiting for bit reverse sequence skip range to finish failed: %w", err)
 }
 fmt.Fprintf(w, "Altered Seq sequence to skip an inclusive range between 1000 and 5000000\n")

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Inserts records into the Customers table.
 // The ReadWriteTransaction function returns the commit timestamp and an error.
 // The commit timestamp is ignored in this case.
 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT INTO Customers (CustomerName) VALUES ('Lea'), ('Catalina'), ('Smith') RETURNING CustomerId`,
     }
     iter := txn.Query(ctx, stmt)
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             break
         }
         if err != nil {
             return err
         }
         var customerId int64
         if err := row.Columns(&customerId); err != nil {
             return err
         }
         fmt.Fprintf(w, "Inserted customer record with CustomerId: %d\n", customerId)
     }
     fmt.Fprintf(w, "Number of customer records inserted is: %d\n", iter.RowCount)
     return nil
 })
 return err
}
```

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.Objects;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class PgAlterSequenceSample {

  static void pgAlterSequence() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    pgAlterSequence(projectId, instanceId, databaseId);
  }

  static void pgAlterSequence(String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {

      databaseAdminClient
          .updateDatabaseDdlAsync(
              DatabaseName.of(projectId, instanceId, databaseId),
              ImmutableList.of("ALTER SEQUENCE Seq SKIP RANGE 1000 5000000"))
          .get(5, TimeUnit.MINUTES);
      System.out.println(
          "Altered Seq sequence to skip an inclusive range between 1000 and 5000000");
      final DatabaseClient dbClient =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      Long insertCount =
          dbClient
              .readWriteTransaction()
              .run(
                  transaction -> {
                    try (ResultSet rs =
                        transaction.executeQuery(
                            Statement.of(
                                "INSERT INTO Customers (CustomerName) VALUES "
                                    + "('Lea'), ('Catalina'), ('Smith') RETURNING CustomerId"))) {
                      while (rs.next()) {
                        System.out.printf(
                            "Inserted customer record with CustomerId: %d\n", rs.getLong(0));
                      }
                      return Objects.requireNonNull(rs.getStats()).getRowCountExact();
                    }
                  });
      System.out.printf("Number of customer records inserted is: %d\n", insertCount);
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw SpannerExceptionFactory.asSpannerException(e.getCause());
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    } catch (TimeoutException e) {
      // If the operation timed out propagate the timeout
      throw SpannerExceptionFactory.propagateTimeout(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library.
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

async function alterSequence(instanceId, databaseId) {
  // Gets a reference to a Cloud Spanner Database Admin Client object
  const databaseAdminClient = spanner.getDatabaseAdminClient();

  const request = ['ALTER SEQUENCE Seq SKIP RANGE 1000 5000000'];

  try {
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log(
      'Altered Seq sequence to skip an inclusive range between 1000 and 5000000.',
    );
  } catch (err) {
    console.error('ERROR:', err);
  }

  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  database.runTransaction(async (err, transaction) => {
    if (err) {
      console.error(err);
      return;
    }
    try {
      const [rows, stats] = await transaction.run({
        sql: "INSERT INTO Customers (CustomerName) VALUES ('Lea'), ('Catalina'), ('Smith') RETURNING CustomerId",
      });

      rows.forEach(row => {
        console.log(
          `Inserted customer record with CustomerId: ${
            row.toJSON({wrapNumbers: true}).customerid.value
          }`,
        );
      });

      const rowCount = Math.floor(stats[stats.rowCount]);
      console.log(`Number of customer records inserted is: ${rowCount}`);

      await transaction.commit();
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the spanner client when finished.
      // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
      spanner.close();
    }
  });
}
await alterSequence(instanceId, databaseId);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Result;

/**
 * Alters a sequence.
 * Example:
 * ```
 * pg_alter_sequence($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function pg_alter_sequence(
    string $projectId,
    string $instanceId,
    string $databaseId
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);
    $transaction = $database->transaction();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $statement = 'ALTER SEQUENCE Seq SKIP RANGE 1000 5000000';
    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [$statement]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf(
        'Altered Seq sequence to skip an inclusive range between 1000 and 5000000' .
        PHP_EOL
    );

    $res = $transaction->execute(
        'INSERT INTO Customers (CustomerName) VALUES ' .
        "('Lea'), ('Catalina'), ('Smith') RETURNING CustomerId"
    );
    $rows = $res->rows(Result::RETURN_ASSOCIATIVE);

    foreach ($rows as $row) {
        printf('Inserted customer record with CustomerId: %d %s',
            $row['customerid'],
            PHP_EOL
        );
    }
    $transaction->commit();

    printf(sprintf(
        'Number of customer records inserted is: %d %s',
        $res->stats()['rowCountExact'],
        PHP_EOL
    ));
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def alter_sequence(instance_id, database_id):
    """Alters the Sequence and insert data"""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=["ALTER SEQUENCE Seq SKIP RANGE 1000 5000000"],
    )
    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Altered Seq sequence to skip an inclusive range between 1000 and 5000000 on database {} on instance {}".format(
            database_id, instance_id
        )
    )

    def insert_customers(transaction):
        results = transaction.execute_sql(
            "INSERT INTO Customers (CustomerName) VALUES "
            "('Lea'), "
            "('Cataline'), "
            "('Smith') "
            "RETURNING CustomerId"
        )
        for result in results:
            print("Inserted customer record with Customer Id: {}".format(*result))
        print(
            "Number of customer records inserted is {}".format(
                results.stats.row_count_exact
            )
        )

    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    database.run_in_transaction(insert_customers)
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to alter a sequence using postgresql.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_postgresql_alter_sequence project_id:, instance_id:, database_id:
  db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

  database_path = db_admin_client.database_path project: project_id,
                                                instance: instance_id,
                                                database: database_id


  job = db_admin_client.update_database_ddl database: database_path, statements: [
    "ALTER SEQUENCE Seq SKIP RANGE 1000 5000000"
  ]

  puts "Waiting for operation to complete..."
  job.wait_until_done!
  puts "Altered Seq sequence to skip an inclusive range between 1000 and 5000000"
end
```

## Drop a sequence

The following code example alters the table `  Customers  ` to remove the sequence `  Seq  ` from the primary key default value, and then drops the sequence `  Seq  ` .

### GoogleSQL

### C++

``` cpp
void DropSequence(
    google::cloud::spanner_admin::DatabaseAdminClient admin_client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  std::vector<std::string> statements;
  statements.emplace_back(R"""(
      ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT
  )""");
  statements.emplace_back(R"""(
      DROP SEQUENCE Seq
  )""");
  auto metadata =
      admin_client.UpdateDatabaseDdl(database.FullName(), std::move(statements))
          .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Altered `Customers` table to"
            << " drop DEFAULT from `CustomerId` column,"
            << " and dropped the `Seq` sequence,"
            << " new DDL:\n"
            << metadata->DebugString();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Threading.Tasks;

public class DropSequenceSample
{
    public async Task DropSequenceSampleAsync(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        string[] statements =
        {
            "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
            "DROP SEQUENCE Seq"
        };
        var operation = await databaseAdminClient.UpdateDatabaseDdlAsync(databaseName, statements);

        var completedResponse = await operation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            throw completedResponse.Exception;
        }
        Console.WriteLine("Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence");
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func dropSequence(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // List of DDL statements to be applied to the database.
 // Drop the DEFAULT from CustomerId column and drop the sequence.
 ddl := []string{
     "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
     "DROP SEQUENCE Seq",
 }
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database:   db,
     Statements: ddl,
 })
 if err != nil {
     return err
 }
 // Wait for the UpdateDatabaseDdl operation to finish.
 if err := op.Wait(ctx); err != nil {
     return fmt.Errorf("waiting for bit reverse sequence drop to finish failed: %w", err)
 }
 fmt.Fprintf(w, "Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence\n")
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class DropSequenceSample {

  static void dropSequence() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    dropSequence(projectId, instanceId, databaseId);
  }

  static void dropSequence(String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      databaseAdminClient
          .updateDatabaseDdlAsync(DatabaseName.of(projectId, instanceId, databaseId),
              ImmutableList.of(
                  "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
                  "DROP SEQUENCE Seq"))
          .get(5, TimeUnit.MINUTES);
      System.out.println(
          "Altered Customers table to drop DEFAULT from CustomerId column "
              + "and dropped the Seq sequence");
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw SpannerExceptionFactory.asSpannerException(e.getCause());
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    } catch (TimeoutException e) {
      // If the operation timed out propagate the timeout
      throw SpannerExceptionFactory.propagateTimeout(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library.
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

async function dropSequence(instanceId, databaseId) {
  // Gets a reference to a Cloud Spanner Database Admin Client object
  const databaseAdminClient = spanner.getDatabaseAdminClient();

  const request = [
    'ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT',
    'DROP SEQUENCE Seq',
  ];

  // Drop sequence from DDL
  try {
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log(
      'Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence.',
    );
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the spanner client when finished.
    // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
    spanner.close();
  }
}
await dropSequence(instanceId, databaseId);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Drops a sequence.
 * Example:
 * ```
 * drop_sequence($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function drop_sequence(
    string $projectId,
    string $instanceId,
    string $databaseId
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [
            'ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT',
            'DROP SEQUENCE Seq'
        ]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf(
        'Altered Customers table to drop DEFAULT from CustomerId ' .
        'column and dropped the Seq sequence' .
        PHP_EOL
    );
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def drop_sequence(instance_id, database_id):
    """Drops the Sequence"""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
            "DROP SEQUENCE Seq",
        ],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence on database {} on instance {}".format(
            database_id, instance_id
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to drop a sequence.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_drop_sequence project_id:, instance_id:, database_id:
  db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

  database_path = db_admin_client.database_path project: project_id,
                                                instance: instance_id,
                                                database: database_id


  job = db_admin_client.update_database_ddl database: database_path, statements: [
    "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
    "DROP SEQUENCE Seq"
  ]

  puts "Waiting for operation to complete..."
  job.wait_until_done!
  puts "Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence"
end
```

### PostgreSQL

### C++

``` cpp
void DropSequence(
    google::cloud::spanner_admin::DatabaseAdminClient admin_client,
    google::cloud::spanner::Database const& database) {
  std::vector<std::string> statements;
  statements.emplace_back(R"""(
      ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT
  )""");
  statements.emplace_back(R"""(
      DROP SEQUENCE Seq
  )""");
  auto metadata =
      admin_client.UpdateDatabaseDdl(database.FullName(), std::move(statements))
          .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Altered `Customers` table to"
            << " drop DEFAULT from `CustomerId` column,"
            << " and dropped the `Seq` sequence,"
            << " new DDL:\n"
            << metadata->DebugString();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Threading.Tasks;

public class DropSequenceSample
{
    public async Task DropSequenceSampleAsync(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        string[] statements =
        {
            "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
            "DROP SEQUENCE Seq"
        };
        var operation = await databaseAdminClient.UpdateDatabaseDdlAsync(databaseName, statements);

        var completedResponse = await operation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            throw completedResponse.Exception;
        }
        Console.WriteLine("Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence");
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func dropSequence(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // List of DDL statements to be applied to the database.
 // Drop the DEFAULT from CustomerId column and drop the sequence.
 ddl := []string{
     "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
     "DROP SEQUENCE Seq",
 }
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database:   db,
     Statements: ddl,
 })
 if err != nil {
     return err
 }
 // Wait for the UpdateDatabaseDdl operation to finish.
 if err := op.Wait(ctx); err != nil {
     return fmt.Errorf("waiting for bit reverse sequence drop to finish failed: %w", err)
 }
 fmt.Fprintf(w, "Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence\n")
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class PgDropSequenceSample {

  static void pgDropSequence() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    pgDropSequence(projectId, instanceId, databaseId);
  }

  static void pgDropSequence(String projectId, String instanceId, String databaseId) {

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      databaseAdminClient
          .updateDatabaseDdlAsync(
              DatabaseName.of(projectId, instanceId, databaseId),
              ImmutableList.of(
                  "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
                  "DROP SEQUENCE Seq"))
          .get(5, TimeUnit.MINUTES);
      System.out.println(
          "Altered Customers table to drop DEFAULT from "
              + "CustomerId column and dropped the Seq sequence");
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw SpannerExceptionFactory.asSpannerException(e.getCause());
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    } catch (TimeoutException e) {
      // If the operation timed out propagate the timeout
      throw SpannerExceptionFactory.propagateTimeout(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library.
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

async function dropSequence(instanceId, databaseId) {
  // Gets a reference to a Cloud Spanner Database Admin Client object
  const databaseAdminClient = spanner.getDatabaseAdminClient();

  const request = [
    'ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT',
    'DROP SEQUENCE Seq',
  ];

  // Drop sequence from DDL
  try {
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log(
      'Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence.',
    );
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the spanner client when finished.
    // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
    spanner.close();
  }
}
await dropSequence(instanceId, databaseId);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Drops a sequence.
 * Example:
 * ```
 * pg_drop_sequence($instanceId, $databaseId);
 * ```
 *
 * @param string $projectId Your Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function pg_drop_sequence(
    string $projectId,
    string $instanceId,
    string $databaseId
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $statements = [
        'ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT',
        'DROP SEQUENCE Seq'
    ];

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => $statements
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf(
        'Altered Customers table to drop DEFAULT from CustomerId ' .
        'column and dropped the Seq sequence' .
        PHP_EOL
    );
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def drop_sequence(instance_id, database_id):
    """Drops the Sequence"""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
            "DROP SEQUENCE Seq",
        ],
    )
    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence on database {} on instance {}".format(
            database_id, instance_id
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to drop a sequence using postgresql.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_postgresql_drop_sequence project_id:, instance_id:, database_id:
  db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

  database_path = db_admin_client.database_path project: project_id,
                                                instance: instance_id,
                                                database: database_id


  job = db_admin_client.update_database_ddl database: database_path, statements: [
    "ALTER TABLE Customers ALTER COLUMN CustomerId DROP DEFAULT",
    "DROP SEQUENCE Seq"
  ]

  puts "Waiting for operation to complete..."
  job.wait_until_done!
  puts "Altered Customers table to drop DEFAULT from CustomerId column and dropped the Seq sequence"
end
```
