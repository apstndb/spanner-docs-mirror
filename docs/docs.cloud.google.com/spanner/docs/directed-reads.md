The page describes Spanner directed reads and how to use it.

Directed reads in Spanner provide the flexibility to route [read-only transactions](/spanner/docs/samples/spanner-read-only-transaction) and [single reads](/spanner/docs/reads#single_read_methods) to a specific replica type or region within a dual-region or multi-region instance configuration or a custom regional configuration with optional read-only region(s).

## Benefits

Directed reads offer the following benefits:

  - Provide more control over load-balancing workloads across multiple regions to achieve more uniform CPU utilization and avoid over-provisioning of Spanner instances.
  - Enable workload isolation. You can direct your analytics workloads and [change streams reads](/spanner/docs/change-streams) to specific Spanner replicas to minimize impact to transactional workloads running on the same Spanner database.

## Supported query operations

<table>
<thead>
<tr class="header">
<th>Query operations</th>
<th>Are directed reads supported?</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/reads#perform-stale-read">Stale read</a></td>
<td>Yes</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reads#perform-strong-read">Strong read</a></td>
<td>Yes</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/transactions#read-write_transactions">Read-write transaction</a></td>
<td>No</td>
</tr>
</tbody>
</table>

Directed reads are not supported for [read-write transactions](/spanner/docs/modify-mutation-api) and [partitioned DML](/spanner/docs/dml-partitioned) types of bulk updates. This is because read-write transactions must be processed in the leader region. If directed reads are used in a read-write transaction, the transaction fails with a `  BAD_REQUEST  ` error.

## Limitations

Spanner directed reads have the following limitations:

  - You can only use directed reads in a Spanner instance that is in a [dual-region instance configuration](/spanner/docs/instance-configurations#dual-region-configurations) or [multi-region instance configuration](/spanner/docs/instance-configurations#multi-region-configurations) or a [custom regional configuration with optional read-only region(s)](/spanner/docs/create-manage-configurations#create-configuration) .
  - You can't use directed reads with read-write requests because write requests are always served by the leader region.
  - You can't use directed reads in the Google Cloud console or Google Cloud CLI. It is available using [REST](/spanner/docs/directed-reads#rest) and [RPC](/spanner/docs/directed-reads#rpc) APIs and the Spanner client libraries.
  - You can specify a maximum of 10 replicas in a single directed read.

## Before you begin

Consider the following before you use directed reads:

  - The application might incur additional latency if you are routing reads to a replica or region other than the one closest to the application.
  - You can route traffic based on:
      - Region name (For example: `  us-central1  ` ).
      - Replica type (Possible values: `  READ_ONLY  ` and `  READ_WRITE  ` ).
  - The auto failover option in directed reads is enabled by default. When auto failover option is enabled and all of the specified replicas are unavailable or unhealthy, Spanner routes requests to a replica outside the `  includeReplicas  ` list. If you disable the auto failover option and all of the specified replicas are unavailable or unhealthy, the directed reads request fails.

### Directed reads parameters

If you're using the REST or RPC API to perform directed reads, you must define these fields in the `  directedReadOptions  ` parameter. You can only include one of `  includeReplicas  ` or `  excludeReplicas  ` , not both.

  - `  includeReplicas  ` : Contains a repeated set of `  replicaSelections  ` . This list indicates the order in which directed reads to specific regions or replica types should be considered. You can specify a maximum of 10 `  includeReplicas  ` .
    
      - `  replicaSelections  ` : Consists of the `  location  ` or replica `  type  ` serving the directed reads request. If you use `  includeReplicas  ` , you must provide at least one of the following fields:
        
          - `  location  ` : The location serving the directed reads request. The location must be one of the regions within the dual-region or multi-region configuration of your database. If the location is not one of the regions within the dual-region or multi-region configuration of your database, requests won't be routed as expected. Instead, they are served by the nearest region. For example, you can direct reads to the location `  us-central1  ` on a database in the multi-region instance configuration `  nam6  ` .
            
            You can also specify the `  location  ` parameter with a `  leader  ` or `  non-leader  ` string literal. If you input the `  leader  ` value, Spanner directs your requests to the database's [leader replica](/spanner/docs/region-types#read-write) . Conversely, if you input the `  non-leader  ` value, Spanner fulfills the request in the nearest non-leader replica.
        
          - `  type  ` : The replica type serving the directed reads request. Possible types include `  READ_WRITE  ` and `  READ_ONLY  ` .
    
      - `  autoFailoverDisabled  ` : By default, this is set to `  False  ` , which means auto failover is enabled. When auto failover option is enabled, and all of the specified replicas are unavailable or unhealthy, Spanner routes requests to a replica outside the `  includeReplicas  ` list. If you disable the auto failover option and all of the specified replicas are unavailable or unhealthy, the directed reads request fails. Possible values include `  TRUE  ` for disabled and `  FALSE  ` for enabled.

  - `  excludeReplicas  ` : Contains a repeated set of `  replicaSelections  ` that is excluded from serving requests. Spanner doesn't route requests to replicas in this list.
    
      - `  replicaSelections  ` : The location or replica type that is excluded from serving the directed reads request. If you use `  excludeReplicas  ` , you must provide at least one of the following fields:
          - `  location  ` : The location that is excluded from serving the directed reads request.
          - `  type  ` : The replica type that is excluded from serving the directed reads request. Possible types include `  READ_WRITE  ` and `  READ_ONLY  ` .

To see an example of what a REST request body looks like, click the REST tab in the [Use directed reads](#use-directed-reads) section.

## Use directed reads

You can use the Spanner client libraries and REST and RPC APIs to perform directed reads.

### Client libraries

### C++

``` cpp
void DirectedRead(std::string const& project_id, std::string const& instance_id,
                  std::string const& database_id) {
  namespace spanner = ::google::cloud::spanner;

  // Create a client with a DirectedReadOption.
  auto client = spanner::Client(
      spanner::MakeConnection(
          spanner::Database(project_id, instance_id, database_id)),
      google::cloud::Options{}.set<spanner::DirectedReadOption>(
          spanner::ExcludeReplicas({spanner::ReplicaSelection("us-east4")})));

  spanner::SqlStatement select(
      "SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
  using RowType = std::tuple<std::int64_t, std::int64_t, std::string>;

  // A DirectedReadOption on the operation will override the option set
  // at the client level.
  auto rows = client.ExecuteQuery(
      std::move(select),
      google::cloud::Options{}.set<spanner::DirectedReadOption>(
          spanner::IncludeReplicas(
              {spanner::ReplicaSelection(spanner::ReplicaType::kReadWrite)},
              /*auto_failover_disabled=*/true)));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " AlbumId: " << std::get<1>(*row)
              << " AlbumTitle: " << std::get<2>(*row) << "\n";
  }
  std::cout << "Read completed for [spanner_directed_read]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using Google.Cloud.Spanner.V1;
using System.Collections.Generic;
using System.Threading.Tasks;

public class DirectedReadsAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> DirectedReadsAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);

        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
        // Set directed read options on a query or read command.
        cmd.DirectedReadOptions = new DirectedReadOptions
        {
            IncludeReplicas = new DirectedReadOptions.Types.IncludeReplicas
            {
                AutoFailoverDisabled = true,
                ReplicaSelections =
                {
                    new DirectedReadOptions.Types.ReplicaSelection
                    {
                        Location = "us-central1",
                        Type = DirectedReadOptions.Types.ReplicaSelection.Types.Type.ReadOnly
                    }
                }
            }
        };

        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
            });
        }
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
 sppb "cloud.google.com/go/spanner/apiv1/spannerpb"
 "google.golang.org/api/iterator"
)

//  Shows how to run a query with directed read options.
//  Only one of ExcludeReplicas or IncludeReplicas can be set
//  Each accepts a list of ReplicaSelections which contains Location and Type
//  * `location` - The location must be one of the regions within the
//  multi-region configuration of your database.
//  * `type` - The type of the replica
//  Some examples of using replica_selectors are:
//  * `location:us-east1` --> The "us-east1" replica(s) of any available type
//      will be used to process the request.
//  * `type:READ_ONLY`    --> The "READ_ONLY" type replica(s) in nearest
//  available location will be used to process the
//  request.
//  * `location:us-east1 type:READ_ONLY` --> The "READ_ONLY" type replica(s)
//  in location "us-east1" will be used to process the request.
//      IncludeReplicas also contains an option for AutoFailoverDisabled which when set
//  Spanner will not route requests to a replica outside the
//  IncludeReplicas list when all the specified replicas are unavailable
//  or unhealthy. The default value is `false`

func directedReadOptions(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 ctx := context.Background()
 directedReadOptionsForClient := &sppb.DirectedReadOptions{
     Replicas: &sppb.DirectedReadOptions_ExcludeReplicas_{
         ExcludeReplicas: &sppb.DirectedReadOptions_ExcludeReplicas{
             ReplicaSelections: []*sppb.DirectedReadOptions_ReplicaSelection{
                 {
                     Location: "us-east4",
                 },
             },
         },
     },
 }
 // DirectedReadOptions can be set at client level and will be used in all read-only transaction requests
 client, err := spanner.NewClientWithConfig(ctx, db, spanner.ClientConfig{DirectedReadOptions: directedReadOptionsForClient})
 if err != nil {
     return err
 }
 defer client.Close()

 // DirectedReadOptions set at Request level will override the options set at Client level.
 directedReadOptionsForRequest := &sppb.DirectedReadOptions{
     Replicas: &sppb.DirectedReadOptions_IncludeReplicas_{
         IncludeReplicas: &sppb.DirectedReadOptions_IncludeReplicas{
             ReplicaSelections: []*sppb.DirectedReadOptions_ReplicaSelection{
                 {
                     Type: sppb.DirectedReadOptions_ReplicaSelection_READ_ONLY,
                 },
             },
             AutoFailoverDisabled: true,
         },
     },
 }

 statement := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 // // Read rows while passing directedReadOptions directly to the query.
 iter := client.Single().QueryWithOptions(ctx, statement, spanner.QueryOptions{DirectedReadOptions: directedReadOptionsForRequest})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
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
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.Options;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import com.google.spanner.v1.DirectedReadOptions;
import com.google.spanner.v1.DirectedReadOptions.ExcludeReplicas;
import com.google.spanner.v1.DirectedReadOptions.IncludeReplicas;
import com.google.spanner.v1.DirectedReadOptions.ReplicaSelection;

public class DirectedReadSample {
  static void directedRead() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    directedRead(projectId, instanceId, databaseId);
  }

  static void directedRead(String projectId, String instanceId, String databaseId) {
    // Only one of excludeReplicas or includeReplicas can be set
    // Each accepts a list of replicaSelections which contains location and type
    //   * `location` - The location must be one of the regions within the
    //      multi-region configuration of your database.
    //   * `type` - The type of the replica
    // Some examples of using replicaSelectors are:
    //   * `location:us-east1` --> The "us-east1" replica(s) of any available type
    //                             will be used to process the request.
    //   * `type:READ_ONLY`    --> The "READ_ONLY" type replica(s) in nearest
    // .                            available location will be used to process the
    //                             request.
    //   * `location:us-east1 type:READ_ONLY` --> The "READ_ONLY" type replica(s)
    //                          in location "us-east1" will be used to process
    //                          the request.
    //  includeReplicas also contains an option called autoFailoverDisabled, which when set to true
    //  will instruct Spanner to not route requests to a replica outside the
    //  includeReplicas list when all the specified replicas are unavailable
    //  or unhealthy. Default value is `false`.
    final DirectedReadOptions directedReadOptionsForClient =
        DirectedReadOptions.newBuilder()
            .setExcludeReplicas(
                ExcludeReplicas.newBuilder()
                    .addReplicaSelections(
                        ReplicaSelection.newBuilder().setLocation("us-east4").build())
                    .build())
            .build();

    // You can set default `DirectedReadOptions` for a Spanner client. These options will be applied
    // to all read-only transactions that are executed by this client, unless specific
    // DirectedReadOptions are set for a query.
    // Directed read can only be used for read-only transactions. The default options will be
    // ignored for any read/write transaction that the client executes.
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .setDirectedReadOptions(directedReadOptionsForClient)
            .build()
            .getService()) {
      final DatabaseClient dbClient =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));

      // DirectedReadOptions at request level will override the options set at
      // client level (through SpannerOptions).
      final DirectedReadOptions directedReadOptionsForRequest =
          DirectedReadOptions.newBuilder()
              .setIncludeReplicas(
                  IncludeReplicas.newBuilder()
                      .addReplicaSelections(
                          ReplicaSelection.newBuilder()
                              .setType(ReplicaSelection.Type.READ_WRITE)
                              .build())
                      .setAutoFailoverDisabled(true)
                      .build())
              .build();

      // Read rows while passing DirectedReadOptions directly to the query.
      try (ResultSet rs =
          dbClient
              .singleUse()
              .executeQuery(
                  Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"),
                  Options.directedRead(directedReadOptionsForRequest))) {
        while (rs.next()) {
          System.out.printf(
              "SingerId: %d, AlbumId: %d, AlbumTitle: %s\n",
              rs.getLong(0), rs.getLong(1), rs.getString(2));
        }
        System.out.println("Successfully executed read-only transaction with directedReadOptions");
      }
    }
  }
}
```

### Node.js

``` javascript
// Imports the Google Cloud Spanner client library
const {Spanner, protos} = require('@google-cloud/spanner');

// Only one of excludeReplicas or includeReplicas can be set
// Each accepts a list of replicaSelections which contains location and type
//   * `location` - The location must be one of the regions within the
//      multi-region configuration of your database.
//   * `type` - The type of the replica
// Some examples of using replicaSelectors are:
//   * `location:us-east1` --> The "us-east1" replica(s) of any available type
//                             will be used to process the request.
//   * `type:READ_ONLY`    --> The "READ_ONLY" type replica(s) in nearest
//.                            available location will be used to process the
//                             request.
//   * `location:us-east1 type:READ_ONLY` --> The "READ_ONLY" type replica(s)
//                          in location "us-east1" will be used to process
//                          the request.
//  includeReplicas also contains an option for autoFailover which when set
//  Spanner will not route requests to a replica outside the
//  includeReplicas list when all the specified replicas are unavailable
//  or unhealthy. The default value is `false`
const directedReadOptionsForClient = {
  excludeReplicas: {
    replicaSelections: [
      {
        location: 'us-east4',
      },
    ],
  },
};

// Instantiates a client with directedReadOptions
const spanner = new Spanner({
  projectId: projectId,
  directedReadOptions: directedReadOptionsForClient,
});

async function spannerDirectedReads() {
  // Gets a reference to a Cloud Spanner instance and backup
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);
  const directedReadOptionsForRequest = {
    includeReplicas: {
      replicaSelections: [
        {
          type: protos.google.spanner.v1.DirectedReadOptions.ReplicaSelection
            .Type.READ_ONLY,
        },
      ],
      autoFailoverDisabled: true,
    },
  };

  await database.getSnapshot(async (err, transaction) => {
    if (err) {
      console.error(err);
      return;
    }
    try {
      // Read rows while passing directedReadOptions directly to the query.
      // These will override the options passed at Client level.
      const [rows] = await transaction.run({
        sql: 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums',
        directedReadOptions: directedReadOptionsForRequest,
      });
      rows.forEach(row => {
        const json = row.toJSON();
        console.log(
          `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
        );
      });
      console.log(
        'Successfully executed read-only transaction with directedReadOptions',
      );
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      transaction.end();
      // Close the database when finished.
      await database.close();
    }
  });
}
spannerDirectedReads();
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\V1\DirectedReadOptions\ReplicaSelection\Type as ReplicaType;

