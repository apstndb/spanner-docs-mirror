Use a read/write transaction.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Insert, update, and delete data using mutations](/spanner/docs/modify-mutation-api)
  - [Transactions overview](/spanner/docs/transactions)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
