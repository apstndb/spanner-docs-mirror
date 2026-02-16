**Note:** The examples in this document are intended for GoogleSQL-dialect databases. However, the concepts and semantics apply to both GoogleSQL-dialect and PostgreSQL-dialect databases.

This page describes transactions in Spanner and introduces Spanner's read-write, read-only, and partitioned DML transaction interfaces.

A transaction in Spanner is a set of reads and writes. All operations in a transaction are atomic, meaning either they all succeed or they all fail.

A session is used to perform transactions in a Spanner database. A session represents a logical communication channel with the Spanner database service. Sessions can execute a single or multiple transactions at a time. For more information, see [Sessions](/spanner/docs/sessions) .

## Transaction types

Spanner supports the following transaction types, each designed for specific data interaction patterns:

  - **Read-write:** These transactions are used for read and write operations, followed by a commit. They might acquire locks. If they fail, they'll require retries. While they're confined to a single database, they can modify data across multiple tables within that database.

  - **Read-only:** These transactions guarantee data consistency across multiple read operations, but don't permit data modifications. They execute at a system-determined timestamp for consistency, or at a user-configured past timestamp. Unlike read-write transactions, they don't require a commit operation or locks. However, they might pause to wait for ongoing write operations to conclude.

  - **Partitioned DML:** This transaction type executes DML statements as [partitioned DML](/spanner/docs/dml-partitioned) operations. It's optimized for executing DML statements at scale but with restrictions to ensure the statement is idempotent and partitionable in a way that lets it execute independently of other partitions. For numerous writes that don't need an atomic transaction, consider using batch writes. For more information, see [Modify data using batch writes](/spanner/docs/batch-write) .

**Note:** Because read-only transactions can execute on any replica, use them for read operations unless a read-write transaction is explicitly required. Only use locking read-write transactions when required.

## Read-write transactions

A read-write transaction consists of zero or more reads or query statements followed by a commit request. At any time before the commit request, the client can send a rollback request to abort the transaction.

### Serializable isolation

Using the default serializable isolation level, read-write transactions atomically read, modify, and write data. This type of transaction is [externally consistent](/spanner/docs/true-time-external-consistency) .

When you use read-write transactions, we recommend that you minimize the time that a transaction is active. Shorter transaction durations result in locks being held for less time, which increases the probability of a successful commit and reduces contention. This is because long-held locks can lead to deadlocks and transaction aborts. Spanner attempts to keep read locks active as long as the transaction continues to perform reads and the transaction has not terminated through commit or roll back. If the client remains inactive for long periods of time, Spanner might release the transaction's locks and abort the transaction.

To perform a write operation that depends on one or more read operations, use a read-write transaction:

  - If you must commit one or more write operations atomically, perform those writes within the same read-write transaction. For example, if you transfer $200 from account A to account B, perform both write operations (decreasing account A by $200 and increasing account B by $200) and the reads of the initial account balances within the same transaction.
  - If you want to double the balance of account A, perform the read and write operations within the same transaction. This ensures the system reads the balance before doubling and updating it.
  - If write operations depend on read operations, perform both within the same read-write transaction, even if the writes don't execute. For example, if you want to transfer $200 from account A to account B only if A's balance is greater than $500, include the read of A's balance and the conditional write operations within the same transaction, even if the transfer doesn't occur.

To perform read operations, use a single read method or read-only transaction:

  - If you're only performing read operations, and you can express the read operation using a [single read method](/spanner/docs/reads#single_read_methods) , use the single read method or a read-only transaction. Unlike read-write transactions, single reads don't acquire locks.

### Repeatable read isolation

**Preview — Repeatable read isolation**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

