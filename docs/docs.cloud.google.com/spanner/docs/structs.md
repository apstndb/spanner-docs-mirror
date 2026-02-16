**PostgreSQL interface note:** The `  STRUCT  ` data type is not supported in the [PostgreSQL interface for Spanner](/spanner/docs/postgresql-interface) .

Spanner allows you to create `  STRUCT  ` objects from data, as well as to use `  STRUCT  ` objects as bound parameters when running a SQL query with one of the Spanner client libraries.

For more information about the `  STRUCT  ` type in Spanner, see [Data types](/spanner/docs/reference/standard-sql/data-types#struct_type) .

## Declaring a user-defined type of STRUCT object

You can declare a `  STRUCT  ` object in queries using the syntax described in [Declaring a `  STRUCT  ` type](/spanner/docs/reference/standard-sql/data-types#constructing_a_struct) .

You can define a type of `  STRUCT  ` object as a sequence of field names and their data types. You can then supply this type along with queries containing `  STRUCT  ` -typed parameter bindings and Spanner will use it to check that the `  STRUCT  ` parameter values in your query are valid.

### C++

``` cpp
// Cloud Spanner STRUCT<> types are represented by std::tuple<...>. The
// following represents a STRUCT<> with two unnamed STRING fields.
using NameType = std::tuple<std::string, std::string>;
```

### C\#

``` csharp
var nameType = new SpannerStruct
{
    { "FirstName", SpannerDbType.String, null},
    { "LastName", SpannerDbType.String, null}
};
```

### Go

``` go
type nameType struct {
 FirstName string
 LastName  string
}
```

### Java

``` java
Type nameType =
    Type.struct(
        Arrays.asList(
            StructField.of("FirstName", Type.string()),
            StructField.of("LastName", Type.string())));
```

### Node.js

``` javascript
const nameType = {
  type: 'struct',
  fields: [
    {
      name: 'FirstName',
      type: 'string',
    },
    {
      name: 'LastName',
      type: 'string',
    },
  ],
};
```

### PHP

``` php
$nameType = new ArrayType(
    (new StructType)
        ->add('FirstName', Database::TYPE_STRING)
        ->add('LastName', Database::TYPE_STRING)
);
```

### Python

``` python
name_type = param_types.Struct(
    [
        param_types.StructField("FirstName", param_types.STRING),
        param_types.StructField("LastName", param_types.STRING),
    ]
)
```

### Ruby

``` ruby
name_type = client.fields FirstName: :STRING, LastName: :STRING
```

## Creating STRUCT objects

The following sample shows how to create `  STRUCT  ` objects using the Spanner client libraries.

### C++

``` cpp
// Cloud Spanner STRUCT<> types are represented by std::tuple<...>. The
// following represents a STRUCT<> with two unnamed STRING fields.
using NameType = std::tuple<std::string, std::string>;
auto singer_info = NameType{"Elena", "Campbell"};
```

### C\#

``` csharp
var nameStruct = new SpannerStruct
{
    { "FirstName", SpannerDbType.String, "Elena" },
    { "LastName", SpannerDbType.String, "Campbell" },
};
```

### Go

``` go
type name struct {
 FirstName string
 LastName  string
}
var singerInfo = name{"Elena", "Campbell"}
```

### Java

``` java
Struct name =
    Struct.newBuilder().set("FirstName").to("Elena").set("LastName").to("Campbell").build();
```

### Node.js

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

const nameStruct = Spanner.struct({
  FirstName: 'Elena',
  LastName: 'Campbell',
});
```

### PHP

``` php
$nameValue = (new StructValue)
    ->add('FirstName', 'Elena')
    ->add('LastName', 'Campbell');
$nameType = (new StructType)
    ->add('FirstName', Database::TYPE_STRING)
    ->add('LastName', Database::TYPE_STRING);
```

### Python

``` python
record_type = param_types.Struct(
    [
        param_types.StructField("FirstName", param_types.STRING),
        param_types.StructField("LastName", param_types.STRING),
    ]
)
record_value = ("Elena", "Campbell")
```

### Ruby

``` ruby
name_struct = { FirstName: "Elena", LastName: "Campbell" }
```

You can also use the client libraries to create an array of `  STRUCT  ` objects, as seen in the following sample:

### C++

``` cpp
// Cloud Spanner STRUCT<> types with named fields are represented by
// std::tuple<std::pair<std::string, T>...>, create an alias to make it easier
// to follow this code.
using SingerName = std::tuple<std::pair<std::string, std::string>,
                              std::pair<std::string, std::string>>;
auto make_name = [](std::string first_name, std::string last_name) {
  return std::make_tuple(std::make_pair("FirstName", std::move(first_name)),
                         std::make_pair("LastName", std::move(last_name)));
};
std::vector<SingerName> singer_info{
    make_name("Elena", "Campbell"),
    make_name("Gabriel", "Wright"),
    make_name("Benjamin", "Martinez"),
};
```

### C\#

``` csharp
var bandMembers = new List<SpannerStruct>
{
    new SpannerStruct { { "FirstName", SpannerDbType.String, "Elena" }, { "LastName", SpannerDbType.String, "Campbell" } },
    new SpannerStruct { { "FirstName", SpannerDbType.String, "Gabriel" }, { "LastName", SpannerDbType.String, "Wright" } },
    new SpannerStruct { { "FirstName", SpannerDbType.String, "Benjamin" }, { "LastName", SpannerDbType.String, "Martinez" } },
};
```

### Go

``` go
var bandMembers = []nameType{
 {"Elena", "Campbell"},
 {"Gabriel", "Wright"},
 {"Benjamin", "Martinez"},
}
```

### Java

``` java
List<Struct> bandMembers = new ArrayList<>();
bandMembers.add(
    Struct.newBuilder().set("FirstName").to("Elena").set("LastName").to("Campbell").build());
bandMembers.add(
    Struct.newBuilder().set("FirstName").to("Gabriel").set("LastName").to("Wright").build());
bandMembers.add(
    Struct.newBuilder().set("FirstName").to("Benjamin").set("LastName").to("Martinez").build());
```

### Node.js

``` javascript
const bandMembersType = {
  type: 'array',
  child: nameType,
};

const bandMembers = [
  Spanner.struct({
    FirstName: 'Elena',
    LastName: 'Campbell',
  }),
  Spanner.struct({
    FirstName: 'Gabriel',
    LastName: 'Wright',
  }),
  Spanner.struct({
    FirstName: 'Benjamin',
    LastName: 'Martinez',
  }),
];
```

### PHP

``` php
$bandMembers = [
    (new StructValue)
        ->add('FirstName', 'Elena')
        ->add('LastName', 'Campbell'),
    (new StructValue)
        ->add('FirstName', 'Gabriel')
        ->add('LastName', 'Wright'),
    (new StructValue)
        ->add('FirstName', 'Benjamin')
        ->add('LastName', 'Martinez')
];
```

### Python

``` python
band_members = [
    ("Elena", "Campbell"),
    ("Gabriel", "Wright"),
    ("Benjamin", "Martinez"),
]
```

### Ruby

``` ruby
band_members = [name_type.struct(["Elena", "Campbell"]),
                name_type.struct(["Gabriel", "Wright"]),
                name_type.struct(["Benjamin", "Martinez"])]
```

## Returning STRUCT objects in SQL query results

A Spanner SQL query can return an array of `  STRUCT  ` objects as a column for certain queries. For more information, see [Using STRUCTS with SELECT](/spanner/docs/reference/standard-sql/query-syntax#using_structs_with_select) .

## Using STRUCT objects as bound parameters in SQL queries

You can use `  STRUCT  ` objects as bound parameters in a SQL query. For more information about parameters, see [Query parameters](/spanner/docs/reference/standard-sql/lexical#query_parameters) .

### Querying data with a STRUCT object

The following sample shows how to bind values in a `  STRUCT  ` object to parameters in a SQL query statement, execute the query, and output the results.

### C++

``` cpp
void QueryDataWithStruct(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  // Cloud Spanner STRUCT<> types are represented by std::tuple<...>. The
  // following represents a STRUCT<> with two unnamed STRING fields.
  using NameType = std::tuple<std::string, std::string>;
  auto singer_info = NameType{"Elena", "Campbell"};
  auto rows = client.ExecuteQuery(spanner::SqlStatement(
      "SELECT SingerId FROM Singers WHERE (FirstName, LastName) = @name",
      {{"name", spanner::Value(singer_info)}}));

  for (auto& row : spanner::StreamOf<std::tuple<std::int64_t>>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\n";
  }
  std::cout << "Query completed for [spanner_query_data_with_struct]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithStructAsyncSample
{
    public async Task<List<int>> QueryDataWithStructAsync(string projectId, string instanceId, string databaseId)
    {
        var nameStruct = new SpannerStruct
        {
            { "FirstName", SpannerDbType.String, "Elena" },
            { "LastName", SpannerDbType.String, "Campbell" },
        };

        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var singerIds = new List<int>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            "SELECT SingerId FROM Singers "
            + "WHERE STRUCT<FirstName STRING, LastName STRING>"
            + "(FirstName, LastName) = @name");

        cmd.Parameters.Add("name", nameStruct.GetSpannerDbType(), nameStruct);
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            singerIds.Add(reader.GetFieldValue<int>("SingerId"));
        }
        return singerIds;
    }
}
```

### Go

``` go
stmt := spanner.Statement{
 SQL: `SELECT SingerId FROM SINGERS
         WHERE (FirstName, LastName) = @singerinfo`,
 Params: map[string]interface{}{"singerinfo": singerInfo},
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
 var singerID int64
 if err := row.Columns(&singerID); err != nil {
     return err
 }
 fmt.Fprintf(w, "%d\n", singerID)
}
```

### Java

``` java
Statement s =
    Statement.newBuilder(
            "SELECT SingerId FROM Singers "
                + "WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) "
                + "= @name")
        .bind("name")
        .to(name)
        .build();
try (ResultSet resultSet = dbClient.singleUse().executeQuery(s)) {
  while (resultSet.next()) {
    System.out.printf("%d\n", resultSet.getLong("SingerId"));
  }
}
```

### Node.js

``` javascript
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

const query = {
  sql:
    'SELECT SingerId FROM Singers WHERE ' +
    'STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) = @name',
  params: {
    name: nameStruct,
  },
};

// Queries rows from the Singers table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(`SingerId: ${json.SingerId}`);
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

``` php
$results = $database->execute(
    'SELECT SingerId FROM Singers ' .
    'WHERE STRUCT<FirstName STRING, LastName STRING>' .
    '(FirstName, LastName) = @name',
    [
        'parameters' => [
            'name' => $nameValue
        ],
        'types' => [
            'name' => $nameType
        ]
    ]
);
foreach ($results as $row) {
    printf('SingerId: %s' . PHP_EOL,
        $row['SingerId']);
}
```

### Python

``` python
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)

database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT SingerId FROM Singers WHERE " "(FirstName, LastName) = @name",
        params={"name": record_value},
        param_types={"name": record_type},
    )

for row in results:
    print("SingerId: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id
client.execute(
  "SELECT SingerId FROM Singers WHERE " +
  "(FirstName, LastName) = @name",
  params: { name: name_struct }
).rows.each do |row|
  puts row[:SingerId]
end
```

### Querying data with an array of STRUCT objects

The following sample shows how to execute a query that uses an array of `  STRUCT  ` objects. Use the [UNNEST](/spanner/docs/query-execution-operators#array-unnest) operator to flatten an array of `  STRUCT  ` objects into rows:

### C++

``` cpp
void QueryDataWithArrayOfStruct(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  // Cloud Spanner STRUCT<> types with named fields are represented by
  // std::tuple<std::pair<std::string, T>...>, create an alias to make it easier
  // to follow this code.
  using SingerName = std::tuple<std::pair<std::string, std::string>,
                                std::pair<std::string, std::string>>;
  auto make_name = [](std::string first_name, std::string last_name) {
    return std::make_tuple(std::make_pair("FirstName", std::move(first_name)),
                           std::make_pair("LastName", std::move(last_name)));
  };
  std::vector<SingerName> singer_info{
      make_name("Elena", "Campbell"),
      make_name("Gabriel", "Wright"),
      make_name("Benjamin", "Martinez"),
  };

  auto rows = client.ExecuteQuery(spanner::SqlStatement(
      "SELECT SingerId FROM Singers"
      " WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName)"
      "    IN UNNEST(@names)",
      {{"names", spanner::Value(singer_info)}}));

  for (auto& row : spanner::StreamOf<std::tuple<std::int64_t>>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\n";
  }
  std::cout << "Query completed for"
            << " [spanner_query_data_with_array_of_struct]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithArrayOfStructAsyncSample
{
    public async Task<List<int>> QueryDataWithArrayOfStructAsync(string projectId, string instanceId, string databaseId)
    {
        var nameType = new SpannerStruct
        {
            { "FirstName", SpannerDbType.String, null},
            { "LastName", SpannerDbType.String, null}
        };

        var bandMembers = new List<SpannerStruct>
        {
            new SpannerStruct { { "FirstName", SpannerDbType.String, "Elena" }, { "LastName", SpannerDbType.String, "Campbell" } },
            new SpannerStruct { { "FirstName", SpannerDbType.String, "Gabriel" }, { "LastName", SpannerDbType.String, "Wright" } },
            new SpannerStruct { { "FirstName", SpannerDbType.String, "Benjamin" }, { "LastName", SpannerDbType.String, "Martinez" } },
        };

        var singerIds = new List<int>();
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            "SELECT SingerId FROM Singers WHERE STRUCT<FirstName STRING, LastName STRING> "
            + "(FirstName, LastName) IN UNNEST(@names)");
        cmd.Parameters.Add("names", SpannerDbType.ArrayOf(nameType.GetSpannerDbType()), bandMembers);
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            singerIds.Add(reader.GetFieldValue<int>("SingerId"));
        }
        return singerIds;
    }
}
```

### Go

``` go
stmt := spanner.Statement{
 SQL: `SELECT SingerId FROM SINGERS
     WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName)
     IN UNNEST(@names)`,
 Params: map[string]interface{}{"names": bandMembers},
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
 var singerID int64
 if err := row.Columns(&singerID); err != nil {
     return err
 }
 fmt.Fprintf(w, "%d\n", singerID)
}
```

### Java

``` java
Statement s =
    Statement.newBuilder(
            "SELECT SingerId FROM Singers WHERE "
                + "STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) "
                + "IN UNNEST(@names) "
                + "ORDER BY SingerId DESC")
        .bind("names")
        .toStructArray(nameType, bandMembers)
        .build();
try (ResultSet resultSet = dbClient.singleUse().executeQuery(s)) {
  while (resultSet.next()) {
    System.out.printf("%d\n", resultSet.getLong("SingerId"));
  }
}
```

### Node.js

``` javascript
const query = {
  sql:
    'SELECT SingerId FROM Singers ' +
    'WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) ' +
    'IN UNNEST(@names) ' +
    'ORDER BY SingerId',
  params: {
    names: bandMembers,
  },
  types: {
    names: bandMembersType,
  },
};

// Queries rows from the Singers table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(`SingerId: ${json.SingerId}`);
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

``` php
$results = $database->execute(
    'SELECT SingerId FROM Singers ' .
    'WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) ' .
    'IN UNNEST(@names)',
    [
        'parameters' => [
            'names' => $bandMembers
        ],
        'types' => [
            'names' => $nameType
        ]
    ]
);
foreach ($results as $row) {
    printf('SingerId: %s' . PHP_EOL,
        $row['SingerId']);
}
```

### Python

``` python
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT SingerId FROM Singers WHERE "
        "STRUCT<FirstName STRING, LastName STRING>"
        "(FirstName, LastName) IN UNNEST(@names)",
        params={"names": band_members},
        param_types={"names": param_types.Array(name_type)},
    )

