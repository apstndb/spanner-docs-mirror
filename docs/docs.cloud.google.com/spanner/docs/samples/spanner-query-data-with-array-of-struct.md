Query data by using an array of STRUCT objects.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Working with STRUCT objects](/spanner/docs/structs)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
client.execute(
  "SELECT SingerId FROM Singers WHERE " +
  "STRUCT<FirstName STRING, LastName STRING>(FirstName, LastName) IN UNNEST(@names)",
  params: { names: band_members }
).rows.each do |row|
  puts row[:SingerId]
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