/**
 * Queries sample data from the database with directed read options.
 * Example:
 * ```
 * directed_read($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function directed_read(string $instanceId, string $databaseId): void
{
    $directedReadOptionsForClient = [
        'directedReadOptions' => [
            'excludeReplicas' => [
                'replicaSelections' => [
                    [
                        'location' => 'us-east4'
                    ]
                ]
            ]
        ]
    ];

    $directedReadOptionsForRequest = [
        'directedReadOptions' => [
            'includeReplicas' => [
                'replicaSelections' => [
                    [
                        'type' => ReplicaType::READ_WRITE
                    ]
                ],
                'autoFailoverDisabled' => true
            ]
        ]
    ];

    $spanner = new SpannerClient($directedReadOptionsForClient);
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);
    $snapshot = $database->snapshot();

    // directedReadOptions at Request level will override the options set at
    // Client level
    $results = $snapshot->execute(
        'SELECT SingerId, AlbumId, AlbumTitle FROM Albums',
        $directedReadOptionsForRequest
    );

    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

directed_read_options_for_client = {
    "exclude_replicas": {
        "replica_selections": [
            {
                "location": "us-east4",
            },
        ],
    },
}

# directed_read_options can be set at client level and will be used in all
# read-only transaction requests
spanner_client = spanner.Client(
    directed_read_options=directed_read_options_for_client
)
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

directed_read_options_for_request = {
    "include_replicas": {
        "replica_selections": [
            {
                "type_": DirectedReadOptions.ReplicaSelection.Type.READ_ONLY,
            },
        ],
        "auto_failover_disabled": True,
    },
}

with database.snapshot() as snapshot:
    # Read rows while passing directed_read_options directly to the query.
    # These will override the options passed at Client level.
    results = snapshot.execute_sql(
        "SELECT SingerId, AlbumId, AlbumTitle FROM Albums",
        directed_read_options=directed_read_options_for_request,
    )

    for row in results:
        print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

### Ruby

``` ruby
require "google/cloud/spanner"

##
# This is a snippet for showcasing how to pass in directed read options.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_directed_read project_id:, instance_id:, database_id:
  # Only one of exclude_replicas or include_replicas can be set.
  # Each accepts a list of replica_selections which contains location and type
  #   * `location` - The location must be one of the regions within the
  #      multi-region configuration of your database.
  #   * `type` - The type of the replica
  # Some examples of using replicaSelectors are:
  #   * `location:us-east1` --> The "us-east1" replica(s) of any available type
  #                             will be used to process the request.
  #   * `type:READ_ONLY`    --> The "READ_ONLY" type replica(s) in the nearest
  # .                            available location will be used to process the
  #                             request.
  #   * `location:us-east1 type:READ_ONLY` --> The "READ_ONLY" type replica(s)
  #                          in location "us-east1" will be used to process
  #                          the request.
  #  include_replicas also contains an option for auto_failover_disabled. If set
  #  Spanner will not route requests to a replica outside the
  #  include_replicas list even if all the specified replicas are
  #  unavailable or unhealthy. The default value is `false`.
  directed_read_options_for_client = {
    include_replicas: {
      replica_selections: [{ location: "us-east4" }]
    }
  }

  # Instantiates a client with directedReadOptions
  spanner = Google::Cloud::Spanner.new project: project_id
  client  = spanner.client instance_id, database_id, directed_read_options: directed_read_options_for_client

  directed_read_options = {
    include_replicas: {
      replica_selections: [{ type: "READ_WRITE" }],
      auto_failover_disabled: true
    }
  }

  result = client.execute_sql "SELECT SingerId, AlbumId, AlbumTitle FROM Albums", directed_read_options: directed_read_options
  result.rows.each do |row|
    puts "SingerId: #{row[:SingerId]}"
    puts "AlbumId: #{row[:AlbumId]}"
    puts "AlbumTitle: #{row[:AlbumTitle]}"
  end
  puts "Successfully executed read-only transaction with directed_read_options"
end
```

### REST

You can use the following REST APIs to perform directed reads:

  - [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql)
  - [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql)
  - [`  read  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read)
  - [`  streamingRead  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/streamingRead)

For example, to perform directed reads in `  us-central1  ` using `  executeSQL  ` :

1.  Click [`  projects.instances.databases.sessions.executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#try-it) .

2.  For **session** , enter:
    
    ``` text
    projects/<VAR>PROJECT-ID</VAR>/instances/<VAR>INSTANCE-ID</VAR>/databases/<VAR>DATABASE-ID</VAR>/sessions/<VAR>SESSION-ID</VAR>
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-ID : the instance ID.
      - DATABASE-ID : the database ID.
      - SESSION-ID : the session ID. You receive the `  SESSION-ID  ` value when you [create a session](/spanner/docs/getting-started/rest#create_a_session) .

3.  For **Request body** , use the following:
    
    ``` text
    {
      "directedReadOptions": {
        "includeReplicas": {
          "replicaSelections": [
            {
              "location": "us-central1",
            }
          ]
        }
      },
      "sql": "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
    }
    ```

4.  Click **Execute** . The response shows the query results.

### RPC

You can use the following RPC APIs to perform directed reads:

  - [`  ExecuteSql  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql)
  - [`  ExecuteStreamingSql  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql)
  - [`  Read  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read)
  - [`  StreamingRead  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.StreamingRead)

## Monitoring

Spanner provides a latency metric to help you monitor directed reads activities in your instances. The metric is available in [Cloud Monitoring](/spanner/docs/monitoring-cloud) .

  - `  spanner.googleapis.com/api/read_request_latencies_by_serving_location  `

You can filter this metric using the `  /serving_location  ` or `  /is_directed_read  ` fields. The `  /serving location  ` field indicates the location of the Spanner server where the request is served from. The `  /is_directed_read  ` field indicates whether the directed reads option is enabled.

For a full list of available metrics, see [metrics list for Spanner](/monitoring/api/metrics_gcp_p_z#gcp-spanner) .

## What's next

  - Learn how to perform [Reads outside of transactions](/spanner/docs/reads) .