for row in results:
    print("SingerId: {}".format(*row))
```

### Ruby

``` ruby
client.execute(
  "SELECT SingerId FROM Singers WHERE " +
  "STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) IN UNNEST(@names)",
  params: { names: band_members }
).rows.each do |row|
  puts row[:SingerId]
end
```

### Modifying data with DML

The following code example uses a `  STRUCT  ` with bound parameters and Data Manipulation Language (DML) to update a single value in rows that match the WHERE clause condition. For rows where the `  FirstName  ` is `  Timothy  ` and the `  LastName  ` is `  Campbell  ` , the `  LastName  ` is updated to `  Grant  ` .

### C++

``` cpp
void DmlStructs(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  std::int64_t rows_modified = 0;
  auto commit_result =
      client.Commit([&client, &rows_modified](spanner::Transaction const& txn)
                        -> google::cloud::StatusOr<spanner::Mutations> {
        auto singer_info = std::make_tuple("Marc", "Richards");
        auto sql = spanner::SqlStatement(
            "UPDATE Singers SET FirstName = 'Keith' WHERE "
            "STRUCT<FirstName String, LastName String>(FirstName, LastName) "
            "= @name",
            {{"name", spanner::Value(std::move(singer_info))}});
        auto dml_result = client.ExecuteDml(txn, std::move(sql));
        if (!dml_result) return std::move(dml_result).status();
        rows_modified = dml_result->RowsModified();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << rows_modified
            << " update was successful [spanner_dml_structs]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateUsingDmlWithStructCoreAsyncSample
{
    public async Task<int> UpdateUsingDmlWithStructCoreAsync(string projectId, string instanceId, string databaseId)
    {
        var nameStruct = new SpannerStruct
        {
            { "FirstName", SpannerDbType.String, "Timothy" },
            { "LastName", SpannerDbType.String, "Campbell" }
        };
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("UPDATE Singers SET LastName = 'Grant' WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) = @name");
        cmd.Parameters.Add("name", nameStruct.GetSpannerDbType(), nameStruct);
        int rowCount = await cmd.ExecuteNonQueryAsync();

        Console.WriteLine($"{rowCount} row(s) updated...");
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

func updateUsingDMLStruct(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     type name struct {
         FirstName string
         LastName  string
     }
     var singerInfo = name{"Timothy", "Campbell"}

     stmt := spanner.Statement{
         SQL: `Update Singers Set LastName = 'Grant'
             WHERE STRUCT<FirstName String, LastName String>(Firstname, LastName) = @name`,
         Params: map[string]interface{}{"name": singerInfo},
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

``` java
static void updateUsingDmlWithStruct(DatabaseClient dbClient) {
  Struct name =
      Struct.newBuilder().set("FirstName").to("Timothy").set("LastName").to("Campbell").build();
  Statement s =
      Statement.newBuilder(
              "UPDATE Singers SET LastName = 'Grant' "
                  + "WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) "
                  + "= @name")
          .bind("name")
          .to(name)
          .build();
  dbClient
      .readWriteTransaction()
      .run(transaction -> {
        long rowCount = transaction.executeUpdate(s);
        System.out.printf("%d record updated.\n", rowCount);
        return null;
      });
}
```

### Node.js

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

const nameStruct = Spanner.struct({
  FirstName: 'Timothy',
  LastName: 'Campbell',
});

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

database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  try {
    const [rowCount] = await transaction.runUpdate({
      sql: `UPDATE Singers SET LastName = 'Grant'
      WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) = @name`,
      params: {
        name: nameStruct,
      },
    });

    console.log(`Successfully updated ${rowCount} record.`);
    await transaction.commit();
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
});
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Database;
use Google\Cloud\Spanner\Transaction;
use Google\Cloud\Spanner\StructType;
use Google\Cloud\Spanner\StructValue;

/**
 * Update data with a DML statement using Structs.
 *
 * The database and table must already exist and can be created using
 * `create_database`.
 * Example:
 * ```
 * insert_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_dml_structs(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) {
        $nameValue = (new StructValue)
            ->add('FirstName', 'Timothy')
            ->add('LastName', 'Campbell');
        $nameType = (new StructType)
            ->add('FirstName', Database::TYPE_STRING)
            ->add('LastName', Database::TYPE_STRING);

        $rowCount = $t->executeUpdate(
            "UPDATE Singers SET LastName = 'Grant' "
             . 'WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) '
             . '= @name',
            [
                'parameters' => [
                    'name' => $nameValue
                ],
                'types' => [
                    'name' => $nameType
                ]
            ]);
        $t->commit();
        printf('Updated %d row(s).' . PHP_EOL, $rowCount);
    });
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

record_type = param_types.Struct(
    [
        param_types.StructField("FirstName", param_types.STRING),
        param_types.StructField("LastName", param_types.STRING),
    ]
)
record_value = ("Timothy", "Campbell")

def write_with_struct(transaction):
    row_ct = transaction.execute_update(
        "UPDATE Singers SET LastName = 'Grant' "
        "WHERE STRUCT<FirstName STRING, LastName STRING>"
        "(FirstName, LastName) = @name",
        params={"name": record_value},
        param_types={"name": record_type},
    )
    print("{} record(s) updated.".format(row_ct))

database.run_in_transaction(write_with_struct)
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id
row_count = 0
name_struct = { FirstName: "Timothy", LastName: "Campbell" }

client.transaction do |transaction|
  row_count = transaction.execute_update(
    "UPDATE Singers SET LastName = 'Grant'
     WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) = @name",
    params: { name: name_struct }
  )
end

puts "#{row_count} record updated."
```

## Accessing STRUCT field values

You can access fields inside a `  STRUCT  ` object by name.

### C++

``` cpp
void FieldAccessOnStructParameters(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  // Cloud Spanner STRUCT<> with named fields is represented as
  // tuple<pair<string, T>...>. Create a type alias for this example:
  using SingerName = std::tuple<std::pair<std::string, std::string>,
                                std::pair<std::string, std::string>>;
  SingerName name({"FirstName", "Elena"}, {"LastName", "Campbell"});

  auto rows = client.ExecuteQuery(spanner::SqlStatement(
      "SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName",
      {{"name", spanner::Value(name)}}));

  for (auto& row : spanner::StreamOf<std::tuple<std::int64_t>>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\n";
  }
  std::cout << "Query completed for"
            << " [spanner_field_access_on_struct_parameters]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithStructFieldAsyncSample
{
    public async Task<List<int>> QueryDataWithStructFieldAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var structParam = new SpannerStruct
        {
            { "FirstName", SpannerDbType.String, "Elena" },
            { "LastName", SpannerDbType.String, "Campbell" },
        };

        var singerIds = new List<int>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName");

        cmd.Parameters.Add("name", structParam.GetSpannerDbType(), structParam);
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            singerIds.Add(reader.GetFieldValue<int>("SingerId"));
        }
        return singerIds;
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

func queryWithStructField(w io.Writer, db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 type structParam struct {
     FirstName string
     LastName  string
 }
 var singerInfo = structParam{"Elena", "Campbell"}
 stmt := spanner.Statement{
     SQL: `SELECT SingerId FROM SINGERS
         WHERE FirstName = @name.FirstName`,
     Params: map[string]interface{}{"name": singerInfo},
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
     var singerID int64
     if err := row.Columns(&singerID); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d\n", singerID)
 }
}
```

### Java

``` java
static void queryStructField(DatabaseClient dbClient) {
  Statement s =
      Statement.newBuilder("SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName")
          .bind("name")
          .to(
              Struct.newBuilder()
                  .set("FirstName")
                  .to("Elena")
                  .set("LastName")
                  .to("Campbell")
                  .build())
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(s)) {
    while (resultSet.next()) {
      System.out.printf("%d\n", resultSet.getLong("SingerId"));
    }
  }
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

const nameStruct = Spanner.struct({
  FirstName: 'Elena',
  LastName: 'Campbell',
});
const query = {
  sql: 'SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName',
  params: {
    name: nameStruct,
  },
};

// Queries rows from the Singers table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(`SingerId: ${json.SingerId}`);
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
use Google\Cloud\Spanner\StructType;

/**
 * Queries sample data from the database using a struct field value.
 * Example:
 * ```
 * query_data_with_struct_field($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_struct_field(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $nameType = (new StructType)
        ->add('FirstName', Database::TYPE_STRING)
        ->add('LastName', Database::TYPE_STRING);
    $results = $database->execute(
        'SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName',
        [
            'parameters' => [
                'name' => [
                    'FirstName' => 'Elena',
                    'LastName' => 'Campbell'
                ]
            ],
            'types' => [
                'name' => $nameType
            ]
        ]
    );
    foreach ($results as $row) {
        printf('SingerId: %s' . PHP_EOL,
            $row['SingerId']);
    }
}
````

### Python

``` python
def query_struct_field(instance_id, database_id):
    """Query a table using field access on a STRUCT parameter."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    name_type = param_types.Struct(
        [
            param_types.StructField("FirstName", param_types.STRING),
            param_types.StructField("LastName", param_types.STRING),
        ]
    )

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId FROM Singers " "WHERE FirstName = @name.FirstName",
            params={"name": ("Elena", "Campbell")},
            param_types={"name": name_type},
        )

    for row in results:
        print("SingerId: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

name_struct = { FirstName: "Elena", LastName: "Campbell" }
client.execute(
  "SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName",
  params: { name: name_struct }
).rows.each do |row|
  puts row[:SingerId]
end
```

You can even have fields of `  STRUCT  ` or `  ARRAY<STRUCT>  ` type inside `  STRUCT  ` values and access them similarly:

### C++

``` cpp
void FieldAccessOnNestedStruct(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  // Cloud Spanner STRUCT<> with named fields is represented as
  // tuple<pair<string, T>...>. Create a type alias for this example:
  using SingerFullName = std::tuple<std::pair<std::string, std::string>,
                                    std::pair<std::string, std::string>>;
  auto make_name = [](std::string fname, std::string lname) {
    return SingerFullName({"FirstName", std::move(fname)},
                          {"LastName", std::move(lname)});
  };
  using SongInfo =
      std::tuple<std::pair<std::string, std::string>,
                 std::pair<std::string, std::vector<SingerFullName>>>;
  auto songinfo = SongInfo(
      {"SongName", "Imagination"},
      {"ArtistNames",
       {make_name("Elena", "Campbell"), make_name("Hannah", "Harris")}});

  auto rows = client.ExecuteQuery(spanner::SqlStatement(
      "SELECT SingerId, @songinfo.SongName FROM Singers"
      " WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName)"
      "    IN UNNEST(@songinfo.ArtistNames)",
      {{"songinfo", spanner::Value(songinfo)}}));

  using RowType = std::tuple<std::int64_t, std::string>;
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " SongName: " << std::get<1>(*row) << "\n";
  }
  std::cout << "Query completed for [spanner_field_access_on_nested_struct]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithNestedStructFieldAsyncSample
{
    public async Task<List<int>> QueryDataWithNestedStructFieldAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        SpannerStruct name1 = new SpannerStruct
        {
            { "FirstName", SpannerDbType.String, "Elena" },
            { "LastName", SpannerDbType.String, "Campbell" }
        };
        SpannerStruct name2 = new SpannerStruct
        {
            { "FirstName", SpannerDbType.String, "Hannah" },
            { "LastName", SpannerDbType.String, "Harris" }
        };
        SpannerStruct songInfo = new SpannerStruct
        {
            { "song_name", SpannerDbType.String, "Imagination" },
            { "artistNames", SpannerDbType.ArrayOf(name1.GetSpannerDbType()), new[] { name1, name2 } }
        };

        var singerIds = new List<int>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            "SELECT SingerId, @song_info.song_name "
            + "FROM Singers WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) "
            + "IN UNNEST(@song_info.artistNames)");

        cmd.Parameters.Add("song_info", songInfo.GetSpannerDbType(), songInfo);

        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            var singerId = reader.GetFieldValue<int>("SingerId");
            singerIds.Add(singerId);
            Console.WriteLine($"SingerId: {singerId}");
            Console.WriteLine($"Song Name: {reader.GetFieldValue<string>(1)}");
        }
        return singerIds;
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

func queryWithNestedStructField(w io.Writer, db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 type nameType struct {
     FirstName string
     LastName  string
 }
 type songInfoStruct struct {
     SongName    string
     ArtistNames []nameType
 }
 var songInfo = songInfoStruct{
     SongName: "Imagination",
     ArtistNames: []nameType{
         {FirstName: "Elena", LastName: "Campbell"},
         {FirstName: "Hannah", LastName: "Harris"},
     },
 }
 stmt := spanner.Statement{
     SQL: `SELECT SingerId, @songinfo.SongName FROM Singers
         WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName)
         IN UNNEST(@songinfo.ArtistNames)`,
     Params: map[string]interface{}{"songinfo": songInfo},
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
     var singerID int64
     var songName string
     if err := row.Columns(&singerID, &songName); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s\n", singerID, songName)
 }
}
```

### Java

``` java
static void queryNestedStructField(DatabaseClient dbClient) {
  Type nameType =
      Type.struct(
          Arrays.asList(
              StructField.of("FirstName", Type.string()),
              StructField.of("LastName", Type.string())));

  Struct songInfo =
      Struct.newBuilder()
          .set("song_name")
          .to("Imagination")
          .set("artistNames")
          .toStructArray(
              nameType,
              Arrays.asList(
                  Struct.newBuilder()
                      .set("FirstName")
                      .to("Elena")
                      .set("LastName")
                      .to("Campbell")
                      .build(),
                  Struct.newBuilder()
                      .set("FirstName")
                      .to("Hannah")
                      .set("LastName")
                      .to("Harris")
                      .build()))
          .build();
  Statement s =
      Statement.newBuilder(
              "SELECT SingerId, @song_info.song_name "
                  + "FROM Singers WHERE "
                  + "STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) "
                  + "IN UNNEST(@song_info.artistNames)")
          .bind("song_info")
          .to(songInfo)
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(s)) {
    while (resultSet.next()) {
      System.out.printf("%d %s\n", resultSet.getLong("SingerId"), resultSet.getString(1));
    }
  }
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

const nameType = {
  type: 'struct',
  fields: [
    {
      name: 'FirstName',
      type: 'string',
    },
    {
      name: 'LastName',
      type: 'string',
    },
  ],
};

// Creates Song info STRUCT with a nested ArtistNames array
const songInfoType = {
  type: 'struct',
  fields: [
    {
      name: 'SongName',
      type: 'string',
    },
    {
      name: 'ArtistNames',
      type: 'array',
      child: nameType,
    },
  ],
};

const songInfoStruct = Spanner.struct({
  SongName: 'Imagination',
  ArtistNames: [
    Spanner.struct({FirstName: 'Elena', LastName: 'Campbell'}),
    Spanner.struct({FirstName: 'Hannah', LastName: 'Harris'}),
  ],
});

const query = {
  sql:
    'SELECT SingerId, @songInfo.SongName FROM Singers ' +
    'WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) ' +
    'IN UNNEST(@songInfo.ArtistNames)',
  params: {
    songInfo: songInfoStruct,
  },
  types: {
    songInfo: songInfoType,
  },
};

// Queries rows from the Singers table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(`SingerId: ${json.SingerId}, SongName: ${json.SongName}`);
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
use Google\Cloud\Spanner\StructType;
use Google\Cloud\Spanner\StructValue;
use Google\Cloud\Spanner\ArrayType;

/**
 * Queries sample data from the database using a nested struct field value.
 * Example:
 * ```
 * query_data_with_nested_struct_field($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_nested_struct_field(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $nameType = new ArrayType(
        (new StructType)
            ->add('FirstName', Database::TYPE_STRING)
            ->add('LastName', Database::TYPE_STRING)
    );
    $songInfoType = (new StructType)
        ->add('SongName', Database::TYPE_STRING)
        ->add('ArtistNames', $nameType);
    $nameStructValue1 = (new StructValue)
        ->add('FirstName', 'Elena')
        ->add('LastName', 'Campbell');
    $nameStructValue2 = (new StructValue)
        ->add('FirstName', 'Hannah')
        ->add('LastName', 'Harris');
    $songInfoValues = (new StructValue)
        ->add('SongName', 'Imagination')
        ->add('ArtistNames', [$nameStructValue1, $nameStructValue2]);
    $results = $database->execute(
        'SELECT SingerId, @song_info.SongName FROM Singers ' .
        'WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) ' .
        'IN UNNEST(@song_info.ArtistNames)',
        [
            'parameters' => [
                'song_info' => $songInfoValues
            ],
            'types' => [
                'song_info' => $songInfoType
            ]
        ]
    );
    foreach ($results as $row) {
        printf('SingerId: %s SongName: %s' . PHP_EOL,
            $row['SingerId'], $row['SongName']);
    }
}
````

### Python

``` python
def query_nested_struct_field(instance_id, database_id):
    """Query a table using nested field access on a STRUCT parameter."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    song_info_type = param_types.Struct(
        [
            param_types.StructField("SongName", param_types.STRING),
            param_types.StructField(
                "ArtistNames",
                param_types.Array(
                    param_types.Struct(
                        [
                            param_types.StructField("FirstName", param_types.STRING),
                            param_types.StructField("LastName", param_types.STRING),
                        ]
                    )
                ),
            ),
        ]
    )

    song_info = ("Imagination", [("Elena", "Campbell"), ("Hannah", "Harris")])

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId, @song_info.SongName "
            "FROM Singers WHERE "
            "STRUCT<FirstName STRING, LastName STRING>"
            "(FirstName, LastName) "
            "IN UNNEST(@song_info.ArtistNames)",
            params={"song_info": song_info},
            param_types={"song_info": song_info_type},
        )

    for row in results:
        print("SingerId: {} SongName: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

name_type = client.fields FirstName: :STRING, LastName: :STRING

song_info_struct = {
  SongName:    "Imagination",
  ArtistNames: [name_type.struct(["Elena", "Campbell"]), name_type.struct(["Hannah", "Harris"])]
}

client.execute(
  "SELECT SingerId, @song_info.SongName " \
  "FROM Singers WHERE STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) " \
  "IN UNNEST(@song_info.ArtistNames)",
  params: { song_info: song_info_struct }
).rows.each do |row|
  puts (row[:SingerId]), (row[:SongName])
end
```