In Spanner, [repeatable read isolation](/spanner/docs/isolation-levels#repeatable-read) is implemented using a technique known as snapshot isolation. Repeatable read isolation ensures that all read operations within a transaction are consistent with the database as it existed at the start of the transaction. It also guarantees that concurrent writes on the same data only succeed if there are no conflicts.

With its default optimistic locking, no locks are acquired until commit time if data needs to be written. If there is a conflict with the written data or due to transient events within Spanner like a server restart, Spanner might still abort transactions. Because reads in read-write transactions don't acquire locks in repeatable read isolation, there is no difference between executing read-only operations within a read-only transaction or read-write transaction.

Consider using read-write transactions in repeatable read isolation in the following scenarios:

  - The workload is read-heavy and has low write conflicts.
  - The application is experiencing performance bottlenecks due to delays from lock-contention and transaction aborts caused by older, higher-priority transactions wounding newer, lower-priority transactions to prevent potential deadlocks (wound-wait).
  - The application doesn't require the stricter guarantees provided by the serializable isolation level.

When performing a write operation that depends on one or more read operations, write skew is possible under repeatable read isolation. Write skew arise from a particular kind of concurrent update, where each update is independently accepted, but their combined effect violates application data integrity. Therefore, make sure you perform reads that are part of a transaction's critical section with either a `  FOR UPDATE  ` clause or a `  lock_scanned_ranges=exclusive  ` hint to avoid write skew. For more information, see [Read-write conflicts and correctness](/spanner/docs/isolation-levels#read-write-conflicts-correctness) , and the example discussed in [Read-write semantics](/spanner/docs/transactions#rw_transaction_semantics) .

### Interface

The [Spanner client libraries](/spanner/docs/reference/libraries) provide an interface for executing a body of work within a read-write transaction, with retries for transaction aborts. A transaction might require multiple retries before it commits.

Several situations can cause transaction aborts. For example, if two transactions attempt to modify data concurrently, a deadlock might occur. In such cases, Spanner aborts one transaction to let the other proceed. Less frequently, transient events within Spanner can also cause transaction aborts.

All read-write transactions provide the ACID properties of relational databases. Because transactions are atomic, an aborted transaction doesn't affect the database. Spanner client libraries retry such transactions automatically, but if you don't use the client libraries, retry the transaction within the same session to improve success rates. Each retry that results in an `  ABORTED  ` error increases the transaction's lock priority. In addition, Spanner client drivers include an internal transaction retry logic that masks transient errors by rerunning the transaction.

When using a transaction in a Spanner client library, you define the transaction's body as a function object. This function encapsulates the reads and writes performed on one or more database tables. The Spanner client library executes this function repeatedly until the transaction either commits successfully or encounters an error that can't be retried.

### Example

Assume you have a `  MarketingBudget  ` column in the [`  Albums  ` table](/spanner/docs/schema-and-data-model#creating_multiple_tables) :

``` text
CREATE TABLE Albums (
  SingerId        INT64 NOT NULL,
  AlbumId         INT64 NOT NULL,
  AlbumTitle      STRING(MAX),
  MarketingBudget INT64
) PRIMARY KEY (SingerId, AlbumId);
```

Your marketing department asks you to move $200,000 from the budget of `  Albums (2, 2)  ` to `  Albums (1, 1)  ` , but only if the money is available in that album's budget. You should use a locking read-write transaction for this operation, because the transaction might perform writes depending on the result of a read.

The following client library examples show how to execute a read-write transaction using the default serializable isolation level:

### C++

``` cpp
void ReadWriteTransaction(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  using ::google::cloud::Status;
  using ::google::cloud::StatusCode;
  using ::google::cloud::StatusOr;

  // A helper to read a single album MarketingBudget.
  auto get_current_budget =
      [](spanner::Client client, spanner::Transaction txn,
         std::int64_t singer_id,
         std::int64_t album_id) -> StatusOr<std::int64_t> {
    auto key = spanner::KeySet().AddKey(spanner::MakeKey(singer_id, album_id));
    auto rows = client.Read(std::move(txn), "Albums", std::move(key),
                            {"MarketingBudget"});
    using RowType = std::tuple<std::int64_t>;
    auto row = spanner::GetSingularRow(spanner::StreamOf<RowType>(rows));
    if (!row) return std::move(row).status();
    return std::get<0>(*std::move(row));
  };

  auto constexpr kInsufficientFundsMessage =
      "The second album does not have enough funds to transfer";
  auto commit = client.Commit(
      [&](spanner::Transaction const& txn) -> StatusOr<spanner::Mutations> {
        auto b1 = get_current_budget(client, txn, 1, 1);
        if (!b1) return std::move(b1).status();
        auto b2 = get_current_budget(client, txn, 2, 2);
        if (!b2) return std::move(b2).status();
        std::int64_t transfer_amount = 200000;

        if (*b2 < transfer_amount) {
          return Status(StatusCode::kFailedPrecondition,
                        kInsufficientFundsMessage);
        }

        return spanner::Mutations{
            spanner::UpdateMutationBuilder(
                "Albums", {"SingerId", "AlbumId", "MarketingBudget"})
                .EmplaceRow(1, 1, *b1 + transfer_amount)
                .EmplaceRow(2, 2, *b2 - transfer_amount)
                .Build()};
      });

  if (!commit) throw std::move(commit).status();
  std::cout << "Transfer was successful [spanner_read_write_transaction]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;
using System.Transactions;

public class ReadWriteWithTransactionAsyncSample
{
    public async Task<int> ReadWriteWithTransactionAsync(string projectId, string instanceId, string databaseId)
    {
        // This sample transfers 200,000 from the MarketingBudget
        // field of the second Album to the first Album. Make sure to run
        // the Add Column and Write Data To New Column samples first,
        // in that order.

        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using TransactionScope scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
        decimal transferAmount = 200000;
        decimal secondBudget = 0;
        decimal firstBudget = 0;

        using var connection = new SpannerConnection(connectionString);
        using var cmdLookup1 = connection.CreateSelectCommand("SELECT * FROM Albums WHERE SingerId = 2 AND AlbumId = 2");

        using (var reader = await cmdLookup1.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                // Read the second album's budget.
                secondBudget = reader.GetFieldValue<decimal>("MarketingBudget");
                // Confirm second Album's budget is sufficient and
                // if not raise an exception. Raising an exception
                // will automatically roll back the transaction.
                if (secondBudget < transferAmount)
                {
                    throw new Exception($"The second album's budget {secondBudget} is less than the amount to transfer.");
                }
            }
        }

        // Read the first album's budget.
        using var cmdLookup2 = connection.CreateSelectCommand("SELECT * FROM Albums WHERE SingerId = 1 and AlbumId = 1");
        using (var reader = await cmdLookup2.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                firstBudget = reader.GetFieldValue<decimal>("MarketingBudget");
            }
        }

        // Specify update command parameters.
        using var cmdUpdate = connection.CreateUpdateCommand("Albums", new SpannerParameterCollection
        {
            { "SingerId", SpannerDbType.Int64 },
            { "AlbumId", SpannerDbType.Int64 },
            { "MarketingBudget", SpannerDbType.Int64 },
        });

        // Update second album to remove the transfer amount.
        secondBudget -= transferAmount;
        cmdUpdate.Parameters["SingerId"].Value = 2;
        cmdUpdate.Parameters["AlbumId"].Value = 2;
        cmdUpdate.Parameters["MarketingBudget"].Value = secondBudget;
        var rowCount = await cmdUpdate.ExecuteNonQueryAsync();

        // Update first album to add the transfer amount.
        firstBudget += transferAmount;
        cmdUpdate.Parameters["SingerId"].Value = 1;
        cmdUpdate.Parameters["AlbumId"].Value = 1;
        cmdUpdate.Parameters["MarketingBudget"].Value = firstBudget;
        rowCount += await cmdUpdate.ExecuteNonQueryAsync();
        scope.Complete();
        Console.WriteLine("Transaction complete.");
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

func writeWithTransaction(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     getBudget := func(key spanner.Key) (int64, error) {
         row, err := txn.ReadRow(ctx, "Albums", key, []string{"MarketingBudget"})
         if err != nil {
             return 0, err
         }
         var budget int64
         if err := row.Column(0, &budget); err != nil {
             return 0, err
         }
         return budget, nil
     }
     album2Budget, err := getBudget(spanner.Key{2, 2})
     if err != nil {
         return err
     }
     const transferAmt = 200000
     if album2Budget >= transferAmt {
         album1Budget, err := getBudget(spanner.Key{1, 1})
         if err != nil {
             return err
         }
         album1Budget += transferAmt
         album2Budget -= transferAmt
         cols := []string{"SingerId", "AlbumId", "MarketingBudget"}
         txn.BufferWrite([]*spanner.Mutation{
             spanner.Update("Albums", cols, []interface{}{1, 1, album1Budget}),
             spanner.Update("Albums", cols, []interface{}{2, 2, album2Budget}),
         })
         fmt.Fprintf(w, "Moved %d from Album2's MarketingBudget to Album1's.", transferAmt)
     }
     return nil
 })
 return err
}
```

### Java

``` java
static void writeWithTransaction(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(transaction -> {
        // Transfer marketing budget from one album to another. We do it in a transaction to
        // ensure that the transfer is atomic.
        Struct row =
            transaction.readRow("Albums", Key.of(2, 2), Arrays.asList("MarketingBudget"));
        long album2Budget = row.getLong(0);
        // Transaction will only be committed if this condition still holds at the time of
        // commit. Otherwise it will be aborted and the callable will be rerun by the
        // client library.
        long transfer = 200000;
        if (album2Budget >= transfer) {
          long album1Budget =
              transaction
                  .readRow("Albums", Key.of(1, 1), Arrays.asList("MarketingBudget"))
                  .getLong(0);
          album1Budget += transfer;
          album2Budget -= transfer;
          transaction.buffer(
              Mutation.newUpdateBuilder("Albums")
                  .set("SingerId")
                  .to(1)
                  .set("AlbumId")
                  .to(1)
                  .set("MarketingBudget")
                  .to(album1Budget)
                  .build());
          transaction.buffer(
              Mutation.newUpdateBuilder("Albums")
                  .set("SingerId")
                  .to(2)
                  .set("AlbumId")
                  .to(2)
                  .set("MarketingBudget")
                  .to(album2Budget)
                  .build());
        }
        return null;
      });
}
```

### Node.js

``` javascript
// This sample transfers 200,000 from the MarketingBudget field
// of the second Album to the first Album, as long as the second
// Album has enough money in its budget. Make sure to run the
// addColumn and updateData samples first (in that order).

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

const transferAmount = 200000;

// Note: the `runTransaction()` method is non blocking and returns "void".
// For sequential execution of the transaction use `runTransactionAsync()` method which returns a promise.
// For example: await database.runTransactionAsync(async (err, transaction) => { ... })
database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  let firstBudget, secondBudget;
  const queryOne = {
    columns: ['MarketingBudget'],
    keys: [[2, 2]], // SingerId: 2, AlbumId: 2
  };

  const queryTwo = {
    columns: ['MarketingBudget'],
    keys: [[1, 1]], // SingerId: 1, AlbumId: 1
  };

  Promise.all([
    // Reads the second album's budget
    transaction.read('Albums', queryOne).then(results => {
      // Gets second album's budget
      const rows = results[0].map(row => row.toJSON());
      secondBudget = rows[0].MarketingBudget;
      console.log(`The second album's marketing budget: ${secondBudget}`);

      // Makes sure the second album's budget is large enough
      if (secondBudget < transferAmount) {
        throw new Error(
          `The second album's budget (${secondBudget}) is less than the transfer amount (${transferAmount}).`,
        );
      }
    }),

    // Reads the first album's budget
    transaction.read('Albums', queryTwo).then(results => {
      // Gets first album's budget
      const rows = results[0].map(row => row.toJSON());
      firstBudget = rows[0].MarketingBudget;
      console.log(`The first album's marketing budget: ${firstBudget}`);
    }),
  ])
    .then(() => {
      console.log(firstBudget, secondBudget);
      // Transfers the budgets between the albums
      firstBudget += transferAmount;
      secondBudget -= transferAmount;

      console.log(firstBudget, secondBudget);

      // Updates the database
      // Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
      // must be converted (back) to strings before being inserted as INT64s.
      transaction.update('Albums', [
        {
          SingerId: '1',
          AlbumId: '1',
          MarketingBudget: firstBudget.toString(),
        },
        {
          SingerId: '2',
          AlbumId: '2',
          MarketingBudget: secondBudget.toString(),
        },
      ]);
    })
    .then(() => {
      // Commits the transaction and send the changes to the database
      return transaction.commit();
    })
    .then(() => {
      console.log(
        `Successfully executed read-write transaction to transfer ${transferAmount} from Album 2 to Album 1.`,
      );
    })
    .catch(err => {
      console.error('ERROR:', err);
    })
    .then(() => {
      transaction.end();
      // Closes the database when finished
      return database.close();
    });
});
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;
use UnexpectedValueException;

