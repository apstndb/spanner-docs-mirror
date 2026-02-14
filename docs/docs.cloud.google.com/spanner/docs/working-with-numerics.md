Spanner supports a `  NUMERIC  ` data type in both GoogleSQL and PostgreSQL databases.

## GoogleSQL NUMERIC

The GoogleSQL [`  NUMERIC  `](/spanner/docs/reference/standard-sql/data-types#numeric_type) is an exact numeric data type capable of representing an exact numeric value with a precision of 38 and scale of 9. This page provides an overview of how `  NUMERIC  ` is represented in client libraries.

## PostgreSQL NUMERIC

The PostgreSQL `  NUMERIC  ` type is an arbitrary decimal precision numeric data type with a maximum precision (total digits) of 147,455 and a maximum scale (digits to the right of the decimal point) of 16,383.

Spanner DDL does not support specifying precision and scale for PostgreSQL `  NUMERIC  ` columns. However, numeric values can be cast to fixed precision values in DML statements. For example:

``` text
update t1 set numeric_column = (numeric_column*0.8)::numeric(5,2);
```

The type `  DECIMAL  ` is an alias for `  NUMERIC  ` .

PostgreSQL `  NUMERIC  ` columns cannot be used when specifying primary keys, foreign keys, or secondary indexes.

## Represent NUMERIC in each client library language

To maintain the fidelity of `  NUMERIC  ` values, each Spanner [client library](/spanner/docs/reference/libraries) stores those values in an appropriate data type in the client library language. The following table lists the data types to which `  NUMERIC  ` is mapped in each supported language.

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Language</th>
<th style="text-align: left;">GoogleSQL</th>
<th>PostgreSQL</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">C++</td>
<td style="text-align: left;"><a href="https://googleapis.dev/cpp/google-cloud-spanner/1.32.0/classgoogle_1_1cloud_1_1spanner_1_1v1_1_1Numeric.html">spanner::Numeric</a></td>
<td></td>
</tr>
<tr class="even">
<td style="text-align: left;">C#</td>
<td style="text-align: left;"><a href="https://googleapis.github.io/google-cloud-dotnet/docs/Google.Cloud.Spanner.Data/api/Google.Cloud.Spanner.V1.SpannerNumeric.html">SpannerNumeric</a></td>
<td></td>
</tr>
<tr class="odd">
<td style="text-align: left;">Go</td>
<td style="text-align: left;"><a href="https://golang.org/pkg/math/big/#Rat">big.Rat</a></td>
<td>Custom PGNumeric</td>
</tr>
<tr class="even">
<td style="text-align: left;">Java</td>
<td style="text-align: left;"><a href="https://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html">BigDecimal</a></td>
<td>Custom type. See <a href="#pg-java-notes">PostgreSQL Java library notes</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Node.js</td>
<td style="text-align: left;"><a href="https://github.com/MikeMcl/big.js">Big</a></td>
<td></td>
</tr>
<tr class="even">
<td style="text-align: left;">PHP</td>
<td style="text-align: left;">custom <a href="https://googleapis.github.io/google-cloud-php/#/docs/google-cloud/v0.140.0/spanner/spannerclient?method=numeric">Numeric</a></td>
<td></td>
</tr>
<tr class="odd">
<td style="text-align: left;">Python</td>
<td style="text-align: left;"><a href="https://docs.python.org/3/library/decimal.html">Decimal</a></td>
<td><a href="https://docs.python.org/3/library/decimal.html">Decimal</a> with custom annotation</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ruby</td>
<td style="text-align: left;"><a href="https://ruby-doc.org/stdlib-2.5.1/libdoc/bigdecimal/rdoc/BigDecimal.html">BigDecimal</a></td>
<td></td>
</tr>
</tbody>
</table>

Three client libraries, C++, C\# and PHP have each implemented a custom type to represent Spanner SQL's `  NUMERIC  ` type. All other libraries use an existing type.

The C++ client library `  spanner::Numeric  ` object does not support arithmetic operations. Instead, convert the contained number to the C++ object of choice. For example, you can extract the number as a string, which would represent the number at full fidelity and with no data loss. If, however, you know in advance that number fits, for example, within the range of `  std:int64_t  ` or `  double  ` , then you can access the value as that type.

### PostgreSQL Java library notes

The Spanner Java client library uses a custom `  Value.pgNumeric  ` type to store PostgreSQL NUMERIC values.

#### Write to a NUMERIC column

Multiple types are supported when writing to a NUMERIC column in a PostgreSQL table.

  - Numerics
    
    ``` text
    INSERT INTO Table (id, PgNumericColumn) VALUES (1, 1.23)
    ```

  - Integers
    
    ``` text
    INSERT INTO Table (id, PgNumericColumn) VALUES (1, 1)
    ```

  - Doubles
    
    ``` text
    INSERT INTO Table (id, PgNumericColumn) VALUES (1, 1.23::float8)
    ```

  - Untyped literals
    
    ``` text
    INSERT INTO Table (id, PgNumericColumn) VALUES (1, 'NaN')
    ```

#### Parameterized queries

When using parameterized queries, specify the parameters with `  $<index>  ` , where `  <index>  ` denotes the parameter position. The parameter should then be bound using `  p<index>  ` . For example, `  INSERT INTO MyTable (PgNumericColumn) VALUES ($1)  ` with the parameter being `  p1  ` .

The Java client library supports the following types as parameterized values:

  - Custom `  Value.pgNumeric  `
    
    ``` text
    Statement
      .newBuilder("INSERT INTO MyTable (PgNumericColumn) VALUES ($1), ($2)")
      .bind("p1")
      .to(Value.pgNumeric("1.23"))
      .bind("p2")
      .to(Value.pgNumeric("NaN"))
      .build()
    ```

  - Doubles
    
    ``` text
    Statement
      .newBuilder("INSERT INTO MyTable (PgNumericColumn) VALUES ($1), ($2)")
      .bind("p1")
      .to(1.23D)
      .bind("p2")
      .to(Double.NaN)
      .build()
    ```

  - Integers
    
    ``` text
      Statement
        .newBuilder("INSERT INTO MyTable (PgNumericColumn) VALUES ($1)")
        .bind("p1")
        .to(1)
        .build()
    ```

  - Longs
    
    ``` text
      Statement
        .newBuilder("INSERT INTO MyTable (PgNumericColumn) VALUES ($1)")
        .bind("p1")
        .to(1L)
        .build()
    ```

#### Mutations

When using Mutations, the following values are allowed to be written to columns of numeric type:

  - Strings
    
    ``` text
    Mutation
      .newInsertBuilder("MyTable")
      .set("PgNumericColumn")
      .to("1.23")
      .build()
    ```

  - Values of BigDecimal types
    
    ### BigDecimals
    
    ``` text
    Mutation
      .newInsertBuilder("MyTable")
      .set("PgNumericColumn")
      .to(new BigDecimal("1.23"))
      .build()
    ```
    
    ### Ints
    
    ``` text
    Mutation
      .newInsertBuilder("MyTable")
      .set("PgNumericColumn")
      .to(1)
      .build()
    ```
    
    ### Longs
    
    ``` text
    Mutation
      .newInsertBuilder("MyTable")
      .set("PgNumericColumn")
      .to(1L)
      .build()
    ```

  - Values obtained as a result of a call to Value.pgNumeric
    
    ``` text
    Mutation
      .newInsertBuilder("MyTable")
      .set("PgNumericColumn")
      .to(Value.pgNumeric("1.23"))
      .build()
    ```

#### Retrieve from a NUMERIC column

To obtain values stored in numeric columns of a ResultSet, use `  ResultSet.getString()  ` or `  ResultSet.getValue()  ` .

  - Strings
    
    ``` text
    resultSet.getString("PgNumericColumn")
    ```

  - Custom Value
    
    ``` text
    Value pgNumeric = resultSet.getValue("PgNumericColumn");
    pgNumeric.getString(); // get underlying value as a String
    pgNumeric.getNumeric(); // get underlying value as a BigDecimal
    pgNumeric.getFloat64(); // get underlying value as aDouble
    ```

## Add a NUMERIC column

The following sample shows how to add a `  NUMERIC  ` column to a table called `  Venues  ` using the Spanner client libraries.

### C++

``` cpp
void AddNumericColumn(google::cloud::spanner_admin::DatabaseAdminClient client,
                      std::string const& project_id,
                      std::string const& instance_id,
                      std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  auto metadata = client
                      .UpdateDatabaseDdl(database.FullName(), {R"""(
                        ALTER TABLE Venues ADD COLUMN Revenue NUMERIC)"""})
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

public class AddNumericColumnAsyncSample
{
    public async Task AddNumericColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string alterStatement = "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC";

        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(alterStatement);
        await updateCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added the Revenue column.");
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func addNumericColumn(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added Revenue column\n")
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

class AddNumericColumnSample {

  static void addNumericColumn() throws InterruptedException, ExecutionException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    addNumericColumn(projectId, instanceId, databaseId);
  }

  static void addNumericColumn(String projectId, String instanceId, String databaseId)
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
          ImmutableList.of("ALTER TABLE Venues ADD COLUMN Revenue NUMERIC")).get();
      System.out.printf("Successfully added column `Revenue`%n");
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

const request = ['ALTER TABLE Venues ADD COLUMN Revenue NUMERIC'];

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
  `Added Revenue column to Venues table in database ${databaseId}.`,
);
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds a NUMERIC column to a table.
 * Example:
 * ```
 * add_numeric_column($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function add_numeric_column(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => ['ALTER TABLE Venues ADD COLUMN Revenue NUMERIC']
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Added Revenue as a NUMERIC column in Venues table' . PHP_EOL);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def add_numeric_column(instance_id, database_id):
    """Adds a new NUMERIC column to the Venues table in the example database."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=["ALTER TABLE Venues ADD COLUMN Revenue NUMERIC"],
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

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

db_path = database_admin_client.database_path project: project_id,
                                              instance: instance_id,
                                              database: database_id

job = database_admin_client.update_database_ddl database: db_path,
                                                statements: [
                                                  "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC"
                                                ]

puts "Waiting for database update to complete"

job.wait_until_done!

puts "Added the Revenue as a numeric column in Venues table"
```

## Update NUMERIC data

The following sample shows how to update `  NUMERIC  ` data using the Spanner client libraries.

### C++

``` cpp
void UpdateDataWithNumeric(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto insert_venues =
      spanner::InsertMutationBuilder(
          "Venues", {"VenueId", "VenueName", "Revenue", "LastUpdateTime"})
          .EmplaceRow(1, "Venue 1", spanner::MakeNumeric(35000).value(),
                      spanner::CommitTimestamp())
          .EmplaceRow(6, "Venue 6", spanner::MakeNumeric(104500).value(),
                      spanner::CommitTimestamp())
          .EmplaceRow(
              14, "Venue 14",
              spanner::MakeNumeric("99999999999999999999999999999.99").value(),
              spanner::CommitTimestamp())
          .Build();

  auto commit_result = client.Commit(spanner::Mutations{insert_venues});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout
      << "Insert was successful [spanner_update_data_with_numeric_column]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using Google.Cloud.Spanner.V1;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class UpdateDataWithNumericAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public SpannerNumeric Revenue { get; set; }
    }

    public async Task<int> UpdateDataWithNumericAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        List<Venue> venues = new List<Venue>
        {
            new Venue { VenueId = 4, Revenue = SpannerNumeric.Parse("35000") },
            new Venue { VenueId = 19, Revenue = SpannerNumeric.Parse("104500") },
            new Venue { VenueId = 42, Revenue = SpannerNumeric.Parse("99999999999999999999999999999.99") },
        };
        // Create connection to Cloud Spanner.
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        var affectedRows = await Task.WhenAll(venues.Select(venue =>
        {
            // Update rows in the Venues table.
            using var cmd = connection.CreateUpdateCommand("Venues", new SpannerParameterCollection
            {
                { "VenueId", SpannerDbType.Int64, venue.VenueId },
                { "Revenue", SpannerDbType.Numeric, venue.Revenue }
            });
            return cmd.ExecuteNonQueryAsync();
        }));

        Console.WriteLine("Data updated.");
        return affectedRows.Sum();
    }
}
```

### Go

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func updateDataWithNumericColumn(w io.Writer, db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 cols := []string{"VenueId", "Revenue"}
 _, err = client.Apply(ctx, []*spanner.Mutation{
     spanner.Update("Venues", cols, []interface{}{4, "35000"}),
     spanner.Update("Venues", cols, []interface{}{19, "104500"}),
     spanner.Update("Venues", cols, []interface{}{42, "99999999999999999999999999999.99"}),
 })
 return err
}
```

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.Mutation;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.common.collect.ImmutableList;
import java.math.BigDecimal;

class UpdateNumericDataSample {

  static void updateNumericData() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      DatabaseClient client =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      updateNumericData(client);
    }
  }

  static void updateNumericData(DatabaseClient client) {
    client.write(
        ImmutableList.of(
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(4L)
                .set("Revenue")
                .to(new BigDecimal("35000"))
                .build(),
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(19L)
                .set("Revenue")
                .to(new BigDecimal("104500"))
                .build(),
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(42L)
                .set("Revenue")
                .to(new BigDecimal("99999999999999999999999999999.99"))
                .build()));
    System.out.println("Venues successfully updated");
  }
}
```

### Node.js

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
    VenueId: '4',
    Revenue: Spanner.numeric('35000'),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '19',
    Revenue: Spanner.numeric('104500'),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '42',
    Revenue: Spanner.numeric('99999999999999999999999999999.99'),
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

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Updates sample data in a table with a NUMERIC column.
 *
 * Before executing this method, a new column Revenue has to be added to the Venues
 * table by applying the DDL statement "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC".
 *
 * Example:
 * ```
 * update_data_with_numeric_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_numeric_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->transaction(['singleUse' => true])
        ->updateBatch('Venues', [
            ['VenueId' => 4, 'Revenue' => $spanner->numeric('35000')],
            ['VenueId' => 19, 'Revenue' => $spanner->numeric('104500')],
            ['VenueId' => 42, 'Revenue' => $spanner->numeric('99999999999999999999999999999.99')],
        ])
        ->commit();

    print('Updated data.' . PHP_EOL);
}
````

### Python

``` python
def update_data_with_numeric(instance_id, database_id):
    """Updates Venues tables in the database with the NUMERIC
    column.

    This updates the `Revenue` column which must be created before
    running this sample. You can add the column by running the
    `add_numeric_column` sample or by running this DDL statement
     against your database:

        ALTER TABLE Venues ADD COLUMN Revenue NUMERIC
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.batch() as batch:
        batch.update(
            table="Venues",
            columns=("VenueId", "Revenue"),
            values=[
                (4, decimal.Decimal("35000")),
                (19, decimal.Decimal("104500")),
                (42, decimal.Decimal("99999999999999999999999999999.99")),
            ],
        )

    print("Updated data.")
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.commit do |c|
  c.update "Venues", [
    { VenueId: 4, Revenue: "35000" },
    { VenueId: 19, Revenue: "104500" },
    { VenueId: 42, Revenue: "99999999999999999999999999999.99" }
  ]
end

puts "Updated data"
```

## Query NUMERIC data

The following sample shows how to query `  NUMERIC  ` data using the Spanner client libraries.

### C++

``` cpp
void QueryWithNumericParameter(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  auto revenue = spanner::MakeNumeric(100000).value();
  spanner::SqlStatement select(
      "SELECT VenueId, Revenue"
      "  FROM Venues"
      " WHERE Revenue < @revenue",
      {{"revenue", spanner::Value(std::move(revenue))}});
  using RowType = std::tuple<std::int64_t, absl::optional<spanner::Numeric>>;

  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "VenueId: " << std::get<0>(*row) << "\t";
    auto revenue = std::get<1>(*row).value();
    std::cout << "Revenue: " << revenue.ToString()
              << " (d.16=" << std::setprecision(16)
              << spanner::ToDouble(revenue)
              << ", i*10^2=" << spanner::ToInteger<int>(revenue, 2).value()
              << ")\n";
  }
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using Google.Cloud.Spanner.V1;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithNumericParameterAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public SpannerNumeric Revenue { get; set; }
    }

    public async Task<List<Venue>> QueryDataWithNumericParameterAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            "SELECT VenueId, Revenue FROM Venues WHERE Revenue < @maxRevenue",
            new SpannerParameterCollection
            {
                { "maxRevenue", SpannerDbType.Numeric, SpannerNumeric.Parse("100000") }
            });

        var venues = new List<Venue>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            venues.Add(new Venue
            {
                VenueId = reader.GetFieldValue<int>("VenueId"),
                Revenue = reader.GetFieldValue<SpannerNumeric>("Revenue")
            });
        }
        return venues;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "math/big"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func queryWithNumericParameter(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{
     SQL: `SELECT VenueId, Revenue FROM Venues WHERE Revenue < @revenue`,
     Params: map[string]interface{}{
         "revenue": big.NewRat(100000, 1),
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
     var revenue big.Rat
     if err := row.Columns(&venueID, &revenue); err != nil {
         return err
     }
     fmt.Fprintf(w, "%v %v\n", venueID, revenue)
 }
}
```

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import java.math.BigDecimal;

class QueryWithNumericParameterSample {

  static void queryWithNumericParameter() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      DatabaseClient client =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      queryWithNumericParameter(client);
    }
  }

  static void queryWithNumericParameter(DatabaseClient client) {
    Statement statement =
        Statement.newBuilder(
                "SELECT VenueId, Revenue FROM Venues WHERE Revenue < @numeric")
            .bind("numeric")
            .to(new BigDecimal("100000"))
            .build();
    try (ResultSet resultSet = client.singleUse().executeQuery(statement)) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %s%n", resultSet.getLong("VenueId"), resultSet.getBigDecimal("Revenue"));
      }
    }
  }
}
```

### Node.js

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
  type: 'numeric',
};

