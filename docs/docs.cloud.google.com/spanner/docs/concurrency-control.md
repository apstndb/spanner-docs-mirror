Spanner transactions offer two modes of concurrency control: *pessimistic* and *optimistic* . The choice of concurrency control mode affects how transactions handle simultaneous reads and writes, influencing performance, latency, and transaction abort rates. Choose the mode that best suits your application's performance and consistency requirements.

The default behavior depends on the [isolation level](https://docs.cloud.google.com/spanner/docs/isolation-levels) your transaction uses:

  - [Serializable isolation](https://docs.cloud.google.com/spanner/docs/isolation-levels#serializable) uses pessimistic concurrency control by default.
  - [Repeatable read isolation](https://docs.cloud.google.com/spanner/docs/isolation-levels#repeatable-read) , uses optimistic concurrency control by default.

## Pessimistic concurrency control

By default, Spanner uses pessimistic concurrency with [serializable isolation](https://docs.cloud.google.com/spanner/docs/isolation-levels#serializable) . You can also use pessimistic concurrency with [repeatable read isolation](https://docs.cloud.google.com/spanner/docs/isolation-levels#repeatable-read) .

### Pessimistic concurrency in serializable isolation

This mode assumes that concurrent transactions might contend for the same data. It acquires [locks](https://docs.cloud.google.com/spanner/docs/introspection/lock-statistics#explain-lock-modes) proactively on data as it is read or written within a transaction. It also verifies that locks acquired earlier in the transaction remain held in later statements. When Spanner detects a lock conflict, it uses the wound-wait algorithm to resolve the conflict.

In pessimistic concurrency, transactions acquire locks on data during both the execution and commit phases of the transaction.

  - **For reads:** When a transaction reads data, it acquires a [shared read ( `ReaderShared` ) lock](https://docs.cloud.google.com/spanner/docs/introspection/lock-statistics#explain-lock-modes) during the execution phase. These locks are held until the transaction commits.
  - **For DML and writes:**
      - During execution, for data modified by DML or writes, the transaction might acquire read locks on row-existence.
      - At commit time, the transaction attempts to acquire write or exclusive locks for the written data. Write locks block concurrent reads, but might not block concurrent writes, especially when they both use write locks. This means multiple transactions can proceed to commit, and write-write conflicts are resolved at commit time using the wound-wait algorithm. All locks are held until the transaction commits.

### Pessimistic concurrency in repeatable read isolation

Use pessimistic concurrency in repeatable read isolation to serialize writes. In this mode, read operations use snapshots, but [exclusive locks](https://docs.cloud.google.com/spanner/docs/introspection/lock-statistics#explain-lock-modes) apply to data read from `FOR UPDATE` queries or `lock_scanned_ranges=exclusive` hints, and data written with DML queries.

### Benefits of pessimistic concurrency with serializable isolation

The main benefit of using pessimistic concurrency with serializable isolation is that, in highly contentious workloads, it helps transactions make progress. Spanner prioritizes older transactions over newer ones during conflicts, ensuring that transactions eventually complete while reducing the amount of repeatedly aborting transactions.

### Benefits of pessimistic concurrency with repeatable read isolation

With repeatable read isolation, transactions that acquire locks might still abort at commit time if the data read as part of a query with `FOR UPDATE` or as part of a DML query, was modified by a concurrent transaction before the transaction commits. However, after locks are acquired, it prevents further concurrent updates until the transaction commits, serializing the writes.

### Risks of pessimistic concurrency

Pessimistic concurrency with serializable isolation presents the following risks:

  - Long-running reads might block latency-sensitive writes.
  - Transactions that involve user interaction before completion might cause locks to be held for a long time, potentially blocking other operations.

### Use cases for pessimistic concurrency with serializable isolation

Pessimistic concurrency is suitable for workloads with high read-write and write-write contention. It is also appropriate when transaction aborts and retries are costly. Use this default mode unless your workload has excessive long lock delays, or is significantly impacted by lock conflicts.

### Use cases for pessimistic concurrency with repeatable read isolation

Use pessimistic concurrency with repeatable read for workloads that require a `FOR UPDATE` clause or DML queries to acquire locks. This approach is particularly useful for workloads migrated to Spanner from other databases that acquire locks for these statements.

## Optimistic concurrency control

Spanner also provides optimistic concurrency control. When you use repeatable read isolation, the default mode is optimistic concurrency control. You can also configure serializable isolation to use optimistic concurrency control.

Optimistic concurrency control assumes that conflicts are rare. Reads and queries even within a read-write transaction proceed without acquiring locks. With Spanner's default serializable isolation, reads are validated at commit time. This ensures that no other concurrently committed transaction modified the data previously read by the transaction. If you use [repeatable read isolation](https://docs.cloud.google.com/spanner/docs/isolation-levels#repeatable-read) , reads with either a `FOR UPDATE` or `lock_scanned_ranges=exclusive` hint are validated at commit time. If Spanner detects a conflict, it aborts the transaction.

### How optimistic concurrency works

Optimistic concurrency changes how Spanner executes reads, queries, and commits transactions. It performs lock-free execution during the read phase and validates consistency at commit.

#### For reads and queries

Reads and queries are lock-free. All reads and queries within an optimistic transaction execute at a single, snapshot timestamp. Spanner chooses this timestamp when the first read or query executes. This ensures that all subsequent reads and queries within the transaction see writes committed before the first read or query.

#### For reads and writes

For an optimistic transaction with reads and writes, Spanner performs a validation step at commit time. The transaction commits successfully only if no conflicts are detected and the following conditions are met:

  - No concurrently committed writes conflict with the data read by this transaction; that is, no writes were committed after the read timestamp but before this transaction commits its own writes.
  - The schema wasn't modified since the read timestamp.

The isolation level determines the set of reads that are validated. With serializable isolation, all reads are validated. With repeatable read isolation, reads with either a `FOR UPDATE` or `lock_scanned_ranges=exclusive` hint are validated at commit time.

Under high contention, optimistic transactions might repeatedly abort. In contrast, pessimistic transactions resolve read-write conflicts by allowing the older transaction to commit and retrying the newer transaction.

### Benefits of optimistic concurrency

Optimistic concurrency offers the following benefits:

  - Reads don't acquire locks: Optimistic transactions don't acquire locks for reads, so long-running reads don't block latency-sensitive writes.
  - Reduced commit latency for read-only transactions: Because all reads within an optimistic transaction are based on the same snapshot timestamp, there's no need to verify consistency during execution or commit for these reads, which significantly reduces latency.

### Risks of optimistic concurrency

Optimistic concurrency introduces risks, particularly under high read-write contention when used with serializable isolation. Understand these risks before you use optimistic concurrency control with serializable isolation for your workload.

  - Under high read-write contention, optimistic transactions might experience a high rate of aborts, because concurrent writes might invalidate the reads of an optimistic transaction.
  - With persistent high contention, a transaction might be repeatedly aborted and never commit from transaction starvation.

### Use cases for optimistic concurrency

Optimistic concurrency is suitable for transactional workloads with low read-write contention. For serializable transactions, it also benefits workloads that can tolerate transaction aborts.

Consider optimistic concurrency for the following workloads:

  - **Low-priority, latency-tolerant workloads with long-running transactions:** Use optimistic concurrency if long-running reads or queries might delay latency-sensitive writes. This avoids delays caused by read locks. For example, transactions in mobile clients with slow connections, or low-SLA transactions holding read locks for many rows or large ranges.
  - **Read latency-sensitive transactional workloads with low read-write contention:** In a [multi-region configuration](https://docs.cloud.google.com/spanner/docs/instance-configurations#multi-region-configurations) , use optimistic concurrency to serve reads regionally, reduce read latencies, and avoid production issues from spiky read traffic to a hot split. It also improves read availability during leader overload or unavailability.
  - **Transactional workloads where most transactions are read-only:** Switching to optimistic concurrency reduces commit latency for common read-only transactions in these workloads. Ensure low read-write contention to avoid high abort rates for read-write transactions.

Avoid using optimistic concurrency for latency-sensitive transactional workloads where read-write conflicts are frequent.

## Configure concurrency control

You can use the Spanner client libraries, REST, and RPC API to specify the concurrency mode for read-write transactions.

### Client libraries

### Java

    static void readLockModeSetting(DatabaseId db) {
      // The read lock mode specified at the client-level will be applied to all
      // RW transactions.
      DefaultReadWriteTransactionOptions transactionOptions =
          DefaultReadWriteTransactionOptions.newBuilder()
              .setReadLockMode(ReadLockMode.OPTIMISTIC)
              .build();
      SpannerOptions options =
          SpannerOptions.newBuilder()
              .setDefaultTransactionOptions(transactionOptions)
              .build();
      Spanner spanner = options.getService();
      DatabaseClient dbClient = spanner.getDatabaseClient(db);
      dbClient
          // The read lock mode specified at the transaction-level takes precedence
          // over the read lock mode configured at the client-level.
          .readWriteTransaction(Options.readLockMode(ReadLockMode.PESSIMISTIC))
          .run(transaction -> {
            // Read an AlbumTitle.
            String selectSql =
                "SELECT AlbumTitle from Albums WHERE SingerId = 1 and AlbumId = 1";
            String title = null;
            try (ResultSet resultSet = transaction.executeQuery(Statement.of(selectSql))) {
              if (resultSet.next()) {
                title = resultSet.getString("AlbumTitle");
              }
            }
            System.out.printf("Current album title: %s\n", title);
    
            // Update the title.
            String updateSql =
                "UPDATE Albums "
                    + "SET AlbumTitle = 'New Album Title' "
                    + "WHERE SingerId = 1 and AlbumId = 1";
            long rowCount = transaction.executeUpdate(Statement.of(updateSql));
            System.out.printf("%d record updated.\n", rowCount);
            return null;
          });
    }

### Go

    import (
     "context"
     "fmt"
     "io"
    
     "cloud.google.com/go/spanner"
     pb "cloud.google.com/go/spanner/apiv1/spannerpb"
    )
    
    // writeWithTransactionUsingReadLockMode sets the ReadLockMode globally
    // by using ClientConfig and shows how to override it for a specific
    // transaction. ReadLockMode determines the locking strategy used during
    // transaction execution.
    func writeWithTransactionUsingReadLockMode(w io.Writer, db string) error {
     ctx := context.Background()
    
     // Client-level configuration: Applies to all read-write transactions
     // for this client. OPTIMISTIC mode avoids locks during reads and
     // verifies changes during the commit phase.
     cfg := spanner.ClientConfig{
         TransactionOptions: spanner.TransactionOptions{
             ReadLockMode: pb.TransactionOptions_ReadWrite_OPTIMISTIC,
         },
     }
     client, err := spanner.NewClientWithConfig(ctx, db, cfg)
     if err != nil {
         return fmt.Errorf("failed to create client: %w", err)
     }
     defer client.Close()
    
     // Transaction-level options take precedence over client-level
     // configuration. PESSIMISTIC mode is used here to override the
     // client-level setting and ensure immediate locking during reads.
     txnOpts := spanner.TransactionOptions{
         ReadLockMode: pb.TransactionOptions_ReadWrite_PESSIMISTIC,
     }
    
     _, err = client.ReadWriteTransactionWithOptions(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
         // In PESSIMISTIC mode with SERIALIZABLE isolation, the transaction
         // acquires a shared lock during this read.
         key := spanner.Key{1, 2}
         row, err := txn.ReadRow(ctx, "Albums", key, []string{"AlbumTitle"})
         if err != nil {
             return fmt.Errorf("failed to read album: %w", err)
         }
         var title string
         if err := row.Column(0, &title); err != nil {
             return fmt.Errorf("failed to get album title: %w", err)
         }
         fmt.Fprintf(w, "Current album title: %s\n", title)
    
         // Update the album title
         stmt := spanner.Statement{
             SQL: `UPDATE Albums
                 SET AlbumTitle = @AlbumTitle
                 WHERE SingerId = @SingerId AND AlbumId = @AlbumId`,
             Params: map[string]interface{}{
                 "SingerId":   1,
                 "AlbumId":    2,
                 "AlbumTitle": "New Album Title",
             },
         }
         count, err := txn.Update(ctx, stmt)
         if err != nil {
             return fmt.Errorf("failed to update album: %w", err)
         }
         fmt.Fprintf(w, "Updated %d record(s).\n", count)
         return nil
     }, txnOpts)
    
     if err != nil {
         return fmt.Errorf("transaction failed: %w", err)
     }
     return nil
    }

### Node.js

    // Imports the Google Cloud Spanner client library
    const {Spanner, protos} = require('@google-cloud/spanner');
    // The read lock mode specified at the client-level will be applied
    // to all RW transactions.
    const defaultTransactionOptions = {
      readLockMode:
        protos.google.spanner.v1.TransactionOptions.ReadWrite.ReadLockMode
          .OPTIMISTIC,
    };
    
    // Instantiates a client with defaultTransactionOptions
    const spanner = new Spanner({
      projectId: projectId,
      defaultTransactionOptions,
    });
    
    function runTransactionWithReadLockMode() {
      // Gets a reference to a Cloud Spanner instance and database
      const instance = spanner.instance(instanceId);
      const database = instance.database(databaseId);
      // The read lock mode specified at the request-level takes precedence over
      // the read lock mode configured at the client-level.
      const readLockModeOptionsForTransaction = {
        readLockMode:
          protos.google.spanner.v1.TransactionOptions.ReadWrite.ReadLockMode
            .PESSIMISTIC,
      };
    
      database.runTransaction(
        readLockModeOptionsForTransaction,
        async (err, transaction) => {
          if (err) {
            console.error(err);
            return;
          }
          try {
            const query =
              'SELECT AlbumTitle FROM Albums WHERE SingerId = 2 AND AlbumId = 1';
            const results = await transaction.run(query);
            // Gets first album's title
            const rows = results[0].map(row => row.toJSON());
            const albumTitle = rows[0].AlbumTitle;
            console.log(`previous album title ${albumTitle}`);
    
            const update =
              "UPDATE Albums SET AlbumTitle = 'New Album Title' WHERE SingerId = 2 AND AlbumId = 1";
            const [rowCount] = await transaction.runUpdate(update);
            console.log(
              `Successfully updated ${rowCount} record in Albums table.`,
            );
            await transaction.commit();
            console.log(
              'Successfully executed read-write transaction with readLockMode option.',
            );
          } catch (err) {
            console.error('ERROR:', err);
            transaction.end();
          } finally {
            // Close the database when finished.
            await database.close();
          }
        },
      );
    }
    runTransactionWithReadLockMode();

### Python

    # instance_id = "your-spanner-instance"
    # database_id = "your-spanner-db-id"
    from google.cloud.spanner_v1 import TransactionOptions, DefaultTransactionOptions
    
    # The read lock mode specified at the client-level will be applied to all
    # RW transactions.
    read_lock_mode_options_for_client = TransactionOptions.ReadWrite.ReadLockMode.OPTIMISTIC
    
    # Create a client that uses Serializable isolation (default) with
    # optimistic locking for read-write transactions.
    spanner_client = spanner.Client(
        default_transaction_options=DefaultTransactionOptions(
            read_lock_mode=read_lock_mode_options_for_client
        )
    )
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)
    
    # The read lock mode specified at the request level takes precedence over
    # the read lock mode configured at the client level.
    read_lock_mode_options_for_transaction = (
        TransactionOptions.ReadWrite.ReadLockMode.PESSIMISTIC
    )
    
    def update_albums_with_read_lock_mode(transaction):
        # Read an AlbumTitle.
        results = transaction.execute_sql(
            "SELECT AlbumTitle from Albums WHERE SingerId = 2 and AlbumId = 1"
        )
        for result in results:
            print("Current Album Title: {}".format(*result))
    
        # Update the AlbumTitle.
        row_ct = transaction.execute_update(
            "UPDATE Albums SET AlbumTitle = 'A New Title' WHERE SingerId = 2 and AlbumId = 1"
        )
    
        print("{} record(s) updated.".format(row_ct))
    
    database.run_in_transaction(
        update_albums_with_read_lock_mode,
        read_lock_mode=read_lock_mode_options_for_transaction
    )

### C\#

    using Google.Cloud.Spanner.Data;
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    
    public class ReadLockModeAsyncSample
    {
        public async Task ReadLockModeAsync(string projectId, string instanceId, string databaseId)
        {
            // Create client with ReadLockMode.Optimistic.
            string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId};ReadLockMode=Optimistic";
    
            using var connection = new SpannerConnection(connectionString);
            await connection.OpenAsync();
    
            // Create transaction options with ReadLockMode.Pessimistic.
            var transactionOptions = SpannerTransactionCreationOptions.ReadWrite
                .WithReadLockMode(ReadLockMode.Pessimistic);
    
            using var transaction = await connection.BeginTransactionAsync(transactionOptions, null, CancellationToken.None);
    
            var cmd = connection.CreateSelectCommand("SELECT AlbumTitle FROM Albums WHERE SingerId = 2 AND AlbumId = 1");
            cmd.Transaction = transaction;
            using (var reader = await cmd.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    Console.WriteLine($"AlbumTitle: {reader.GetFieldValue<string>("AlbumTitle")}");
                }
            }
    
            var updateCmd = connection.CreateDmlCommand("UPDATE Albums SET AlbumTitle = 'A New Title' WHERE SingerId = 2 AND AlbumId = 1");
            updateCmd.Transaction = transaction;
            var rowCount = await updateCmd.ExecuteNonQueryAsync();
            Console.WriteLine($"{rowCount} records updated.");
    
            await transaction.CommitAsync();
        }
    }

### C++

    void ReadLockModeSetting(std::string const& project_id,
                             std::string const& instance_id,
                             std::string const& database_id) {
      namespace spanner = ::google::cloud::spanner;
      using ::google::cloud::Options;
      using ::google::cloud::StatusOr;
    
      auto db = spanner::Database(project_id, instance_id, database_id);
    
      // The read lock mode specified at the client-level will be applied
      // to all RW transactions.
      auto options = Options{}.set<spanner::TransactionReadLockModeOption>(
          spanner::Transaction::ReadLockMode::kOptimistic);
      auto client = spanner::Client(spanner::MakeConnection(db, options));
    
      auto commit = client.Commit(
          [&client](
              spanner::Transaction const& txn) -> StatusOr<spanner::Mutations> {
            // Read an AlbumTitle.
            auto sql = spanner::SqlStatement(
                "SELECT AlbumTitle from Albums WHERE SingerId = @SingerId and "
                "AlbumId = @AlbumId",
                {{"SingerId", spanner::Value(2)}, {"AlbumId", spanner::Value(1)}});
            auto rows = client.ExecuteQuery(txn, std::move(sql));
            for (auto const& row :
                 spanner::StreamOf<std::tuple<std::string>>(rows)) {
              if (!row) return row.status();
              std::cout << "Current Album Title: " << std::get<0>(*row) << "\n";
            }
    
            // Update the title.
            auto update_sql = spanner::SqlStatement(
                "UPDATE Albums "
                "SET AlbumTitle = @AlbumTitle "
                "WHERE SingerId = @SingerId and AlbumId = @AlbumId",
                {{"AlbumTitle", spanner::Value("A New Title")},
                 {"SingerId", spanner::Value(2)},
                 {"AlbumId", spanner::Value(1)}});
            auto result = client.ExecuteDml(txn, std::move(update_sql));
            if (!result) return result.status();
            std::cout << result->RowsModified() << " record(s) updated.\n";
    
            return spanner::Mutations{};
          },
          // The read lock mode specified at the transaction-level takes
          // precedence over the read lock mode configured at the client-level.
          // kPessimistic is used here to demonstrate overriding the client-level
          // setting.
          Options{}.set<spanner::TransactionReadLockModeOption>(
              spanner::Transaction::ReadLockMode::kPessimistic));
    
      if (!commit) throw std::move(commit).status();
      std::cout << "Update was successful [spanner_read_lock_mode]\n";
    }

### REST

The Spanner [TransactionOptions](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions) REST API provides a `ReadLockMode` enum within the `ReadWrite` message that lets you select either the `PESSIMISTIC` or `OPTIMISTIC` lock mode.

### RPC

The Spanner [Transactionoptions](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#transactionoptions) RPC API provides a `ReadLockMode` enum within the `ReadWrite` message that lets you select either the `PESSIMISTIC` or `OPTIMISTIC` lock mode.

### Drivers

You can use Spanner's drivers to set `read_lock_mode` as a connection parameter at the connection level or as a `SET` statement option at the transaction level. For more information about each driver, see [Overview of drivers](https://docs.cloud.google.com/spanner/docs/drivers-overview) .

## What's next

  - Learn more about [Spanner isolation levels](https://docs.cloud.google.com/spanner/docs/isolation-levels) .
  - Learn how to [use repeatable read isolation](https://docs.cloud.google.com/spanner/docs/use-repeatable-read-isolation) .