/**
 * Performs a read-write transaction to update two sample records in the
 * database.
 *
 * This will transfer 200,000 from the `MarketingBudget` field for the second
 * Album to the first Album. If the `MarketingBudget` for the second Album is
 * too low, it will raise an exception.
 *
 * Before running this sample, you will need to run the `update_data` sample
 * to populate the fields.
 * Example:
 * ```
 * read_write_transaction($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_write_transaction(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) use ($spanner) {
        $transferAmount = 200000;

        // Read the second album's budget.
        $secondAlbumKey = [2, 2];
        $secondAlbumKeySet = $spanner->keySet(['keys' => [$secondAlbumKey]]);
        $secondAlbumResult = $t->read(
            'Albums',
            $secondAlbumKeySet,
            ['MarketingBudget'],
            ['limit' => 1]
        );

        $firstRow = $secondAlbumResult->rows()->current();
        $secondAlbumBudget = $firstRow['MarketingBudget'];
        if ($secondAlbumBudget < $transferAmount) {
            // Throwing an exception will automatically roll back the transaction.
            throw new UnexpectedValueException(
                'The second album\'s budget is lower than the transfer amount: ' . $transferAmount
            );
        }

        $firstAlbumKey = [1, 1];
        $firstAlbumKeySet = $spanner->keySet(['keys' => [$firstAlbumKey]]);
        $firstAlbumResult = $t->read(
            'Albums',
            $firstAlbumKeySet,
            ['MarketingBudget'],
            ['limit' => 1]
        );

        // Read the first album's budget.
        $firstRow = $firstAlbumResult->rows()->current();
        $firstAlbumBudget = $firstRow['MarketingBudget'];

        // Update the budgets.
        $secondAlbumBudget -= $transferAmount;
        $firstAlbumBudget += $transferAmount;
        printf('Setting first album\'s budget to %s and the second album\'s ' .
            'budget to %s.' . PHP_EOL, $firstAlbumBudget, $secondAlbumBudget);

        // Update the rows.
        $t->updateBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'MarketingBudget' => $firstAlbumBudget],
            ['SingerId' => 2, 'AlbumId' => 2, 'MarketingBudget' => $secondAlbumBudget],
        ]);

        // Commit the transaction!
        $t->commit();

        print('Transaction complete.' . PHP_EOL);
    });
}
````

### Python

``` python
def read_write_transaction(instance_id, database_id):
    """Performs a read-write transaction to update two sample records in the
    database.

    This will transfer 200,000 from the `MarketingBudget` field for the second
    Album to the first Album. If the `MarketingBudget` is too low, it will
    raise an exception.

    Before running this sample, you will need to run the `update_data` sample
    to populate the fields.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    def update_albums(transaction):
        # Read the second album budget.
        second_album_keyset = spanner.KeySet(keys=[(2, 2)])
        second_album_result = transaction.read(
            table="Albums",
            columns=("MarketingBudget",),
            keyset=second_album_keyset,
            limit=1,
        )
        second_album_row = list(second_album_result)[0]
        second_album_budget = second_album_row[0]

        transfer_amount = 200000

        if second_album_budget < transfer_amount:
            # Raising an exception will automatically roll back the
            # transaction.
            raise ValueError("The second album doesn't have enough funds to transfer")

        # Read the first album's budget.
        first_album_keyset = spanner.KeySet(keys=[(1, 1)])
        first_album_result = transaction.read(
            table="Albums",
            columns=("MarketingBudget",),
            keyset=first_album_keyset,
            limit=1,
        )
        first_album_row = list(first_album_result)[0]
        first_album_budget = first_album_row[0]

        # Update the budgets.
        second_album_budget -= transfer_amount
        first_album_budget += transfer_amount
        print(
            "Setting first album's budget to {} and the second album's "
            "budget to {}.".format(first_album_budget, second_album_budget)
        )

        # Update the rows.
        transaction.update(
            table="Albums",
            columns=("SingerId", "AlbumId", "MarketingBudget"),
            values=[(1, 1, first_album_budget), (2, 2, second_album_budget)],
        )

    database.run_in_transaction(update_albums)

    print("Transaction complete.")
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner         = Google::Cloud::Spanner.new project: project_id
client          = spanner.client instance_id, database_id
transfer_amount = 200_000

client.transaction do |transaction|
  first_album  = transaction.read("Albums", [:MarketingBudget], keys: [[1, 1]]).rows.first
  second_album = transaction.read("Albums", [:MarketingBudget], keys: [[2, 2]]).rows.first

  raise "The second album does not have enough funds to transfer" if second_album[:MarketingBudget] < transfer_amount

  new_first_album_budget  = first_album[:MarketingBudget] + transfer_amount
  new_second_album_budget = second_album[:MarketingBudget] - transfer_amount

  transaction.update "Albums", [
    { SingerId: 1, AlbumId: 1, MarketingBudget: new_first_album_budget  },
    { SingerId: 2, AlbumId: 2, MarketingBudget: new_second_album_budget }
  ]
end

puts "Transaction complete"
```

For examples on how to execute a read-write transaction using repeatable read isolation, see [Use repeatable read isolation level](/spanner/docs/use-repeatable-read-isolation) .

### Semantics

This section describes the semantics for read-write transactions in Spanner.

#### Properties

Serializable isolation is the default isolation level in Spanner. Under serializable isolation, Spanner provides clients with the strictest concurrency-control guarantees for transactions, [external consistency](/spanner/docs/true-time-external-consistency) . A read-write transaction executes a set of reads and writes atomically. Writes can proceed without being blocked by read-only transactions. The timestamp at which read-write transactions execute matches elapsed time. The serialization order matches this timestamp order.

Because of these properties, as an application developer, you can focus on the correctness of each transaction on its own, without worrying about how to protect its execution from other transactions that might execute at the same time.

You can also execute your read-write transactions using repeatable read isolation. Repeatable read isolation ensures that all read operations within a transaction see a consistent strong snapshot of the database as it existed at the start of the transaction. For more information, see [Repeatable read isolation](/spanner/docs/isolation-levels#repeatable-read) .

#### Read-write transactions with serializable isolation

After successfully committing a transaction that contains a series of reads and writes in the default serializable isolation, the following applies:

  - The transaction returns values that reflect a consistent snapshot at the transaction's commit timestamp.
  - Empty rows or ranges remain empty at commit time.
  - The transaction commits all writes at the transaction's commit timestamp.
  - No transaction can see the writes until after the transaction commits.

Spanner client drivers include transaction retry logic that masks transient errors by rerunning the transaction and validating the data the client observes.

The effect is that all reads and writes appear to have occurred at a single point in time, both from the perspective of the transaction itself and from the perspective of other readers and writers to the Spanner database. This means the reads and writes occur at the same timestamp. For an example, see [Serializability and external consistency](#serializability_and_external_consistency) .

#### Read-write transactions with repeatable read isolation

After successfully committing a transaction with repeatable read isolation, the following applies:

  - The transaction returns values that reflect a consistent snapshot of the database. The snapshot is typically established during the first transaction operation, which might not be the same as the commit timestamp.
  - Since repeatable read is implemented using snapshot isolation, the transaction commits all writes at the transaction's commit timestamp only if the write-set hasn't changed between the transaction snapshot timestamp and the commit timestamp.
  - Other transactions don't see the writes until after the transaction commits.

#### Isolation for read-write transactions with read-only operations

When a read-write transaction performs only read operations, it provides similar consistency guarantees as a read-only transaction. All reads within the transaction return data from a consistent timestamp, including confirmation of non-existent rows.

One difference is when a read-write transaction commits without executing a write operation. In this scenario, there's no guarantee that the data read within the transaction remained unchanged in the database between the read operation and the transaction's commit.

To ensure data freshness and validate that data hasn't been modified since its last retrieval, a subsequent read is required. This re-read can be performed either within another read-write transaction or with a strong read.

For optimal efficiency, if a transaction is exclusively performing reads, use a [read-only transaction](#read-only_transactions) instead of a read-write transaction, especially when using serializable isolation.

#### How serializability and external consistency differs from repeatable read

By default, Spanner offers strong transactional guarantees, including *serializability* and *external consistency* . These properties ensure that data remains consistent and operations occur in a predictable order, even in a distributed environment.

Serializability ensures that all transactions appear to execute one after another in a single, sequential order, even if they are processed concurrently. Spanner achieves this by assigning commit timestamps to transactions, reflecting the order in which they were committed.

Spanner provides an even stronger guarantee known as [external consistency](/spanner/docs/true-time-external-consistency) . This means that not only do transactions commit in an order reflected by their commit timestamps, but these timestamps also align with real-world time. This lets you compare commit timestamps to real time, providing a consistent and globally ordered view of your data.

In essence, if a transaction `  Txn1  ` commits before another transaction `  Txn2  ` in real time, then `  Txn1  ` 's commit timestamp is earlier than `  Txn2  ` 's commit timestamp.

Consider the following example:

In this scenario, during the timeline `  t  ` :

  - Transaction `  Txn1  ` reads data `  A  ` , stages a write to `  A  ` , and then successfully commits.
  - Transaction `  Txn2  ` begins after `  Txn1  ` starts. It reads data `  B  ` and then reads data `  A  ` .

Even though `  Txn2  ` started before Txn1 completed, `  Txn2  ` observes the changes made by `  Txn1  ` to `  A  ` . This is because `  Txn2  ` reads `  A  ` after `  Txn1  ` commits its write to `  A  ` .

While `  Txn1  ` and `  Txn2  ` might overlap in their execution time, their commit timestamps, `  c1  ` and `  c2  ` respectively, enforce a linear transaction order. This means:

  - All reads and writes within `  Txn1  ` appear to have occurred at a single point in time, `  c1  ` .
  - All reads and writes within `  Txn2  ` appear to have occurred at a single point in time, `  c2  ` .
  - Crucially, `  c1  ` is earlier than `  c2  ` for committed writes, even if the writes occurred on different machines. If `  Txn2  ` performs only reads, `  c1  ` is earlier or at the same time as `  c2  ` .

This strong ordering means that if a subsequent read operation observes the effects of `  Txn2  ` , it also observes the effects of `  Txn1  ` . This property holds true for all successfully committed transactions.

On the other hand, if you use repeatable read isolation, the following scenario occurs for the same transactions:

  - `  Txn1  ` starts by reading data `  A  ` , creating its own snapshot of the database at that moment.
  - `  Txn2  ` then begins, reading data `  B  ` , and establishes its own snapshot.
  - Next, `  Txn1  ` modifies data `  A  ` , and successfully commits its changes.
  - `  Txn2  ` attempts to read data `  A  ` . Crucially, because it's operating at an earlier snapshot, `  Txn2  ` doesn't see the update `  Txn1  ` just made to `  A  ` . `  Txn2  ` reads the older value.
  - `  Txn2  ` modifies data `  B  ` and commits.

In this scenario, each transaction operates on its own consistent snapshot of the database, taken from the moment the transaction starts. This sequence can lead to a write skew anomaly if the write to `  B  ` by `  Txn2  ` was logically dependent on the value it read from `  A  ` . In essence, `  Txn2  ` made its updates based on outdated information, and its subsequent write might violate an application-level invariant. To prevent this scenario from arising, consider either [using `  SELECT...FOR UPDATE  ` for repeatable read isolation](/spanner/docs/use-select-for-update-repeatable-read) , or [creating check constraints in your schema](/spanner/docs/check-constraint/how-to) .

**Note:** Regardless of isolation level, when using [DML statements](/spanner/docs/dml-tasks#using-dml) , changes are immediately visible to subsequent read statements within the same transaction. However, if you use [mutations](/spanner/docs/modify-mutation-api) , changes are buffered locally and are only visible after the transaction commits.

#### Read and write guarantees on transaction failure

If a call to execute a transaction fails, the read and write guarantees you have depend on what error the underlying commit call failed with.

Spanner might execute a transaction's operations multiple times internally. If an execution attempt fails, the returned error specifies the conditions that occurred and indicates the guarantees you receive. However, if Spanner retries your transaction, any side effects from its operations (for example, changes to external systems or a system state outside a Spanner database) might occur multiple times.

When a Spanner transaction fails, the guarantees you receive for reads and writes depend on the specific error encountered during the commit operation.

For example, an error message such as "Row Not Found" or "Row Already Exists" indicates an issue during the writing of buffered mutations. This can occur if, for example, a row the client is attempting to update doesn't exist. In these scenarios:

  - **Reads are consistent:** Any data read during the transaction is guaranteed to be consistent up to the point of the error.
  - **Writes are not applied:** The mutations the transaction attempted aren't committed to the database.
  - **Row consistency:** The non-existence (or existing state) of the row that triggered the error is consistent with the reads performed within the transaction.

You can cancel asynchronous read operations in Spanner at any time without affecting other ongoing operations within the same transaction. This flexibility is useful if a higher-level operation is cancelled, or if you decide to abort a read based on initial results.

However, it's important to understand that requesting the cancellation of a read doesn't guarantee its immediate termination. After a cancellation request, the read operation might still:

  - **Successfully complete:** the read might finish processing and return results before the cancellation takes effect.
  - **Fail for another reason:** the read could terminate due to a different error, such as an abort.
  - **Return incomplete results:** the read might return partial results, which are then validated as part of the transaction commit process.

Cancelling a commit operation aborts the entire transaction, unless the transaction has already committed or failed due to another reason.

#### Atomicity, consistency, durability

In addition to isolation, Spanner provides the other ACID property guarantees:

  - **Atomicity** : A transaction is considered atomic if all its operations are completed successfully, or none at all. If any operation within a transaction fails, the entire transaction is rolled back to its original state, ensuring data integrity.
  - **Consistency** : A transaction must maintain the integrity of the database's rules and constraints. After a transaction completes, the database should be in a valid state, adhering to predefined rules.
  - **Durability** : After a transaction is committed, its changes are permanently stored in the database and persist in the event of system failures, power outages, or other disruptions.

### Performance

This section describes issues that affect read-write transaction performance.

#### Locking concurrency control

By default, Spanner permits multiple clients to interact with the same database concurrently in its default serializable isolation level. To maintain data consistency across these concurrent transactions, Spanner has a locking mechanism that uses both shared and exclusive locks. These read locks are only acquired for serializable transactions, but not for transactions that use repeatable read isolation.

When a serializable transaction performs a read operation, Spanner acquires shared read locks on the relevant data. These shared locks let other concurrent read operations access the same data. This concurrency is maintained until your transaction prepares to commit its changes.

In serializable isolation, during the commit phase, as writes are applied, the transaction attempts to upgrade its locks to exclusive locks. To achieve this, Spanner does the following:

  - Blocks any new shared read lock requests on the affected data.
  - Waits for all existing shared read locks on that data to be released.
  - After all shared read locks are cleared, it places an exclusive lock, granting it sole access to the data for the duration of the write.

When committing a transaction in repeatable read isolation, the transaction acquires exclusive locks for the written data. The transaction might have to wait for locks if a concurrent transaction is also committing writes to the same data.

Notes about locks:

  - **Granularity:** Spanner applies locks at the row-and-column granularity. This means that if transaction `  T1  ` holds a lock on column `  A  ` of row `  albumid  ` , transaction `  T2  ` can still concurrently write to column `  B  ` of the same row `  albumid  ` without conflict.

  - **Writes without reads:**
    
      - When there are no reads in the transaction, Spanner might not require an exclusive lock for writes without reads. Instead, it might use a writer shared lock. This is because the order of application for writes without reads is determined by their commit timestamps, letting multiple writers operate on the same item concurrently without conflict. An exclusive lock is only necessary if your transaction first reads the data it intends to write.
      - In repeatable read isolation, transactions commonly acquire exclusive locks for written cells at commit time.

  - **Secondary indexes for row lookups:** in serializable isolation, when performing reads within a read-write transaction, using secondary indexes can significantly improve performance. By using secondary indexes to limit the scanned rows to a smaller range, Spanner locks fewer rows in the table, thereby enabling greater concurrent modification of rows outside of that specific range.

  - **External resource exclusive access:** Spanner's internal locks are designed for data consistency within the Spanner database itself. Don't use them to guarantee exclusive access to resources outside of Spanner. Spanner can abort transactions for various reasons, including internal system optimizations like data movement across compute resources. If a transaction is retried (either explicitly by your application code or implicitly by client libraries like the Spanner JDBC driver), locks are only guaranteed to have been held during the successful commit attempt.

  - **Lock statistics:** to diagnose and investigate lock conflicts within your database, use the [Lock statistics](/spanner/docs/introspection/lock-statistics) introspection tool.

#### Deadlock detection

Spanner detects when multiple transactions might be deadlocked and forces all but one of the transactions to abort. Consider this scenario: `  Txn1  ` holds a lock on record `  A  ` and is waiting for a lock on record `  B  ` , while `  Txn2  ` holds a lock on record `  B  ` and is waiting for a lock on record `  A  ` . To resolve this, one of the transactions must abort, releasing its lock and allowing the other to proceed.

Spanner uses the standard wound-wait algorithm for deadlock detection. Under the hood, Spanner tracks the age of each transaction requesting conflicting locks. It lets older transactions abort younger ones. An older transaction is one whose earliest read, query, or commit occurred sooner.

By prioritizing older transactions, Spanner ensures that every transaction eventually acquires locks after it becomes old enough to have higher priority. For example, an older transaction needing a writer-shared lock can abort a younger transaction holding a reader-shared lock.

#### Distributed execution

Spanner can execute transactions on data that spans multiple servers, though this capability comes with a performance cost compared to single-server transactions.

What types of transactions might be distributed? Spanner can distribute responsibility for database rows across many servers. Typically, a row and its corresponding interleaved table rows are served by the same server, as are two rows in the same table with nearby keys. Spanner can perform transactions across rows on different servers. However, as a general rule, transactions affecting many co-located rows are faster and cheaper than those affecting many rows scattered throughout the database or a large table.

The most efficient transactions in Spanner include only the reads and writes that should be applied atomically. Transactions are fastest when all reads and writes access data in the same part of the key space.

## Read-only transactions

In addition to locking read-write transactions, Spanner offers read-only transactions.

Use a read-only transaction when you need to execute more than one read at the same timestamp. If you can express your read using one of Spanner's [single read methods](/spanner/docs/reads#single_read_methods) , you should use that single read method instead. The performance of using such a single read call should be comparable to the performance of a single read done in a read-only transaction.

If you are reading a large amount of data, consider using partitions to [read the data in parallel](/spanner/docs/reads#read_data_in_parallel) .

Because read-only transactions don't write, they don't hold locks and they don't block other transactions. Read-only transactions observe a consistent prefix of the transaction commit history, so your application always gets consistent data.

### Interface

Spanner provides an interface for executing a body of work in the context of a read-only transaction, with retries for transaction aborts.

### Example

The following example shows how to use a read-only transaction to get consistent data for two reads at the same timestamp:

### C++

``` cpp
void ReadOnlyTransaction(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto read_only = spanner::MakeReadOnlyTransaction();

  spanner::SqlStatement select(
      "SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
  using RowType = std::tuple<std::int64_t, std::int64_t, std::string>;

  // Read#1.
  auto rows1 = client.ExecuteQuery(read_only, select);
  std::cout << "Read 1 results\n";
  for (auto& row : spanner::StreamOf<RowType>(rows1)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " AlbumId: " << std::get<1>(*row)
              << " AlbumTitle: " << std::get<2>(*row) << "\n";
  }
  // Read#2. Even if changes occur in-between the reads the transaction ensures
  // that Read #1 and Read #2 return the same data.
  auto rows2 = client.ExecuteQuery(read_only, select);
  std::cout << "Read 2 results\n";
  for (auto& row : spanner::StreamOf<RowType>(rows2)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " AlbumId: " << std::get<1>(*row)
              << " AlbumTitle: " << std::get<2>(*row) << "\n";
  }
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Transactions;

public class QueryDataWithTransactionAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> QueryDataWithTransactionAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();
        using TransactionScope scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
        using var connection = new SpannerConnection(connectionString);

        // Opens the connection so that the Spanner transaction included in the TransactionScope
        // is read-only TimestampBound.Strong.
        await connection.OpenAsync(SpannerTransactionCreationOptions.ReadOnly, options: null, cancellationToken: default);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

        // Read #1.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine("SingerId : " + reader.GetFieldValue<string>("SingerId")
                    + " AlbumId : " + reader.GetFieldValue<string>("AlbumId")
                    + " AlbumTitle : " + reader.GetFieldValue<string>("AlbumTitle"));
            }
        }

        // Read #2. Even if changes occur in-between the reads,
        // the transaction ensures that Read #1 and Read #2
        // return the same data.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                albums.Add(new Album
                {
                    AlbumId = reader.GetFieldValue<int>("AlbumId"),
                    SingerId = reader.GetFieldValue<int>("SingerId"),
                    AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
                });
            }
        }
        scope.Complete();
        Console.WriteLine("Transaction complete.");
        return albums;
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
 "google.golang.org/api/iterator"
)

func readOnlyTransaction(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 ro := client.ReadOnlyTransaction()
 defer ro.Close()
 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := ro.Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     var singerID int64
     var albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }

 iter = ro.Read(ctx, "Albums", spanner.AllKeys(), []string{"SingerId", "AlbumId", "AlbumTitle"})
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
     var albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

### Java

``` java
static void readOnlyTransaction(DatabaseClient dbClient) {
  // ReadOnlyTransaction must be closed by calling close() on it to release resources held by it.
  // We use a try-with-resource block to automatically do so.
  try (ReadOnlyTransaction transaction = dbClient.readOnlyTransaction()) {
    try (ResultSet queryResultSet =
        transaction.executeQuery(
            Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"))) {
      while (queryResultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            queryResultSet.getLong(0), queryResultSet.getLong(1), queryResultSet.getString(2));
      }
    } // queryResultSet.close() is automatically called here
    try (ResultSet readResultSet =
        transaction.read(
          "Albums", KeySet.all(), Arrays.asList("SingerId", "AlbumId", "AlbumTitle"))) {
      while (readResultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            readResultSet.getLong(0), readResultSet.getLong(1), readResultSet.getString(2));
      }
    } // readResultSet.close() is automatically called here
  } // transaction.close() is automatically called here
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

// Gets a transaction object that captures the database state
// at a specific point in time
database.getSnapshot(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  const queryOne = 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums';

  try {
    // Read #1, using SQL
    const [qOneRows] = await transaction.run(queryOne);

    qOneRows.forEach(row => {
      const json = row.toJSON();
      console.log(
        `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
      );
    });

    const queryTwo = {
      columns: ['SingerId', 'AlbumId', 'AlbumTitle'],
    };

    // Read #2, using the `read` method. Even if changes occur
    // in-between the reads, the transaction ensures that both
    // return the same data.
    const [qTwoRows] = await transaction.read('Albums', queryTwo);

    qTwoRows.forEach(row => {
      const json = row.toJSON();
      console.log(
        `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
      );
    });

    console.log('Successfully executed read-only transaction.');
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    transaction.end();
    // Close the database when finished.
    await database.close();
  }
});
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads data inside of a read-only transaction.
 *
 * Within the read-only transaction, or "snapshot", the application sees
 * consistent view of the database at a particular timestamp.
 * Example:
 * ```
 * read_only_transaction($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_only_transaction(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $snapshot = $database->snapshot();
    $results = $snapshot->execute(
        'SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
    );
    print('Results from the first read:' . PHP_EOL);
    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }

    // Perform another read using the `read` method. Even if the data
    // is updated in-between the reads, the snapshot ensures that both
    // return the same data.
    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['SingerId', 'AlbumId', 'AlbumTitle']
    );

    print('Results from the second read:' . PHP_EOL);
    foreach ($results->rows() as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

### Python

``` python
def read_only_transaction(instance_id, database_id):
    """Reads data inside of a read-only transaction.

    Within the read-only transaction, or "snapshot", the application sees
    consistent view of the database at a particular timestamp.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot(multi_use=True) as snapshot:
        # Read using SQL.
        results = snapshot.execute_sql(
            "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
        )

        print("Results from first read:")
        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))

        # Perform another read using the `read` method. Even if the data
        # is updated in-between the reads, the snapshot ensures that both
        # return the same data.
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums", columns=("SingerId", "AlbumId", "AlbumTitle"), keyset=keyset
        )

        print("Results from second read:")
        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.snapshot do |snapshot|
  snapshot.execute("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").rows.each do |row|
    puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
  end

  # Even if changes occur in-between the reads, the transaction ensures that
  # both return the same data.
  snapshot.read("Albums", [:AlbumId, :AlbumTitle, :SingerId]).rows.each do |row|
    puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
  end
end
```

### Semantics

This section describes semantics for read-only transactions.

#### Snapshot read-only transactions

When a read-only transaction executes in Spanner, it performs all its reads at a single logical point in time. This means that both the read-only transaction and any other concurrent readers and writers see a consistent snapshot of the database at that specific moment.

These snapshot read-only transactions offer a simpler approach for consistent reads compared to locking read-write transactions. Here's why:

  - **No locks:** read-only transactions don't acquire locks. Instead, they operate by selecting a Spanner timestamp and executing all reads against that historical version of the data. Because they don't use locks, they won't block concurrent read-write transactions.
  - **No aborts:** these transactions never abort. While they might fail if their chosen read timestamp is garbage collected, Spanner's default garbage collection policy is typically generous enough that most applications won't encounter this issue.
  - **No commits or rollbacks:** read-only transactions don't require calls to `  sessions.commit  ` or `  sessions.rollback  ` and are actually prevented from doing so.

To execute a snapshot transaction, the client defines a timestamp bound, which instructs Spanner how to select a read timestamp. The types of timestamp bounds include the following:

  - **Strong reads:** these reads guarantee that you'll see the effects of all transactions committed before the read began. All rows within a single read are consistent. However, strong reads aren't repeatable, although strong reads do return a timestamp, and reading again at that same timestamp is repeatable. Two consecutive strong read-only transactions might produce different results due to concurrent writes. Queries on change streams must use this bound. For more details, see [TransactionOptions.ReadOnly.strong](/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly.FIELDS.strong) .
  - **Exact staleness:** this option executes reads at a timestamp you specify, either as an absolute timestamp or as a staleness duration relative to the current time. It ensures you observe a consistent prefix of the global transaction history up to that timestamp and blocks conflicting transactions that might commit with a timestamp less than or equal to the read timestamp. While slightly faster than bounded staleness modes, it might return older data. For more details, see [TransactionOptions.ReadOnly.read\_timestamp](/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly.FIELDS.read_timestamp) and [TransactionOptions.ReadOnly.exact\_staleness](/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly.FIELDS.exact_staleness) .
  - **Bounded staleness:** Spanner selects the newest timestamp within a user-defined staleness limit, allowing execution at the nearest available replica without blocking. All rows returned are consistent. Like strong reads, bounded staleness isn't repeatable, as different reads might execute at different timestamps even with the same bound. These reads operate in two phases (timestamp negotiation, then read) and are usually slightly slower than exact staleness, but they often return fresher results and are more likely to execute at a local replica. This mode is only available for single-use read-only transactions because timestamp negotiation requires knowing which rows will be read beforehand. For more details, see [TransactionOptions.ReadOnly.max\_staleness](/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly.FIELDS.max_staleness) and [TransactionOptions.ReadOnly.min\_read\_timestamp](/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly.FIELDS.min_read_timestamp) .

## Partitioned DML transactions

You can use [partitioned DML](/spanner/docs/dml-partitioned) to execute large-scale `  UPDATE  ` and `  DELETE  ` statements without encountering transaction limits or locking an entire table. Spanner achieves this by partitioning the key space and executing the DML statements on each partition within a separate read-write transaction.

To use non-partitioned DML, you execute statements within read-write transactions that you explicitly create in your code. For more details, see [Using DML](/spanner/docs/dml-tasks#using-dml) .

### Interface

Spanner provides the [TransactionOptions.partitionedDml](/spanner/docs/reference/rest/v1/TransactionOptions#partitioneddml) interface for executing a single partitioned DML statement.

### Examples

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

### Semantics

This section describes the semantics for partitioned DML.

#### Understanding partitioned DML execution

You can execute only one partitioned DML statement at a time, whether you are using a client library method or the Google Cloud CLI.

Partitioned transactions don't support commits or rollbacks. Spanner executes and applies the DML statement immediately. If you cancel the operation, or the operation fails, Spanner cancels all the executing partitions and doesn't start any remaining ones. However, Spanner doesn't roll back any partitions that have already executed.

#### Partitioned DML lock acquisition strategy

To reduce lock contention, partitioned DML acquires read locks only on rows that match the `  WHERE  ` clause. Smaller, independent transactions used for each partition also hold locks for less time.

## Old read timestamps and version garbage collection

Spanner performs version garbage collection to collect deleted or overwritten data and reclaim storage. By default, data older than one hour is reclaimed. Spanner can't perform reads at timestamps older than the configured `  VERSION_RETENTION_PERIOD  ` , which defaults to one hour but can be configured to up to one week. When reads become too old during execution, they fail and return the `  FAILED_PRECONDITION  ` error.

## Queries on change streams

A *change stream* is a schema object you can configure to monitor data modifications across an entire database, specific tables, or a defined set of columns within a database.

When you create a change stream, Spanner defines a corresponding SQL table-valued function (TVF). You can use this TVF to query the change records in the associated change stream with the [`  sessions.executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method. The TVF's name is generated from the change stream's name and always starts with `  READ_  ` .

All queries on change stream TVFs must be executed using the `  sessions.executeStreamingSql  ` API within a single-use read-only transaction with a strong read-only `  timestamp_bound  ` . The change stream TVF lets you specify `  start_timestamp  ` and `  end_timestamp  ` for the time range. All change records within the retention period are accessible using this strong read-only `  timestamp_bound  ` . All other [`  TransactionOptions  `](/spanner/docs/reference/rest/v1/TransactionOptions) are invalid for change stream queries.

Additionally, if [`  TransactionOptions.read_only.return_read_timestamp  `](/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly.FIELDS.return_read_timestamp) is set to `  true  ` , the [`  Transaction  `](/spanner/docs/reference/rest/v1/Transaction) message describing the transaction returns a special value of `  2^63 - 2  ` instead of a valid read timestamp. You should discard this special value and not use it for any subsequent queries.

For more information, see [Change streams query workflow](/spanner/docs/change-streams/details#query) .

## Idle Transactions

A transaction is considered idle if it has no outstanding reads or SQL queries and hasn't started one in the last 10 seconds. Spanner can abort idle transactions to prevent them from holding locks indefinitely. If an idle transaction is aborted, the commit fails and returns an `  ABORTED  ` error. Periodically executing a small query, such as `  SELECT 1  ` , within the transaction can prevent it from becoming idle.

## What's next

  - Learn more about [Spanner isolation levels](/spanner/docs/isolation-levels) .
