Query data by using a NUMERIC parameter.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Work with NUMERIC data](/spanner/docs/working-with-numerics)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
