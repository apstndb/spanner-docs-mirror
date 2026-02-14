This page describes how to configure the maximum commit (write) delay time to optimize write throughput in Spanner.

## Overview

To ensure data consistency, Spanner sends write requests to all voting replicas in the database. This replication process can have computational overhead. For more information, see [Replication](/spanner/docs/replication#role-in-writes) .

Throughput optimized writes provide the option to amortize these computation costs by executing a group of writes together. To do this, Spanner introduces a small delay and collects a group of writes that need to be sent to the same voting participants. Executing writes in this way can provide substantial throughput improvements at the cost of slightly increased latency.

## Default behavior

If you don't set a commit delay time, Spanner might set a small delay for you if it thinks that will amortize the cost of your writes.

## Common use cases

You can manually set the delay time of your write requests depending on your application needs. You can also disable commit delays for applications that are highly latency sensitive by setting the maximum commit delay time to 0 ms.

If you have a latency tolerant application and want to optimize throughput, setting a longer commit delay time significantly improves throughput while incurring higher latency for each write. For example, if you are bulk loading a large amount of data and the application doesn't care about how quickly Spanner writes any individual data, then you can set the commit delay time to a longer value like 100 ms. We recommend that you start with a value of 100 ms, and then adjust up and down until the latency and throughput tradeoffs satisfy your needs. For most applications, a value between 20 ms and 100 ms works best.

If you have a latency sensitive application, Spanner's is also latency sensitive by default. If you have a spiky workload, Spanner may set a small delay. You can experiment with setting a value of 0 ms to determine if the reduced latency at the cost of increased throughput is reasonable for your application.

## Set mixed commit delay times

You can configure different max commit delay times on subsets of your writes. If you do this, Spanner uses the shortest delay time configured for the set of writes. However, we recommend picking a single value for most use cases as this results in more predictable behavior.

## Limitations

You can set a commit delay time between 0 and 500 ms. Setting commit delays higher than 500 ms results in an error.

## Set max commit delay on commit requests

The max commit delay parameter is part of the `  CommitRequest  ` method. You can access this method with the [RPC API](/spanner/docs/reference/rpc/google.spanner.v1#commitrequest) , [REST API](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit) , or using the Cloud Spanner client library.

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class CommitDelayAsyncSample
{
    public async Task<int> CommitDelayAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        return await connection.RunWithRetriableTransactionAsync(async transaction =>
        {
            transaction.TransactionOptions.MaxCommitDelay = TimeSpan.FromMilliseconds(100);

            using var insertSingerCmd = connection.CreateInsertCommand("Singers",
                new SpannerParameterCollection
                {
                    { "SingerId", SpannerDbType.Int64, 1 },
                    { "FirstName", SpannerDbType.String, "Marc" },
                    { "LastName", SpannerDbType.String, "Richards" }
                });
            insertSingerCmd.Transaction = transaction;
            int rowsInserted = await insertSingerCmd.ExecuteNonQueryAsync();

            using var insertAlbumCmd = connection.CreateInsertCommand("Albums",
                new SpannerParameterCollection
                {
                    { "SingerId", SpannerDbType.Int64, 1 },
                    { "AlbumId", SpannerDbType.Int64, 2 },
                    { "AlbumTitle", SpannerDbType.String, "Go, Go, Go" }
                });
            insertAlbumCmd.Transaction = transaction;
            rowsInserted += await insertAlbumCmd.ExecuteNonQueryAsync();

            return rowsInserted;
        });
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 "cloud.google.com/go/spanner"
)

func setMaxCommitDelay(w io.Writer, db string) error {
 // db is the fully-qualified database name of the form `projects/<project>/instances/<instance-id>/database/<database-id>`
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return fmt.Errorf("setMaxCommitDelay.NewClient: %w", err)
 }
 defer client.Close()

 commitDelay := 100 * time.Millisecond
 resp, err := client.ReadWriteTransactionWithOptions(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT Singers (SingerId, FirstName, LastName)
                 VALUES (111, 'Virginia', 'Watson')`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
     return nil
 }, spanner.TransactionOptions{CommitOptions: spanner.CommitOptions{MaxCommitDelay: &commitDelay, ReturnCommitStats: true}})
 if err != nil {
     return fmt.Errorf("setMaxCommitDelay.ReadWriteTransactionWithOptions: %w", err)
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
import java.time.Duration;
import java.util.Arrays;

public class SetMaxCommitDelaySample {

  static void setMaxCommitDelay() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      final DatabaseClient databaseClient = spanner
          .getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      setMaxCommitDelay(databaseClient);
    }
  }

  static void setMaxCommitDelay(DatabaseClient databaseClient) {
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
    ), Options.maxCommitDelay(Duration.ofMillis(100)));

    System.out.println(
        "Updated data with timestamp + " + commitResponse.getCommitTimestamp() + ".");
  }
}
```

### Node.js

``` javascript
const {Spanner, protos} = require('@google-cloud/spanner');

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

async function setMaxCommitDelay() {
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  database.runTransaction(async (err, transaction) => {
    if (err) {
      console.error(err);
      return;
    }
    try {
      const [rowCount] = await transaction.runUpdate({
        sql: 'INSERT Singers (SingerId, FirstName, LastName) VALUES (111, @firstName, @lastName)',
        params: {
          firstName: 'Virginia',
          lastName: 'Watson',
        },
      });

      console.log(
        `Successfully inserted ${rowCount} record into the Singers table.`,
      );

      await transaction.commit({
        maxCommitDelay: protos.google.protobuf.Duration({
          seconds: 0, // 0 seconds
          nanos: 100000000, // 100 milliseconds
        }),
      });
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the database when finished.
      database.close();
    }
  });
}
setMaxCommitDelay();
```

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

def insert_singers(transaction):
    row_ct = transaction.execute_update(
        "INSERT Singers (SingerId, FirstName, LastName) "
        " VALUES (111, 'Grace', 'Bennis')"
    )

    print("{} record(s) inserted.".format(row_ct))

database.run_in_transaction(
    insert_singers, max_commit_delay=datetime.timedelta(milliseconds=100)
)
```

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to pass max_commit_delay in  commit_options.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_set_max_commit_delay project_id:, instance_id:, database_id:
  # Instantiates a client
  spanner = Google::Cloud::Spanner.new project: project_id
  client  = spanner.client instance_id, database_id

  records = [
    { SingerId: 1, AlbumId: 1, MarketingBudget: 200_000 },
    { SingerId: 2, AlbumId: 2, MarketingBudget: 400_000 }
  ]
  # max_commit_delay is the amount of latency in millisecond, this request
  # is willing to incur in order to improve throughput.
  # The commit delay must be at least 0ms and at most 500ms.
  # Default value is nil.
  commit_options = {
    return_commit_stats: true,
    max_commit_delay: 100
  }
  resp = client.upsert "Albums", records, commit_options: commit_options
  puts "Updated data with #{resp.stats.mutation_count} mutations."
end
```

## Monitor write request latency

You can monitor Spanner CPU utilization and latency using the Google Cloud console. When you set a longer delay time for your write requests, expect to see [CPU utilization](/spanner/docs/cpu-utilization) potentially decrease, while latency increases. To learn about latency in Spanner requests, see [Capture and visualize Spanner API request latency](/spanner/docs/latency-points) .