const exampleNumeric = Spanner.numeric('100000');

const query = {
  sql: `SELECT VenueId, VenueName, Revenue FROM Venues
          WHERE Revenue < @revenue`,
  params: {
    revenue: exampleNumeric,
  },
  types: {
    revenue: fieldType,
  },
};

// Queries rows from the Venues table.
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(`VenueId: ${json.VenueId}, Revenue: ${json.Revenue.value}`);
  });
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
use Google\Cloud\Spanner\Database;

/**
 * Queries sample data from the database using SQL with a NUMERIC parameter.
 * Example:
 * ```
 * query_data_with_numeric_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_numeric_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $exampleNumeric = $spanner->numeric('100000');

    $results = $database->execute(
        'SELECT VenueId, Revenue FROM Venues ' .
        'WHERE Revenue < @revenue',
        [
            'parameters' => [
                'revenue' => $exampleNumeric
            ]
        ]
    );

    foreach ($results as $row) {
        printf('VenueId: %s, Revenue: %s' . PHP_EOL,
            $row['VenueId'], $row['Revenue']);
    }
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

example_numeric = decimal.Decimal("100000")
param = {"revenue": example_numeric}
param_type = {"revenue": param_types.NUMERIC}

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, Revenue FROM Venues " "WHERE Revenue < @revenue",
        params=param,
        param_types=param_type,
    )

    for row in results:
        print("VenueId: {}, Revenue: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "bigdecimal"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

sql_query = "SELECT VenueId, Revenue FROM Venues WHERE Revenue < @revenue"

params      = { revenue: BigDecimal("100000") }
param_types = { revenue: :NUMERIC }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:Revenue]}"
end
```

`  NUMERIC  ` is supported in the Spanner JDBC driver using the Java [BigDecimal](https://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html) type. For examples of how `  NUMERIC  ` is used, see the code samples in [Connect JDBC to a GoogleSQL-dialect database](/spanner/docs/use-oss-jdbc) .

## Handle NUMERIC when creating a client library or driver

The `  NUMERIC  ` type is encoded as a string in decimal or scientific notation within a [google.protobuf.Value](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#value) proto. This proto is wrapped as either a [ResultSet](https://github.com/googleapis/googleapis/blob/master/google/spanner/v1/result_set.proto#L35) , [PartialResultSet](https://github.com/googleapis/googleapis/blob/master/google/spanner/v1/result_set.proto#L61) , or a [Mutation](https://github.com/googleapis/googleapis/blob/master/google/spanner/v1/mutation.proto#L33) depending on whether it is being read or written. ResultSetMetadata will use the [NUMERIC TypeCode](https://github.com/googleapis/googleapis/blob/master/google/spanner/v1/type.proto#L130) to indicate that the corresponding value should be read as a `  NUMERIC  ` .

When working with NUMERIC in a client library or driver you create, observe the following guidance.

  - To read a `  NUMERIC  ` from the ResultSet:
    
    1.  Read the string\_value from the [google.protobuf.Value](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#value) proto when TypeCode is `  NUMERIC  `
    
    2.  Convert that string to the relevant type for the given language

  - To write a `  NUMERIC  ` using Mutations, use the string representation as the string\_value in the [google.protobuf.Value](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#value) proto when given the relevant type.
