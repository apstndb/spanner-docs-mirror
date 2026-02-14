Delete data in a Spanner Graph.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void DeleteData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  // Delete the 'Owns' relationships with key (1,7) and (2,20).
  auto delete_ownerships =
      spanner::DeleteMutationBuilder("PersonOwnAccount",
                                     spanner::KeySet()
                                         .AddKey(spanner::MakeKey(1, 7))
                                         .AddKey(spanner::MakeKey(2, 20)))
          .Build();

  // Delete transfers using the keys in the range [1, 8]
  auto delete_transfer_range =
      spanner::DeleteMutationBuilder(
          "AccountTransferAccount",
          spanner::KeySet().AddRange(spanner::MakeKeyBoundClosed(1),
                                     spanner::MakeKeyBoundOpen(8)))
          .Build();

  // Deletes rows from the Account table and the AccountTransferAccount
  // table, because the AccountTransferAccount table is defined with
  // ON DELETE CASCADE.
  auto delete_accounts_all =
      spanner::MakeDeleteMutation("Account", spanner::KeySet::All());

  // Deletes rows from the Person table and the PersonOwnAccount table,
  // because the PersonOwnAccount table is defined with ON DELETE CASCADE.
  auto delete_persons_all =
      spanner::MakeDeleteMutation("Person", spanner::KeySet::All());

  auto commit_result = client.Commit(
      spanner::Mutations{delete_ownerships, delete_transfer_range,
                         delete_accounts_all, delete_persons_all});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Delete was successful [spanner_delete_graph_data]\n";
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func deleteGraphData(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Apply a series of mutations to tables underpinning edges and nodes in the
 // example graph. If there are referential integrity constraints defined
 // between edges and the nodes they connect, the edge must be deleted
 // before the nodes that the edge connects are deleted.
 m := []*spanner.Mutation{
     // spanner.Key can be used to delete a specific set of rows.
     // Delete the PersonOwnAccount rows with the key values (1,7) and (2,20).
     spanner.Delete("PersonOwnAccount", spanner.Key{1, 7}),
     spanner.Delete("PersonOwnAccount", spanner.Key{2, 20}),

     // spanner.KeyRange can be used to delete rows with a key in a specific range.
     // Delete a range of rows where the key prefix is >=1 and <8
     spanner.Delete("AccountTransferAccount",
         spanner.KeyRange{Start: spanner.Key{1}, End: spanner.Key{8}, Kind: spanner.ClosedOpen}),

     // spanner.AllKeys can be used to delete all the rows in a table.
     // Delete all Account rows, which will also delete the remaining
     // AccountTransferAccount rows since it was defined with ON DELETE CASCADE.
     spanner.Delete("Account", spanner.AllKeys()),

     // Delete remaining Person rows, which will also delete the remaining
     // PersonOwnAccount rows since it was defined with ON DELETE CASCADE.
     spanner.Delete("Person", spanner.AllKeys()),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void deleteData(DatabaseClient dbClient) {
  List<Mutation> mutations = new ArrayList<>();

  // KeySet.Builder can be used to delete a specific set of rows.
  // Delete the PersonOwnAccount rows with the key values (1,7) and (2,20).
  mutations.add(
      Mutation.delete(
          "PersonOwnAccount",
          KeySet.newBuilder().addKey(Key.of(1, 7)).addKey(Key.of(2, 20)).build()));

  // KeyRange can be used to delete rows with a key in a specific range.
  // Delete a range of rows where the key prefix is >=1 and <8
  mutations.add(
      Mutation.delete(
          "AccountTransferAccount", KeySet.range(KeyRange.closedOpen(Key.of(1), Key.of(8)))));

  // KeySet.all() can be used to delete all the rows in a table.
  // Delete all Account rows, which will also delete the remaining
  // AccountTransferAccount rows since it was defined with ON DELETE CASCADE.
  mutations.add(Mutation.delete("Account", KeySet.all()));

  // KeySet.all() can be used to delete all the rows in a table.
  // Delete all Person rows, which will also delete the remaining
  // PersonOwnAccount rows since it was defined with ON DELETE CASCADE.
  mutations.add(Mutation.delete("Person", KeySet.all()));

  dbClient.write(mutations);
  System.out.printf("Records deleted.\n");
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def delete_data(instance_id, database_id):
    """Deletes sample data from the given database.

    The database, table, and data must already exist and can be created using
    `create_database` and `insert_data`.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    # Delete individual rows
    ownerships_to_delete = spanner.KeySet(keys=[[1, 7], [2, 20]])

    # Delete a range of rows where the column key is >=1 and <8
    transfers_range = spanner.KeyRange(start_closed=[1], end_open=[8])
    transfers_to_delete = spanner.KeySet(ranges=[transfers_range])

    # Delete Account/Person rows, which will also delete the remaining
    # AccountTransferAccount and PersonOwnAccount rows because
    # AccountTransferAccount and PersonOwnAccount are defined with
    # ON DELETE CASCADE
    remaining_nodes = spanner.KeySet(all_=True)

    with database.batch() as batch:
        batch.delete("PersonOwnAccount", ownerships_to_delete)
        batch.delete("AccountTransferAccount", transfers_to_delete)
        batch.delete("Account", remaining_nodes)
        batch.delete("Person", remaining_nodes)

    print("Deleted data.")
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
