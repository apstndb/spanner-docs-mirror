To help you better understand, optimize, and diagnose transaction issues, Spanner gives you access to transaction commit statistics. Currently, you can retrieve the total number of mutations for a transaction.

## When to use commit statistics

Knowing the mutation count for a transaction can be useful in the following scenarios.

### Optimize for round trips

To help improve the performance of your application you can reduce the number of round trips to the database by doing as much work as possible in each transaction. In this scenario you want to maximize the number of mutations per transaction, while at the same time staying within system [limits](/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) .

To determine how many rows you can commit per transaction while staying under the limit, first commit one row in a transaction. This gives you a baseline of the mutation count per row. Then divide the system limit by your baseline to get a rows-per-transaction number. For more information on how mutations are counted, refer to this [note](/spanner/quotas#note2) .

Note that optimizing for round trips is not always beneficial, particularly if it results in more lock contentions. You can troubleshoot lock conflicts in your database using [lock statistics](/spanner/docs/introspection/lock-statistics) .

### Monitor your transactions to avoid hitting system limits

As application usage increases, it's possible that the number of mutations in your transaction also grows. To avoid hitting the system limit and having your transaction eventually fail, you can proactively monitor the mutation count commit statistic over time. If you observe this value increasing for the same transaction, it might be time to re-optimize your transaction as described in the preceding section.

## How to access commit statistics

Commit statistics are not returned by default. Instead, you need to set the `  return_commit_stats  ` flag to true on each [CommitRequest](/spanner/docs/reference/rpc/google.spanner.v1#commitrequest) . If your commit attempt exceeds the maximum allowable number of mutations for a transaction, the commit fails and an [INVALID\_ARGUMENT](/spanner/docs/reference/rest/v1/Code#ENUM_VALUES.INVALID_ARGUMENT) error is returned.

Here's an example of how to return commit statistics using the Spanner client libraries.

### Retrieve commit statistics

The following sample shows how to get commit statistics using the Spanner client libraries.

### C++

The following code calls `  set_return_stats()  ` on `  CommitOptions  ` and returns a mutation count of 6, because we are inserting or updating 2 rows and 3 columns in each row.

``` cpp
void GetCommitStatistics(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  auto commit = client.Commit(
      spanner::Mutations{
          spanner::UpdateMutationBuilder(
              "Albums", {"SingerId", "AlbumId", "MarketingBudget"})
              .EmplaceRow(1, 1, 200000)
              .EmplaceRow(2, 2, 400000)
              .Build()},
      google::cloud::Options{}.set<spanner::CommitReturnStatsOption>(true));

  if (!commit) throw std::move(commit).status();
  if (commit->commit_stats) {
    std::cout << "Updated data with " << commit->commit_stats->mutation_count
              << " mutations.\n";
  }
  std::cout << "Update was successful [spanner_get_commit_stats]\n";
}
```

### C\#

In C\#, commit statistics are not returned directly through the API. Instead, they are logged at the **Information** log level by the default logger.

The following code enables commit statistics logging for all transactions by setting the `  LogCommitStats  ` property on `  SpannerConnectionStringBuilder  ` to true. The code also implements a sample logger that keeps a reference to the last seen commit response. The `  MutationCount  ` is then retrieved from this response and displayed.

``` csharp
using Google.Cloud.Spanner.Data;
using Google.Cloud.Spanner.V1;
using Google.Cloud.Spanner.V1.Internal.Logging;
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Threading.Tasks;

public class LogCommitStatsAsyncSample
{
    public async Task<long> LogCommitStatsAsync(string projectId, string instanceId, string databaseId)
    {
        // Commit statistics are logged at level Info by the default logger.
        // This sample uses a custom logger to access the commit statistics.
        // See https://googleapis.github.io/google-cloud-dotnet/docs/Google.Cloud.Spanner.Data/logging.html
        // for more information on how to use loggers.
        var logger = new CommitStatsSampleLogger();
        var options = new SessionPoolOptions();
        var poolManager = SessionPoolManager.Create(options, logger);
        var connectionStringBuilder = new SpannerConnectionStringBuilder
        {
            ConnectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}",
            // Set LogCommitStats to true to enable logging commit statistics for all transactions on the connection.
            // LogCommitStats can also be enabled/disabled for individual Spanner transactions.
            LogCommitStats = true,
            SessionPoolManager = poolManager,
        };

        using var connection = new SpannerConnection(connectionStringBuilder);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("INSERT Singers (SingerId, FirstName, LastName) VALUES (110, 'Virginia', 'Watson')");
        var rowCount = await cmd.ExecuteNonQueryAsync();
        var mutationCount = logger._lastCommitResponse.CommitStats.MutationCount;

        Console.WriteLine($"{rowCount} row(s) inserted...");
        Console.WriteLine($"{mutationCount} mutation(s) in transaction...");

        return mutationCount;
    }

    /// <summary>
    /// Sample logger that keeps a reference to the last seen commit response.
    /// Use the default logger if you only want to log the commit stats.
    /// </summary>
    public class CommitStatsSampleLogger : Logger
    {
        internal CommitResponse _lastCommitResponse;

        /// <summary>
        /// This method is called when a transaction that requested commit stats is committed.
        /// </summary>
        public override void LogCommitStats(CommitRequest request, CommitResponse response)
        {
            _lastCommitResponse = response;
            base.LogCommitStats(request, response);
        }

        protected override void LogImpl(LogLevel level, string message, Exception exception) =>
            WriteLine(exception == null ? $"{level}: {message}" : $"{level}: {message}, Exception: {exception}");

        protected override void LogPerformanceEntries(IEnumerable<string> entries)
        {
            string separator = Environment.NewLine + "  ";
            WriteLine($"Performance:{separator}{string.Join(separator, entries)}");
        }

        private void WriteLine(string line) => Trace.TraceInformation(line);
    }
}
```

### Go

The following code sets the `  ReturnCommitStats  ` flag and prints out the mutation count when the transaction is successfully committed.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func commitStats(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return fmt.Errorf("commitStats.NewClient: %w", err)
 }
 defer client.Close()

 resp, err := client.ReadWriteTransactionWithOptions(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT Singers (SingerId, FirstName, LastName)
                 VALUES (110, 'Virginia', 'Watson')`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
     return nil
 }, spanner.TransactionOptions{CommitOptions: spanner.CommitOptions{ReturnCommitStats: true}})
 if err != nil {
     return fmt.Errorf("commitStats.ReadWriteTransactionWithOptions: %w", err)
 }
 fmt.Fprintf(w, "%d mutations in transaction\n", resp.CommitStats.MutationCount)
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.CommitResponse;
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.Mutation;
import com.google.cloud.spanner.Options;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import java.util.Arrays;

public class GetCommitStatsSample {

  static void getCommitStats() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      final DatabaseClient databaseClient = spanner
          .getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      getCommitStats(databaseClient);
    }
  }

  static void getCommitStats(DatabaseClient databaseClient) {
    final CommitResponse commitResponse = databaseClient.writeWithOptions(Arrays.asList(
        Mutation.newInsertOrUpdateBuilder("Albums")
            .set("SingerId")
            .to("1")
            .set("AlbumId")
            .to("1")
            .set("MarketingBudget")
            .to("200000")
            .build(),
        Mutation.newInsertOrUpdateBuilder("Albums")
            .set("SingerId")
            .to("2")
            .set("AlbumId")
            .to("2")
            .set("MarketingBudget")
            .to("400000")
            .build()
    ), Options.commitStats());

    System.out.println(
        "Updated data with " + commitResponse.getCommitStats().getMutationCount() + " mutations.");
  }
}
```

### Node.js

The following code sets the `  returnCommitStats  ` flag and returns a mutation count of 6, because we are inserting or updating 2 rows and 3 columns in each row.

``` javascript
// Imports the Google Cloud client library.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client.
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database.
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// Instantiate Spanner table objects.
const albumsTable = database.table('Albums');

// Updates rows in the Venues table.
try {
  const [response] = await albumsTable.upsert(
    [
      {SingerId: '1', AlbumId: '1', MarketingBudget: '200000'},
      {SingerId: '2', AlbumId: '2', MarketingBudget: '400000'},
    ],
    {returnCommitStats: true},
  );
  console.log(
    `Updated data with ${response.commitStats.mutationCount} mutations.`,
  );
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
use Google\Cloud\Spanner\Transaction;

/**
 * Creates a database and tables for sample data.
 * Example:
 * ```
 * create_database($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function get_commit_stats(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $commitStats = $database->runTransaction(function (Transaction $t) {
        $t->updateBatch('Albums', [
            [
                'SingerId' => 1,
                'AlbumId' => 1,
                'MarketingBudget' => 200000,
            ],
            [
                'SingerId' => 2,
                'AlbumId' => 2,
                'MarketingBudget' => 400000,
            ]
        ]);
        $t->commit(['returnCommitStats' => true]);
        return $t->getCommitStats();
    });

    print('Updated data with ' . $commitStats['mutationCount'] . ' mutations.' . PHP_EOL);
}
````

### Python

Instead of returning commit statistics directly through the API, the Python client library logs them using `  stdout  ` at level `  Info  ` .

The following code enables commit statistics logging for all transactions by setting `  database.log_commit_stats = True  ` . The code also implements a sample logger that keeps a reference to the last seen commit response. The `  mutation_count  ` is then retrieved from this response and displayed.

``` python
def log_commit_stats(instance_id, database_id):
    """Inserts sample data using DML and displays the commit statistics."""

    # By default, commit statistics are logged via stdout at level Info.
    # This sample uses a custom logger to access the commit statistics.
    class CommitStatsSampleLogger(logging.Logger):
        def __init__(self):
            self.last_commit_stats = None
            super().__init__("commit_stats_sample")

        def info(self, msg, *args, **kwargs):
            if (
                "extra" in kwargs
                and kwargs["extra"]
                and "commit_stats" in kwargs["extra"]
            ):
                self.last_commit_stats = kwargs["extra"]["commit_stats"]
            super().info(msg, *args, **kwargs)

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id, logger=CommitStatsSampleLogger())
    database.log_commit_stats = True

    def insert_singers(transaction):
        row_ct = transaction.execute_update(
            "INSERT Singers (SingerId, FirstName, LastName) "
            " VALUES (110, 'Virginia', 'Watson')"
        )

        print("{} record(s) inserted.".format(row_ct))

    database.run_in_transaction(insert_singers)
    commit_stats = database.logger.last_commit_stats
    print("{} mutation(s) in transaction.".format(commit_stats.mutation_count))
```

### Ruby

The following code sets the `  return_commit_stats  ` flag and returns a mutation count of 6, because we are inserting or updating 2 rows and 3 columns in each row.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

records = [
  { SingerId: 1, AlbumId: 1, MarketingBudget: 200_000 },
  { SingerId: 2, AlbumId: 2, MarketingBudget: 400_000 }
]
commit_options = { return_commit_stats: true }
resp = client.upsert "Albums", records, commit_options: commit_options
puts "Updated data with #{resp.stats.mutation_count} mutations."
```

## What's next

  - [CommitStats REST reference](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#commitstats) .
  - [CommitStats gRPC reference](/spanner/docs/reference/rpc/google.spanner.v1#commitstats) .
