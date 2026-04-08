**PostgreSQL interface note:** The `  JSON  ` data type is not supported in PostgreSQL-dialect databases. Use the [`  JSONB  ` data type](/spanner/docs/working-with-jsonb) for PostgreSQL-dialect databases.

This page describes how to work with JSON using Spanner.

The JSON data type is a semi-structured data type used for holding JSON (JavaScript Object Notation) data. The specifications for the JSON format are described in [RFC 7159](https://tools.ietf.org/pdf/rfc7159.pdf) .

JSON is useful to supplement a relational schema for data that is sparse or has a loosely-defined or changing structure. However, the query optimizer relies on the relational model to efficiently filter, join, aggregate, and sort at scale. Queries over JSON will have fewer built-in optimizations and fewer affordances to inspect and tune performance.

## Specifications

Spanner JSON type stores a normalized representation of the input JSON document.

  - JSON can be nested to a maximum of 80 levels.
  - Whitespace is not preserved.
  - Comments are not supported. Transactions or queries with comments will fail.
  - Members of a JSON object are sorted lexicographically.
  - JSON array elements have their order preserved.
  - If a JSON object has duplicate keys, only the first one is preserved.
  - Primitive types (string, boolean, number, and null) have their type and value preserved.
      - String type values are preserved exactly.
      - Number type values are preserved, but may have their textual representation changed as a result of the normalization process. For example, an input number of 10000 may have a normalized representation of 1e+4. Number value preservation semantics are as follows:
          - Signed integers in the range of \[INT64\_MIN, INT64\_MAX\] are preserved.
          - Unsigned integers in the range of \[0, UINT64\_MAX\] are preserved.
          - Double values that can be roundtripped from string to double to string without decimal precision loss are preserved. If a double value cannot round trip in this manner, the transaction or query fails.
              - For example, `  SELECT JSON '2.2412421353246235436'  ` fails.
              - A functional workaround is `  PARSE_JSON('2.2412421353246235436', wide_number_mode=>'round')  ` , which returns `  JSON '2.2412421353246237'  ` .
  - Use the [`  TO_JSON()  `](/spanner/docs/reference/standard-sql/json_functions#to_json) , [`  JSON_OBJECT()  `](/spanner/docs/reference/standard-sql/json_functions#json_object) , and the [`  JSON_ARRAY()  `](/spanner/docs/reference/standard-sql/json_functions#json_array) functions to construct JSON documents in SQL. These functions implement the necessary quoting and escaping characters.

The maximum permitted size of the normalized document is 10 MB.

### Nullability

JSON `  null  ` values are treated as SQL non-NULL.

For example:

``` text
SELECT (JSON '{"a":null}').a IS NULL; -- Returns FALSE
SELECT (JSON '{"a":null}').b IS NULL; -- Returns TRUE

SELECT JSON_QUERY(JSON '{"a":null}', "$.a"); -- Returns a JSON 'null'
SELECT JSON_QUERY(JSON '{"a":null}', "$.b"); -- Returns a SQL NULL
```

### Encoding

JSON documents must be encoded in UTF-8. Transactions or queries with JSON documents encoded in other formats return an error.

## Create a table with JSON columns

A JSON column can be added to a table when the table is created. JSON type values can be nullable.

``` text
CREATE TABLE Venues (
  VenueId   INT64 NOT NULL,
  VenueName  STRING(1024),
  VenueAddress STRING(1024),
  VenueFeatures JSON,
  DateOpened  DATE,
) PRIMARY KEY(VenueId);
```

## Add and remove JSON columns from existing tables

A JSON column can also be added to and dropped from existing tables.

``` text
ALTER TABLE Venues ADD COLUMN VenueDetails JSON;
ALTER TABLE Venues DROP COLUMN VenueDetails;
```

The following sample shows how to add a `  JSON  ` column called `  VenueDetails  ` to the `  Venues  ` table using the gcloud CLI and Spanner client libraries.

### gcloud

``` text
gcloud spanner databases ddl update DATABASE_ID \ --instance=INSTANCE_ID \
--ddl="ALTER TABLE Venues ADD COLUMN VenueDetails JSON;"
```

### C++

``` cpp
void AddJsonColumn(google::cloud::spanner_admin::DatabaseAdminClient client,
                   std::string const& project_id,
                   std::string const& instance_id,
                   std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  auto metadata = client
                      .UpdateDatabaseDdl(database.FullName(), {R"""(
                        ALTER TABLE Venues ADD COLUMN VenueDetails JSON)"""})
                      .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "`Venues` table altered, new DDL:\n" << metadata->DebugString();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class AddJsonColumnAsyncSample
{
    public async Task AddJsonColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string alterStatement = "ALTER TABLE Venues ADD COLUMN VenueDetails JSON";

        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(alterStatement);
        await updateCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added the VenueDetails column.");
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

// addJsonColumn creates a column in the database of type JSON
func addJsonColumn(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("addJsonColumn: invalid database id %s", db)
 }

 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "ALTER TABLE Venues ADD COLUMN VenueDetails JSON",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added VenueDetails column\n")
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.concurrent.ExecutionException;

class AddJsonColumnSample {

  static void addJsonColumn() throws InterruptedException, ExecutionException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    addJsonColumn(projectId, instanceId, databaseId);
  }

  static void addJsonColumn(String projectId, String instanceId, String databaseId)
      throws InterruptedException, ExecutionException {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      // Wait for the operation to finish.
      // This will throw an ExecutionException if the operation fails.
      databaseAdminClient.updateDatabaseDdlAsync(
          DatabaseName.of(projectId, instanceId, databaseId),
          ImmutableList.of("ALTER TABLE Venues ADD COLUMN VenueDetails JSON")).get();
      System.out.printf("Successfully added column `VenueDetails`%n");
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

const request = ['ALTER TABLE Venues ADD COLUMN VenueDetails JSON'];

// Alter existing table to add a column.
const [operation] = await databaseAdminClient.updateDatabaseDdl({
  database: databaseAdminClient.databasePath(
    projectId,
    instanceId,
    databaseId,
  ),
  statements: request,
});

console.log(`Waiting for operation on ${databaseId} to complete...`);

await operation.promise();

console.log(
  `Added VenueDetails column to Venues table in database ${databaseId}.`,
);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds a JSON column to a table.
 * Example:
 * ```
 * add_json_column($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function add_json_column(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => ['ALTER TABLE Venues ADD COLUMN VenueDetails JSON']
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Added VenueDetails as a JSON column in Venues table' . PHP_EOL);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def add_json_column(instance_id, database_id):
    """Adds a new JSON column to the Venues table in the example database."""
    # instance_id = "your-spanner-instance"
    # database_id = "your-spanner-db-id"

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=["ALTER TABLE Venues ADD COLUMN VenueDetails JSON"],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        'Altered table "Venues" on database {} on instance {}.'.format(
            database_id, instance_id
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

database_path = db_admin_client.database_path project: project_id,
                                              instance: instance_id,
                                              database: database_id

statements = ["ALTER TABLE Venues ADD COLUMN VenueDetails JSON"]
job = db_admin_client.update_database_ddl database: database_path,
                                          statements: statements
job.wait_until_done!

puts "Added VenueDetails column to Venues table in database #{database_id}"
```

## Modify JSON data

The following sample shows how to update `  JSON  ` data using the gcloud CLI and Spanner client libraries.

### gcloud

``` text
gcloud spanner databases execute-sql DATABASE_ID --instance=INSTANCE_ID \
--sql="UPDATE Venues SET VenueDetails = JSON '{\"rating\": 9, \"open\": true}' WHERE VenueId = 1"
```

### C++

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void UpdateDataWithJson(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto venue19_details = spanner::Json(R"""(
        {"rating": 9, "open": true}
      )""");  // object
  auto venue4_details = spanner::Json(R"""(
        [
          {"name": "room 1", "open": true},
          {"name": "room 2", "open": false}
        ]
      )""");  // array
  auto venue42_details = spanner::Json(R"""(
        {
          "name": null,
          "open": {"Monday": true, "Tuesday": false},
          "tags": ["large", "airy"]
        }
      )""");  // nested
  auto update_venues =
      spanner::UpdateMutationBuilder(
          "Venues", {"VenueId", "VenueName", "VenueDetails", "LastUpdateTime"})
          .EmplaceRow(19, "Venue 19", venue19_details,
                      spanner::CommitTimestamp())
          .EmplaceRow(4, "Venue 4", venue4_details, spanner::CommitTimestamp())
          .EmplaceRow(42, "Venue 42", venue42_details,
                      spanner::CommitTimestamp())
          .Build();

  auto commit_result = client.Commit(spanner::Mutations{update_venues});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Insert was successful [spanner_update_data_with_json_column]\n";
}
```

### C\#

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class UpdateDataWithJsonAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueDetails { get; set; }
    }

    public async Task UpdateDataWithJsonAsync(string projectId, string instanceId, string databaseId)
    {
        List<Venue> venues = new List<Venue>
        {
            // If you are using .NET Core 3.1 or later, you can use System.Text.Json for serialization instead.
            new Venue
            {
                VenueId = 19,
                VenueDetails = JsonConvert.SerializeObject(new
                {
                    rating = 9,
                    open = true,
                })
            },
            new Venue
            {
                VenueId = 4,
                VenueDetails = JsonConvert.SerializeObject(new object[]
                {
                    new
                    {
                        name = "room 1",
                        open = true,
                    },
                    new
                    {
                        name = "room 2",
                        open = false,
                    },
                })
            },
            new Venue
            {
                VenueId = 42,
                VenueDetails = JsonConvert.SerializeObject(new
                {
                    name = "Central Park",
                    open = new
                    {
                        Monday = true,
                        Tuesday = false,
                    },
                    tags = new string[] {"large", "airy" },
                }),
            },
        };
        // Create connection to Cloud Spanner.
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        await Task.WhenAll(venues.Select(venue =>
        {
            // Update rows in the Venues table.
            using var cmd = connection.CreateUpdateCommand("Venues", new SpannerParameterCollection
            {
                    { "VenueId", SpannerDbType.Int64, venue.VenueId },
                    { "VenueDetails", SpannerDbType.Json, venue.VenueDetails }
            });
            return cmd.ExecuteNonQueryAsync();
        }));

        Console.WriteLine("Data updated.");
    }
}
```

### Go

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 "cloud.google.com/go/spanner"
)

// updateDataWithJsonColumn updates database with Json type values
func updateDataWithJsonColumn(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("addJsonColumn: invalid database id %s", db)
 }

 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 type VenueDetails struct {
     Name   spanner.NullString   `json:"name"`
     Rating spanner.NullFloat64  `json:"rating"`
     Open   interface{}          `json:"open"`
     Tags   []spanner.NullString `json:"tags"`
 }

 details_1 := spanner.NullJSON{Value: []VenueDetails{
     {Name: spanner.NullString{StringVal: "room1", Valid: true}, Open: true},
     {Name: spanner.NullString{StringVal: "room2", Valid: true}, Open: false},
 }, Valid: true}
 details_2 := spanner.NullJSON{Value: VenueDetails{
     Rating: spanner.NullFloat64{Float64: 9, Valid: true},
     Open:   true,
 }, Valid: true}

 details_3 := spanner.NullJSON{Value: VenueDetails{
     Name: spanner.NullString{Valid: false},
     Open: map[string]bool{"monday": true, "tuesday": false},
     Tags: []spanner.NullString{{StringVal: "large", Valid: true}, {StringVal: "airy", Valid: true}},
 }, Valid: true}

 cols := []string{"VenueId", "VenueDetails"}
 _, err = client.Apply(ctx, []*spanner.Mutation{
     spanner.Update("Venues", cols, []interface{}{4, details_1}),
     spanner.Update("Venues", cols, []interface{}{19, details_2}),
     spanner.Update("Venues", cols, []interface{}{42, details_3}),
 })

 if err != nil {
     return err
 }
 fmt.Fprintf(w, "Updated data to VenueDetails column\n")

 return nil
}
```

### Java

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.Mutation;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Value;
import com.google.common.collect.ImmutableList;

class UpdateJsonDataSample {

  static void updateJsonData() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      DatabaseClient client =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      updateJsonData(client);
    }
  }

  static void updateJsonData(DatabaseClient client) {
    client.write(
        ImmutableList.of(
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(4L)
                .set("VenueDetails")
                .to(
                    Value.json(
                        "[{\"name\":\"room 1\",\"open\":true},"
                            + "{\"name\":\"room 2\",\"open\":false}]"))
                .build(),
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(19L)
                .set("VenueDetails")
                .to(Value.json("{\"rating\":9,\"open\":true}"))
                .build(),
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(42L)
                .set("VenueDetails")
                .to(
                    Value.json(
                        "{\"name\":null,"
                            + "\"open\":{\"Monday\":true,\"Tuesday\":false},"
                            + "\"tags\":[\"large\",\"airy\"]}"))
                .build()));
    System.out.println("Venues successfully updated");
  }
}
```

### Node.js

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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
const venuesTable = database.table('Venues');

const data = [
  {
    VenueId: '19',
    VenueDetails: {rating: 9, open: true},
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '4',
    // VenueDetails must be specified as a string, as it contains a top-level
    // array of objects that should be inserted into a JSON column. If we were
    // to specify this value as an array instead of a string, the client
    // library would encode this value as ARRAY<JSON> instead of JSON.
    VenueDetails: `[
      {
        "name": null,
        "open": true
      },
      {
        "name": "room 2",
        "open": false
      },
      {
        "main hall": {
          "description": "this is the biggest space",
          "size": 200
        }
      }
    ]`,
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '42',
    VenueDetails: {
      name: null,
      open: {
        Monday: true,
        Tuesday: false,
      },
      tags: ['large', 'airy'],
    },
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
];
// Updates rows in the Venues table.
try {
  await venuesTable.update(data);
  console.log('Updated data.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Updates sample data in a table with a JSON column.
 *
 * Before executing this method, a new column Revenue has to be added to the Venues
 * table by applying the DDL statement "ALTER TABLE Venues ADD COLUMN VenueDetails JSON".
 *
 * Example:
 * ```
 * update_data_with_json_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_json_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->transaction(['singleUse' => true])
        ->updateBatch('Venues', [
            [
                'VenueId' => 4,
                'VenueDetails' =>
                    '[{"name":"room 1","open":true},{"name":"room 2","open":false}]'
            ],
            [
                'VenueId' => 19,
                'VenueDetails' => '{"rating":9,"open":true}'
            ],
            [
                'VenueId' => 42,
                'VenueDetails' =>
                    '{"name":null,"open":{"Monday":true,"Tuesday":false},"tags":["large","airy"]}'
            ],
        ])
        ->commit();

    print('Updated data.' . PHP_EOL);
}
````

