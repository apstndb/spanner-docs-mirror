**Preview — [Batch write](/spanner/docs/batch-write)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

This page describes Spanner batch write requests and how you can use them to modify your Spanner data.

You can use Spanner batch write to insert, update, or delete multiple rows in your Spanner tables. Spanner batch write supports low latency writes without a read operation, and returns responses as mutations are applied in batches. To use batch write, you group related mutations together, and all mutations in a group are committed atomically. The mutations across groups are applied in an unspecified order and are independent of one another (non-atomic). Spanner doesn't need to wait for all mutations to be applied before sending a response, which means that batch write allows for partial failure. You can also execute multiple batch writes at a time. For more information, see [How to use batch write](#how-to) .

## Use cases

Spanner batch write is especially useful if you want to commit a large number of writes without a read operation, but don't require an atomic transaction for all your mutations.

If you want to batch your DML requests, use [batch DML](/spanner/docs/dml-tasks#use-batch) to modify your Spanner data. For more information on the differences between DML and mutations, see [Comparing DML and mutations](/spanner/docs/dml-versus-mutations) .

For single mutation requests, we recommend using a [locking read-write transaction](/spanner/docs/transactions#read-write_transactions) .

## Limitations

Spanner batch write has the following limitations:

  - **Spanner batch write is not available using the Google Cloud console or Google Cloud CLI.** It is only available using REST and RPC APIs and the Spanner client libraries.

  - **[Replay protection](/spanner/docs/sessions#handle_errors_for_write_transactions_that_are_not_idempotent) is not supported using batch write.** It's possible for mutations to be applied more than once, and a mutation that is applied more than once might result in a failure. For example, if an insert mutation is replayed, it might produce an already exists error, or if you use generated or commit timestamp-based keys in the mutation, it might result in additional rows being added to the table. We recommend structuring your writes to be idempotent to avoid this issue.

  - **You can't rollback a completed batch write request.** You can cancel an in-progress batch write request. If you cancel an in-progress batch write, mutations in uncompleted groups are rolled back. Mutations in completed groups are committed to the database.

  - **The maximum size for a batch write request is the same as the limit for a commit request.** For more information, see [Limits for creating, reading, updating, and deleting data](/spanner/quotas#limits-for) .

## How to use batch write

To use batch write, you must have the `  spanner.databases.write  ` permission on the database that you want to modify. You can batch write mutations non-atomically in a single call using a [REST](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchWrite) or [RPC API](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BatchWrite) request call.

You should group the following mutation types together when using batch write:

  - Inserting rows with the same primary key prefix in both the parent and child tables.
  - Inserting rows into tables with a foreign key relationship between the tables.
  - Other types of related mutations depending on your database schema and application logic.

You can also batch write using the Spanner client libraries. The following code example updates the `  Singers  ` table with new rows.

### Client libraries

### Java

``` java
import com.google.api.gax.rpc.ServerStream;
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.Mutation;
import com.google.cloud.spanner.MutationGroup;
import com.google.cloud.spanner.Options;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.common.collect.ImmutableList;
import com.google.rpc.Code;
import com.google.spanner.v1.BatchWriteResponse;

public class BatchWriteAtLeastOnceSample {

  /***
   * Assume DDL for the underlying database:
   * <pre>{@code
   *   CREATE TABLE Singers (
   *     SingerId   INT64 NOT NULL,
   *     FirstName  STRING(1024),
   *     LastName   STRING(1024),
   *   ) PRIMARY KEY (SingerId)
   *
   *   CREATE TABLE Albums (
   *     SingerId     INT64 NOT NULL,
   *     AlbumId      INT64 NOT NULL,
   *     AlbumTitle   STRING(1024),
   *   ) PRIMARY KEY (SingerId, AlbumId),
   *   INTERLEAVE IN PARENT Singers ON DELETE CASCADE
   * }</pre>
   */

  private static final MutationGroup MUTATION_GROUP1 =
      MutationGroup.of(
          Mutation.newInsertOrUpdateBuilder("Singers")
              .set("SingerId")
              .to(16)
              .set("FirstName")
              .to("Scarlet")
              .set("LastName")
              .to("Terry")
              .build());
  private static final MutationGroup MUTATION_GROUP2 =
      MutationGroup.of(
          Mutation.newInsertOrUpdateBuilder("Singers")
              .set("SingerId")
              .to(17)
              .set("FirstName")
              .to("Marc")
              .build(),
          Mutation.newInsertOrUpdateBuilder("Singers")
              .set("SingerId")
              .to(18)
              .set("FirstName")
              .to("Catalina")
              .set("LastName")
              .to("Smith")
              .build(),
          Mutation.newInsertOrUpdateBuilder("Albums")
              .set("SingerId")
              .to(17)
              .set("AlbumId")
              .to(1)
              .set("AlbumTitle")
              .to("Total Junk")
              .build(),
          Mutation.newInsertOrUpdateBuilder("Albums")
              .set("SingerId")
              .to(18)
              .set("AlbumId")
              .to(2)
              .set("AlbumTitle")
              .to("Go, Go, Go")
              .build());

  static void batchWriteAtLeastOnce() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    batchWriteAtLeastOnce(projectId, instanceId, databaseId);
  }

  static void batchWriteAtLeastOnce(String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      DatabaseId dbId = DatabaseId.of(projectId, instanceId, databaseId);
      final DatabaseClient dbClient = spanner.getDatabaseClient(dbId);

      // Creates and issues a BatchWrite RPC request that will apply the mutation groups
      // non-atomically and respond back with a stream of BatchWriteResponse.
      ServerStream<BatchWriteResponse> responses =
          dbClient.batchWriteAtLeastOnce(
              ImmutableList.of(MUTATION_GROUP1, MUTATION_GROUP2),
              Options.tag("batch-write-tag"));

      // Iterates through the results in the stream response and prints the MutationGroup indexes,
      // commit timestamp and status.
      for (BatchWriteResponse response : responses) {
        if (response.getStatus().getCode() == Code.OK_VALUE) {
          System.out.printf(
              "Mutation group indexes %s have been applied with commit timestamp %s",
              response.getIndexesList(), response.getCommitTimestamp());
        } else {
          System.out.printf(
              "Mutation group indexes %s could not be applied with error code %s and "
                  + "error message %s", response.getIndexesList(),
              Code.forNumber(response.getStatus().getCode()), response.getStatus().getMessage());
        }
      }
    }
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
 sppb "cloud.google.com/go/spanner/apiv1/spannerpb"
 "google.golang.org/grpc/status"
)

// batchWrite demonstrates writing mutations to a Spanner database through
// BatchWrite API - https://pkg.go.dev/cloud.google.com/go/spanner#Client.BatchWrite
func batchWrite(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Database is assumed to exist - https://cloud.google.com/spanner/docs/getting-started/go#create_a_database
 singerColumns := []string{"SingerId", "FirstName", "LastName"}
 albumColumns := []string{"SingerId", "AlbumId", "AlbumTitle"}
 mutationGroups := make([]*spanner.MutationGroup, 2)

 mutationGroup1 := []*spanner.Mutation{
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{16, "Scarlet", "Terry"}),
 }
 mutationGroups[0] = &spanner.MutationGroup{Mutations: mutationGroup1}

 mutationGroup2 := []*spanner.Mutation{
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{17, "Marc", ""}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{18, "Catalina", "Smith"}),
     spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{17, 1, "Total Junk"}),
     spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{18, 2, "Go, Go, Go"}),
 }
 mutationGroups[1] = &spanner.MutationGroup{Mutations: mutationGroup2}
 iter := client.BatchWrite(ctx, mutationGroups)
 // See https://pkg.go.dev/cloud.google.com/go/spanner#BatchWriteResponseIterator.Do
 doFunc := func(response *sppb.BatchWriteResponse) error {
     if err = status.ErrorProto(response.GetStatus()); err == nil {
         fmt.Fprintf(w, "Mutation group indexes %v have been applied with commit timestamp %v",
             response.GetIndexes(), response.GetCommitTimestamp())
     } else {
         fmt.Fprintf(w, "Mutation group indexes %v could not be applied with error %v",
             response.GetIndexes(), err)
     }
     // Return an actual error as needed.
     return nil
 }
 return iter.Do(doFunc)
}
```

### Node

``` javascript
// Imports the Google Cloud client library
const {Spanner, MutationGroup} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// Create Mutation Groups
/**
 * Related mutations should be placed in a group, such as insert mutations for both a parent and a child row.
 * A group must contain related mutations.
 * Please see {@link https://cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchWriteRequest.MutationGroup}
 * for more details and examples.
 */
const mutationGroup1 = new MutationGroup();
mutationGroup1.insert('Singers', {
  SingerId: 1,
  FirstName: 'Scarlet',
  LastName: 'Terry',
});

const mutationGroup2 = new MutationGroup();
mutationGroup2.insert('Singers', {
  SingerId: 2,
  FirstName: 'Marc',
});
mutationGroup2.insert('Singers', {
  SingerId: 3,
  FirstName: 'Catalina',
  LastName: 'Smith',
});
mutationGroup2.insert('Albums', {
  AlbumId: 1,
  SingerId: 2,
  AlbumTitle: 'Total Junk',
});
mutationGroup2.insert('Albums', {
  AlbumId: 2,
  SingerId: 3,
  AlbumTitle: 'Go, Go, Go',
});

const options = {
  transactionTag: 'batch-write-tag',
};

try {
  database
    .batchWriteAtLeastOnce([mutationGroup1, mutationGroup2], options)
    .on('error', console.error)
    .on('data', response => {
      // Check the response code of each response to determine whether the mutation group(s) were applied successfully.
      if (response.status.code === 0) {
        console.log(
          `Mutation group indexes ${
            response.indexes
          }, have been applied with commit timestamp ${Spanner.timestamp(
            response.commitTimestamp,
          ).toJSON()}`,
        );
      }
      // Mutation groups that fail to commit trigger a response with a non-zero status code.
      else {
        console.log(
          `Mutation group indexes ${response.indexes}, could not be applied with error code ${response.status.code}, and error message ${response.status.message}`,
        );
      }
    })
    .on('end', () => {
      console.log('Request completed successfully');
    });
} catch (err) {
  console.log(err);
}
```

### Python

``` python
def batch_write(instance_id, database_id):
    """Inserts sample data into the given database via BatchWrite API.

    The database and table must already exist and can be created using
    `create_database`.
    """
    from google.rpc.code_pb2 import OK

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.mutation_groups() as groups:
        group1 = groups.group()
        group1.insert_or_update(
            table="Singers",
            columns=("SingerId", "FirstName", "LastName"),
            values=[
                (16, "Scarlet", "Terry"),
            ],
        )

        group2 = groups.group()
        group2.insert_or_update(
            table="Singers",
            columns=("SingerId", "FirstName", "LastName"),
            values=[
                (17, "Marc", ""),
                (18, "Catalina", "Smith"),
            ],
        )
        group2.insert_or_update(
            table="Albums",
            columns=("SingerId", "AlbumId", "AlbumTitle"),
            values=[
                (17, 1, "Total Junk"),
                (18, 2, "Go, Go, Go"),
            ],
        )

        for response in groups.batch_write():
            if response.status.code == OK:
                print(
                    "Mutation group indexes {} have been applied with commit timestamp {}".format(
                        response.indexes, response.commit_timestamp
                    )
                )
            else:
                print(
                    "Mutation group indexes {} could not be applied with error {}".format(
                        response.indexes, response.status
                    )
                )
```

### C++

``` cpp
namespace spanner = ::google::cloud::spanner;
// Use upserts as mutation groups are not replay protected.
auto commit_results = client.CommitAtLeastOnce({
    // group #0
    spanner::Mutations{
        spanner::InsertOrUpdateMutationBuilder(
            "Singers", {"SingerId", "FirstName", "LastName"})
            .EmplaceRow(16, "Scarlet", "Terry")
            .Build(),
    },
    // group #1
    spanner::Mutations{
        spanner::InsertOrUpdateMutationBuilder(
            "Singers", {"SingerId", "FirstName", "LastName"})
            .EmplaceRow(17, "Marc", "")
            .EmplaceRow(18, "Catalina", "Smith")
            .Build(),
        spanner::InsertOrUpdateMutationBuilder(
            "Albums", {"SingerId", "AlbumId", "AlbumTitle"})
            .EmplaceRow(17, 1, "Total Junk")
            .EmplaceRow(18, 2, "Go, Go, Go")
            .Build(),
    },
});
for (auto& commit_result : commit_results) {
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Mutation group indexes [";
  for (auto index : commit_result->indexes) std::cout << " " << index;
  std::cout << " ]: ";
  if (commit_result->commit_timestamp) {
    auto const& ts = *commit_result->commit_timestamp;
    std::cout << "Committed at " << ts.get<absl::Time>().value();
  } else {
    std::cout << commit_result->commit_timestamp.status();
  }
  std::cout << "\n";
}
```

## What's next

  - Learn more about [Spanner transactions](/spanner/docs/transactions) .
