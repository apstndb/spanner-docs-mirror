Set custom timeout and retry for request settings.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Configure custom timeouts and retries](/spanner/docs/custom-timeout-and-retry)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
namespace spanner = ::google::cloud::spanner;
using ::google::cloud::StatusOr;
[](std::string const& project_id, std::string const& instance_id,
   std::string const& database_id) {
  // Use a truncated exponential backoff with jitter to wait between
  // retries:
  //   https://en.wikipedia.org/wiki/Exponential_backoff
  //   https://cloud.google.com/storage/docs/exponential-backoff
  auto client = spanner::Client(spanner::MakeConnection(
      spanner::Database(project_id, instance_id, database_id),
      google::cloud::Options{}
          .set<spanner::SpannerRetryPolicyOption>(
              std::make_shared<spanner::LimitedTimeRetryPolicy>(
                  /*maximum_duration=*/std::chrono::seconds(60)))
          .set<spanner::SpannerBackoffPolicyOption>(
              std::make_shared<spanner::ExponentialBackoffPolicy>(
                  /*initial_delay=*/std::chrono::milliseconds(500),
                  /*maximum_delay=*/std::chrono::seconds(16),
                  /*scaling=*/1.5))));

  std::int64_t rows_inserted;
  auto commit_result = client.Commit(
      [&client, &rows_inserted](
          spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto insert = client.ExecuteDml(
            std::move(txn),
            spanner::SqlStatement(
                "INSERT INTO Singers (SingerId, FirstName, LastName)"
                "  VALUES (20, 'George', 'Washington')"));
        if (!insert) return std::move(insert).status();
        rows_inserted = insert->RowsModified();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Rows inserted: " << rows_inserted;
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Api.Gax;
using Google.Api.Gax.Grpc;
using Google.Cloud.Spanner.Common.V1;
using Google.Cloud.Spanner.V1;
using Grpc.Core;
using System;
using System.Threading;
using System.Threading.Tasks;
using static Google.Cloud.Spanner.V1.TransactionOptions.Types;

public class CustomTimeoutsAndRetriesAsyncSample
{
    public async Task<long> CustomTimeoutsAndRetriesAsync(string projectId, string instanceId, string databaseId)
    {
        // Create a SessionPool.
        SpannerClient client = SpannerClient.Create();
        SessionPool sessionPool = new SessionPool(client, new SessionPoolOptions());

        // Acquire a session with a read-write transaction to run a query.
        DatabaseName databaseName =
            DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        TransactionOptions transactionOptions = new TransactionOptions
        {
            ReadWrite = new ReadWrite()
        };
        using PooledSession session = await sessionPool.AcquireSessionAsync(
            databaseName, transactionOptions, CancellationToken.None);

        ExecuteSqlRequest request = new ExecuteSqlRequest
        {
            Sql = "INSERT Singers (SingerId, FirstName, LastName) VALUES (20, 'George', 'Washington')"
        };

        // Prepare the call settings with custom timeout and retry settings.
        CallSettings settings = CallSettings
            .FromExpiration(Expiration.FromTimeout(TimeSpan.FromSeconds(60)))
            .WithRetry(RetrySettings.FromExponentialBackoff(
                maxAttempts: 12,
                initialBackoff: TimeSpan.FromMilliseconds(500),
                maxBackoff: TimeSpan.FromSeconds(16),
                backoffMultiplier: 1.5,
                retryFilter: RetrySettings.FilterForStatusCodes(
                    new StatusCode[] { StatusCode.Unavailable })));

        ResultSet result = await session.ExecuteSqlAsync(request, settings);
        await session.CommitAsync(new CommitRequest(), null);

        return result.Stats.RowCountExact;
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
 "time"

 "cloud.google.com/go/spanner"
 spapi "cloud.google.com/go/spanner/apiv1"
 gax "github.com/googleapis/gax-go/v2"
 "google.golang.org/grpc/codes"
)

func setCustomTimeoutAndRetry(w io.Writer, db string) error {
 // Set a custom retry setting.
 co := &spapi.CallOptions{
     ExecuteSql: []gax.CallOption{
         gax.WithRetry(func() gax.Retryer {
             return gax.OnCodes([]codes.Code{
                 codes.Unavailable,
             }, gax.Backoff{
                 Initial:    500 * time.Millisecond,
                 Max:        16000 * time.Millisecond,
                 Multiplier: 1.5,
             })
         }),
     },
 }
 client, err := spanner.NewClientWithConfig(context.Background(), db, spanner.ClientConfig{CallOptions: co})
 if err != nil {
     return err
 }
 defer client.Close()

 // Set timeout to 60s.
 ctx, cancel := context.WithTimeout(context.Background(), 60*time.Second)
 defer cancel()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT Singers (SingerId, FirstName, LastName)
                 VALUES (20, 'George', 'Washington')`,
     }

     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
     return nil
 })
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import com.google.api.gax.retrying.RetrySettings;
import com.google.api.gax.rpc.StatusCode.Code;
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import org.threeten.bp.Duration;

class CustomTimeoutAndRetrySettingsExample {

  static void executeSqlWithCustomTimeoutAndRetrySettings() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    executeSqlWithCustomTimeoutAndRetrySettings(projectId, instanceId, databaseId);
  }

  // Create a Spanner client with custom ExecuteSql timeout and retry settings.
  static void executeSqlWithCustomTimeoutAndRetrySettings(
      String projectId, String instanceId, String databaseId) {
    SpannerOptions.Builder builder = SpannerOptions.newBuilder().setProjectId(projectId);
    // Set custom timeout and retry settings for the ExecuteSql RPC.
    // This must be done in a separate chain as the setRetryableCodes and setRetrySettings methods
    // return a UnaryCallSettings.Builder instead of a SpannerOptions.Builder.
    builder
        .getSpannerStubSettingsBuilder()
        .executeSqlSettings()
        // Configure which errors should be retried.
        .setRetryableCodes(Code.UNAVAILABLE)
        .setRetrySettings(
            RetrySettings.newBuilder()
                // Configure retry delay settings.
                // The initial amount of time to wait before retrying the request.
                .setInitialRetryDelay(Duration.ofMillis(500))
                // The maximum amount of time to wait before retrying. I.e. after this value is
                // reached, the wait time will not increase further by the multiplier.
                .setMaxRetryDelay(Duration.ofSeconds(16))
                // The previous wait time is multiplied by this multiplier to come up with the next
                // wait time, until the max is reached.
                .setRetryDelayMultiplier(1.5)

                // Configure RPC and total timeout settings.
                // Timeout for the first RPC call. Subsequent retries will be based off this value.
                .setInitialRpcTimeout(Duration.ofSeconds(60))
                // The max for the per RPC timeout.
                .setMaxRpcTimeout(Duration.ofSeconds(60))
                // Controls the change of timeout for each retry.
                .setRpcTimeoutMultiplier(1.0)
                // The timeout for all calls (first call + all retries).
                .setTotalTimeout(Duration.ofSeconds(60))
                .build());
    // Create a Spanner client using the custom retry and timeout settings.
    try (Spanner spanner = builder.build().getService()) {
      DatabaseClient client =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      client
          .readWriteTransaction()
          .run(transaction -> {
            String sql =
                "INSERT INTO Singers (SingerId, FirstName, LastName)\n"
                    + "VALUES (20, 'George', 'Washington')";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d record inserted.%n", rowCount);
            return null;
          });
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
const UNAVAILABLE_STATUS_CODE = 14;
const retryAndTimeoutSettings = {
  retry: {
    // The set of error codes that will be retried.
    retryCodes: [UNAVAILABLE_STATUS_CODE],
    backoffSettings: {
      // Configure retry delay settings.
      // The initial amount of time to wait before retrying the request.
      initialRetryDelayMillis: 500,
      // The maximum amount of time to wait before retrying. I.e. after this
      // value is reached, the wait time will not increase further by the
      // multiplier.
      maxRetryDelayMillis: 16000,
      // The previous wait time is multiplied by this multiplier to come up
      // with the next wait time, until the max is reached.
      retryDelayMultiplier: 1.5,

      // Configure RPC and total timeout settings.
      // Timeout for the first RPC call. Subsequent retries will be based off
      // this value.
      initialRpcTimeoutMillis: 60000,
      // Controls the change of timeout for each retry.
      rpcTimeoutMultiplier: 1.0,
      // The max for the per RPC timeout.
      maxRpcTimeoutMillis: 60000,
      // The timeout for all calls (first call + all retries).
      totalTimeoutMillis: 60000,
    },
  },
};

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);
const table = database.table('Singers');

const row = {
  SingerId: 16,
  FirstName: 'Martha',
  LastName: 'Waller',
};

await table.insert(row, retryAndTimeoutSettings);

console.log('record inserted.');
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
from google.api_core import exceptions as core_exceptions
from google.api_core import retry

# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

retry = retry.Retry(
    # Customize retry with an initial wait time of 500 milliseconds.
    initial=0.5,
    # Customize retry with a maximum wait time of 16 seconds.
    maximum=16,
    # Customize retry with a wait time multiplier per iteration of 1.5.
    multiplier=1.5,
    # Customize retry with a timeout on
    # how long a certain RPC may be retried in
    # case the server returns an error.
    timeout=60,
    # Configure which errors should be retried.
    predicate=retry.if_exception_type(
        core_exceptions.ServiceUnavailable,
    ),
)

# Set a custom retry and timeout setting.
with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT SingerId, AlbumId, AlbumTitle FROM Albums",
        # Set custom retry setting for this request
        retry=retry,
        # Set custom timeout of 60 seconds for this request
        timeout=60,
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

spanner   = Google::Cloud::Spanner.new project: project_id
client    = spanner.client instance_id, database_id
row_count = 0

timeout = 60.0
retry_policy = {
  initial_delay: 0.5,
  max_delay:     16.0,
  multiplier:    1.5,
  retry_codes:   ["UNAVAILABLE"]
}
call_options = { timeout: timeout, retry_policy: retry_policy }

client.transaction do |transaction|
  row_count = transaction.execute_update(
    "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES (10, 'Virginia', 'Watson')",
    call_options: call_options
  )
end

puts "#{row_count} record inserted."
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