### Python

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def update_data_with_json(instance_id, database_id):
    """Updates Venues tables in the database with the JSON
    column.

    This updates the `VenueDetails` column which must be created before
    running this sample. You can add the column by running the
    `add_json_column` sample or by running this DDL statement
     against your database:

        ALTER TABLE Venues ADD COLUMN VenueDetails JSON
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.batch() as batch:
        batch.update(
            table="Venues",
            columns=("VenueId", "VenueDetails"),
            values=[
                (
                    4,
                    JsonObject(
                        [
                            JsonObject({"name": "room 1", "open": True}),
                            JsonObject({"name": "room 2", "open": False}),
                        ]
                    ),
                ),
                (19, JsonObject(rating=9, open=True)),
                (
                    42,
                    JsonObject(
                        {
                            "name": None,
                            "open": {"Monday": True, "Tuesday": False},
                            "tags": ["large", "airy"],
                        }
                    ),
                ),
            ],
        )

    print("Updated data.")
```

### Ruby

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

rows = [{
  VenueId: 1,
  VenueDetails: { rating: 9, open: true }
}]
client.update "Venues", rows

# VenueDetails must be specified as a string, as it contains a top-level
# array of objects that should be inserted into a JSON column. If we were
# to specify this value as an array instead of a string, the client
# library would encode this value as ARRAY<JSON> instead of JSON.
venue_details_string = [
  {
    name: "room 1",
    open: true
  },
  {
    name: "room 2",
    open: false
  }
].to_json

rows = [{
  VenueId: 2,
  VenueDetails: venue_details_string
}]
client.update "Venues", rows

puts "Rows are updated."
```

## Index JSON data

You can accelerate querying JSON data by using [secondary indexes](/spanner/docs/secondary-indexes) and [search indexes](/spanner/docs/full-text-search/search-indexes) with your JSON data. Spanner doesn't support using JSON type columns as keys in secondary indexes.

### Use secondary index

Secondary indexes are useful when filtering against scalar values within a JSON document. To use secondary indexes with JSON, create a [generated column](/spanner/docs/generated-column/how-to) that extracts the relevant scalar data and [convert](/spanner/docs/reference/standard-sql/json_functions#categories) the data to an appropriate SQL type. You can then create a secondary index over this generated column. The index accelerates eligible queries that run against the generated column.

In the following example, you create a `  VenuesByCapacity  ` index that the database uses to find the venues with capacities greater than 1000. Instead of checking every row, Spanner uses the index to locate the relevant rows, which improves query performance, especially for large tables.

``` text
ALTER TABLE Venues
ADD COLUMN VenueCapacity INT64 AS (INT64(VenueDetails.capacity));

CREATE INDEX VenuesByCapacity ON Venue (VenueCapacity);

SELECT VenueName
FROM Venues
WHERE VenueCapacity > 1000;
```

### Use search indexes

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Search indexes are useful when you query against JSON documents that are dynamic or varied. Unlike secondary indexes, you can [create search indexes](/spanner/docs/reference/standard-sql/data-definition-language#create-search-index) over any JSON document stored in a JSON column. The search index automatically adapts to variations across JSON documents, between different rows, and over time.

In the following example, you create a `  VenuesByVenueDetails  ` search index that the database uses to find the venues with specific details such as size and operating schedule. Instead of checking every row, Spanner uses the index to locate the relevant rows, which improves query performance, especially for large tables.

``` text
ALTER TABLE Venues
ADD COLUMN VenueDetails_Tokens TOKENLIST AS (TOKENIZE_JSON(VenueDetails)) HIDDEN;

CREATE SEARCH INDEX VenuesByVenueDetails
ON Venue (VenueDetails_Tokens);

SELECT VenueName
FROM Venues
WHERE JSON_CONTAINS(VenueDetails, JSON '{"labels": ["large"], "open": {"Friday": true}}');
```

For more information, see [JSON search indexes](/spanner/docs/full-text-search/json-indexes) .

## Query JSON data

The following sample shows how to query `  JSON  ` data using the gcloud CLI and Spanner client libraries.

### gcloud

``` text
gcloud spanner databases execute-sql DATABASE_ID --instance=INSTANCE_ID \
--sql="SELECT VenueId, VenueDetails FROM Venues \
WHERE JSON_VALUE(VenueDetails, '$.rating') = '9'"
```

### C++

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithJsonParameter(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto rating9_details = spanner::Json(R"""(
        {"rating": 9}
      )""");  // object
  spanner::SqlStatement select(
      "SELECT VenueId, VenueDetails"
      "  FROM Venues"
      " WHERE JSON_VALUE(VenueDetails, '$.rating') ="
      "       JSON_VALUE(@details, '$.rating')",
      {{"details", spanner::Value(std::move(rating9_details))}});
  using RowType = std::tuple<std::int64_t, absl::optional<spanner::Json>>;

  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "VenueId: " << std::get<0>(*row) << ", ";
    auto venue_details = std::get<1>(*row).value();
    std::cout << "VenueDetails: " << venue_details << "\n";
  }
}
```

### C\#

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using Newtonsoft.Json;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithJsonParameterAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueDetails { get; set; }
    }

    public async Task<List<Venue>> QueryDataWithJsonParameterAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);

        // If you are using .NET Core 3.1 or later, you can use System.Text.Json for serialization instead.
        var jsonValue = JsonConvert.SerializeObject(new { rating = 9 });
        // Get all venues with rating 9.
        using var cmd = connection.CreateSelectCommand(
            @"SELECT VenueId, VenueDetails
              FROM Venues
              WHERE JSON_VALUE(VenueDetails, '$.rating') = JSON_VALUE(@details, '$.rating')",
            new SpannerParameterCollection
            {
                { "details", SpannerDbType.Json, jsonValue }
            });

        var venues = new List<Venue>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            venues.Add(new Venue
            {
                VenueId = reader.GetFieldValue<int>("VenueId"),
                VenueDetails = reader.GetFieldValue<string>("VenueDetails")
            });
        }
        return venues;
    }
}
```

### Go

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

// queryWithJsonParameter queries data on the JSON type column of the database
func queryWithJsonParameter(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("addJsonColumn: invalid database id %s", db)
 }
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 type VenueDetails struct {
     Name   spanner.NullString   `json:"name"`
     Rating spanner.NullFloat64  `json:"rating"`
     Open   interface{}          `json:"open"`
     Tags   []spanner.NullString `json:"tags"`
 }

 stmt := spanner.Statement{
     SQL: `SELECT VenueId, VenueDetails FROM Venues WHERE JSON_VALUE(VenueDetails, '$.rating') = JSON_VALUE(@details, '$.rating')`,
     Params: map[string]interface{}{
         "details": spanner.NullJSON{Value: VenueDetails{
             Rating: spanner.NullFloat64{Float64: 9, Valid: true},
         }, Valid: true},
     },
 }
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var venueID int64
     var venueDetails spanner.NullJSON
     if err := row.Columns(&venueID, &venueDetails); err != nil {
         return err
     }
     fmt.Fprintf(w, "The venue details for venue id %v is %v\n", venueID, venueDetails)
 }
}
```

### Java

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import com.google.cloud.spanner.Value;

class QueryWithJsonParameterSample {

  static void queryWithJsonParameter() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      DatabaseClient client =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      queryWithJsonParameter(client);
    }
  }

  static void queryWithJsonParameter(DatabaseClient client) {
    String exampleJson = "{\"rating\": 9}";
    Statement statement =
        Statement.newBuilder(
                "SELECT VenueId, VenueDetails\n"
                    + "FROM Venues\n"
                    + "WHERE JSON_VALUE(VenueDetails, '$.rating') = "
                    + "JSON_VALUE(@details, '$.rating')")
            .bind("details")
            .to(Value.json(exampleJson))
            .build();
    try (ResultSet resultSet = client.singleUse().executeQuery(statement)) {
      while (resultSet.next()) {
        System.out.printf(
            "VenueId: %s, VenueDetails: %s%n",
            resultSet.getLong("VenueId"), resultSet.getJson("VenueDetails"));
      }
    }
  }
}
```

### Node.js

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

// Gets a reference to a Cloud Spanner instance and database.
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const fieldType = {
  type: 'json',
};

const jsonValue = {rating: 9};

const query = {
  sql: `SELECT VenueId, VenueDetails FROM Venues
          WHERE JSON_VALUE(VenueDetails, '$.rating') = JSON_VALUE(@details, '$.rating')`,
  params: {
    details: jsonValue,
  },
  types: {
    details: fieldType,
  },
};

// Queries rows from the Venues table.
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `VenueId: ${json.VenueId}, Details: ${JSON.stringify(
        json.VenueDetails,
      )}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\Database;
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL with a NUMERIC parameter.
 * Example:
 * ```
 * query_data_with_json_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_json_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $exampleJson = [
        'rating' => 9,
        'open' => true,
    ];

    $results = $database->execute(
        'SELECT VenueId, VenueDetails FROM Venues ' .
        'WHERE JSON_VALUE(VenueDetails, \'$.rating\') = JSON_VALUE(@venueDetails, \'$.rating\')',
        [
            'parameters' => [
                'venueDetails' => json_encode($exampleJson)
            ],
            'types' => [
                'venueDetails' => Database::TYPE_JSON
            ]
        ]
    );

    foreach ($results as $row) {
        printf(
            'VenueId: %s, VenueDetails: %s' . PHP_EOL,
            $row['VenueId'],
            $row['VenueDetails']
        );
    }
}
````

### Python

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

example_json = json.dumps({"rating": 9})
param = {"details": example_json}
param_type = {"details": param_types.JSON}

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueDetails "
        "FROM Venues "
        "WHERE JSON_VALUE(VenueDetails, '$.rating') = "
        "JSON_VALUE(@details, '$.rating')",
        params=param,
        param_types=param_type,
    )

    for row in results:
        print("VenueId: {}, VenueDetails: {}".format(*row))
```

### Ruby

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client = spanner.client instance_id, database_id

query = "SELECT VenueId, VenueDetails FROM Venues
  WHERE JSON_VALUE(VenueDetails, '$.rating') = JSON_VALUE(@details, '$.rating')"
result = client.execute_query query,
                              params: { details: { rating: 9 } },
                              types: { details: :JSON }

result.rows.each do |row|
  puts "VenueId: #{row['VenueId']}, VenueDetails: #{row['VenueDetails']}"
end
```

## Restrictions

  - You can't use JSON columns in an `  ORDER BY  ` clause.
  - You can't use JSON type columns as primary keys or as keys in secondary indexes. For more information, see [Index JSON data](#index) .

## What's next

  - [JSON data type](/spanner/docs/reference/standard-sql/data-types#json_type)
  - [JSON functions](/spanner/docs/reference/standard-sql/json_functions)
  - JSON operators:
      - [Dot operator for field access](/spanner/docs/reference/standard-sql/operators#field_access_operator)
      - [Subscript operator for field and array element access](/spanner/docs/reference/standard-sql/operators#json_subscript_operator)
