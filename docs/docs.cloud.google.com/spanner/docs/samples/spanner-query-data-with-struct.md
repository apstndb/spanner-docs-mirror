Query data by using a STRUCT object.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Working with STRUCT objects](/spanner/docs/structs)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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
  //! [spanner-sql-statement-params]

  for (auto& row : spanner::StreamOf<std::tuple<std::int64_t>>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\n";
  }
  std::cout << "Query completed for [spanner_query_data_with_struct]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
