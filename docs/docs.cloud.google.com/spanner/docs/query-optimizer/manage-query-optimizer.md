This page describes how to manage the query optimizer in Spanner for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

The Spanner query optimizer determines the most efficient way to execute a SQL query. However, the query plan determined by the optimizer might change slightly when the query optimizer itself evolves, or when the database statistics are updated. To minimize any potential for performance regression when the query optimizer or statistics change, Spanner provides the following query options.

  - **optimizer\_version** : Changes to the query optimizer are bundled and released as optimizer versions. Spanner starts using the latest version of the optimizer as the default at least 30 days after that version is released. You can use the query optimizer version option to run queries against an older version of the optimizer.

  - **optimizer\_statistics\_package** : Spanner updates optimizer statistics regularly. New statistics are made available as a package. This query option specifies a statistics package for the query optimizer to use when compiling a SQL query. The specified package must have garbage collection disabled:
    
    ### GoogleSQL
    
    ``` text
    ALTER STATISTICS <package_name> SET OPTIONS (allow_gc=false)
    ```
    
    ### PostgreSQL
    
    ``` text
    ALTER STATISTICS spanner."<package_name>" SET OPTIONS (allow_gc = false)
    ```

This guide shows how to set these individual options at different scopes in Spanner.

## List query optimizer options

Spanner stores information about the available optimizer versions and statistics packages that you can select.

### Optimizer versions

The query optimizer version is an integer value, incremented by 1 with each update. The latest version of the query optimizer is **8** .

Execute the following SQL statement to return a list of all supported optimizer versions, along with their corresponding release dates and whether that version is the default. The largest version number returned is the latest supported version of the optimizer.

``` text
SELECT * FROM SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS;
```

#### Default version

By default, Spanner starts using the latest version of the optimizer at least 30 days after that version is released. During the 30+ day period between a new release and that release becoming the default, you're encouraged to test queries against the new version to detect any regression.

To find the default version, execute the following SQL statement:

``` text
SELECT * FROM SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS;
```

The query returns a list of all supported optimizer versions. The `  IS_DEFAULT  ` column specifies which version is the current default.

For details about each version, see [Query optimizer version history](/spanner/docs/query-optimizer/versions) .

### Optimizer statistics packages

Each new optimizer statistics package that Spanner creates is assigned a package name that's guaranteed to be unique within the given database.

The format of the package name is `  auto_{PACKAGE_TIMESTAMP}UTC  ` . In GoogleSQL, the [`  ANALYZE  `](/spanner/docs/reference/standard-sql/data-definition-language#analyze-statistics) statement triggers the creation of the statistics package name. In PostgreSQL, the [`  ANALYZE  `](/spanner/docs/reference/postgresql/data-definition-language#analyze-statistics) statement performs this task. The format of the statistics package name is `  analyze_ {PACKAGE_TIMESTAMP} UTC  ` , where `  {PACKAGE_TIMESTAMP}  ` is the timestamp, in UTC timezone, of when the statistics construction started. Execute the following SQL statement to return a list of all available optimizer statistics packages.

``` text
SELECT * FROM INFORMATION_SCHEMA.SPANNER_STATISTICS;
```

By default, Spanner uses the latest optimizer statistics package unless the database or query is pinned to an older package using one of the methods described on this page.

## Option override precedence

If you're using a GoogleSQL-dialect database, Spanner offers multiple ways to change the optimizer options. For example, you can set the option(s) for a specific query or configure the option in the client library at the process or query level. When an option is set in multiple ways, the following precedence order applies. (Select a link to jump to that section in this document).

Spanner default ← [database option](#db-option) ← [client app](#client-app) ← [environment variable](#environment-variable) ← [client query](#client-query) ← [statement hint](#statement-hint)

For example, here's how to interpret the order of precedence when setting the query optimizer version:

When you create a database, it uses the Spanner [default optimizer version](/spanner/docs/query-optimizer/manage-query-optimizer#default_version) . Setting the optimizer version using one of the methods listed previously takes precedence over anything to the left of it. For example, setting the optimizer for an app using an [environment variable](#environment-variable) takes precedence over any value you set for the database using the [database option](#db-option) . Setting the optimizer version through a [statement hint](#statement-hint) has the highest precedence for the given query, taking precedence over the value set using any other method.

The following sections provided more details about each method.

## Set optimizer options

You can set the default optimizer option through the following methods:

  - [At the database level](#db-option)
  - [For a query using a statement hint](#statement-hint)
  - [With client libraries](#client-libraries)
      - [For a database client](#client-app)
      - [With environment variables](#environment-variable)
      - [For a client query](#client-query)
  - [Using the Spanner JDBC driver](#jdbc-driver)

In some cases, Spanner might use an older query optimizer version for a specific query shape, even if you have pinned a newer version. This is an expected internal behavior to ensure query stability and performance. You can identify the optimizer version that was used for a particular query by examining the [query execution plan](/spanner/docs/query-execution-plans) .

### Set optimizer options at the database level

To set the default optimizer version on a database, use the following [`  ALTER DATABASE  `](/spanner/docs/reference/standard-sql/data-definition-language#alter-database) DDL command. Setting this option doesn't require all queries to run that version. Instead, it sets an upper bound on the QO version used for queries. Its intended use is to mitigate regressions that occur after a new version of the optimizer is released.

### GoogleSQL

``` text
ALTER DATABASE MyDatabase
SET OPTIONS (optimizer_version =  8);
```

### PostgreSQL

``` text
ALTER DATABASE MyDatabase SET spanner.optimizer_version = 5;
```

You can set the statistics package similarly, as shown in the following example.

### GoogleSQL

``` text
ALTER DATABASE MyDatabase
SET OPTIONS (optimizer_statistics_package = "auto_20191128_14_47_22UTC");
```

### PostgreSQL

``` text
ALTER DATABASE MyDatabase
SET spanner.optimizer_statistics_package = "auto_20191128_14_47_22UTC";
```

You can also set more than one option at the same time, as shown in the following DDL command.

### GoogleSQL

``` text
ALTER DATABASE MyDatabase
SET OPTIONS (optimizer_version = 8,
            optimizer_statistics_package = "auto_20191128_14_47_22UTC");
```

You can run `  ALTER DATABASE  ` in gcloud CLI with the [`  gcloud CLI databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) command as follows.

### GoogleSQL

``` text
gcloud spanner databases ddl update MyDatabase --instance=test-instance \
    --ddl='ALTER DATABASE MyDatabase SET OPTIONS ( optimizer_version = 8 )'
```

### PostgreSQL

``` text
gcloud spanner databases ddl update MyDatabase --instance=test-instance \
  --ddl='ALTER DATABASE MyDatabase SET spanner.optimizer_version = 8'
```

Setting a database option to `  NULL  ` (in GoogleSQL) or `  DEFAULT  ` (in PostgreSQL) clears it so that the default value is used.

To see the current value of these options for a database, query the [`  INFORMATION_SCHEMA.DATABASE_OPTIONS  `](/spanner/docs/information-schema#database-option-optimizer) view for GoogleSQL, or the [`  information_schema database_options  `](/spanner/docs/information-schema-pg#database-option-optimizer) table for PostgreSQL, as follows.

### GoogleSQL

``` text
SELECT
  s.OPTION_NAME,
  s.OPTION_VALUE
FROM
  INFORMATION_SCHEMA.DATABASE_OPTIONS s
WHERE
  s.SCHEMA_NAME=""
  AND s.OPTION_NAME IN ('optimizer_version', 'optimizer_statistics_package')
```

### PostgreSQL

``` text
  SELECT
    s.option_name,
    s.option_value
  FROM
    information_schema.database_options s
  WHERE
    s.schema_name='public'
    AND s.option_name IN ('optimizer_version',
      'optimizer_statistics_package')
```

### Set optimizer options for a query using a statement hint

A [statement hint](/spanner/docs/reference/standard-sql/query-syntax#statement-hints) is a hint on a query statement that changes the execution of the query from the default behavior. Setting the `  OPTIMIZER_VERSION  ` hint on a statement forces that query to run using the specified query optimizer version.

The `  OPTIMIZER_VERSION  ` hint has the highest optimizer version precedence. If the statement hint is specified, it's used regardless of all other optimizer version settings.

### GoogleSQL

``` text
@{OPTIMIZER_VERSION=8} SELECT * FROM MyTable;
```

### PostgreSQL

``` text
/*@OPTIMIZER_VERSION=8*/ SELECT * FROM MyTable;
```

You can also use the **latest\_version** literal to set the optimizer version for a query to the latest version as shown here.

### GoogleSQL

``` text
@{OPTIMIZER_VERSION=latest_version} SELECT * FROM MyTable;
```

### PostgreSQL

``` text
/*@OPTIMIZER_VERSION=latest_version*/ SELECT * FROM MyTable;
```

Setting the `  OPTIMIZER_STATISTICS_PACKAGE  ` hint on a statement forces that query to run using the specified query optimizer statistics package version. The specified package [must have garbage collection disabled](#handle-invalid-setting) :

### GoogleSQL

``` text
ALTER STATISTICS <package_name> SET OPTIONS (allow_gc=false)
```

### PostgreSQL

``` text
ALTER STATISTICS spanner."<package_name>" SET OPTIONS (allow_gc=false)
```

The `  OPTIMIZER_STATISTICS_PACKAGE  ` hint has the highest optimizer package setting precedence. If the statement hint is specified, it's used regardless of all other optimizer package version settings.

``` text
@{OPTIMIZER_STATISTICS_PACKAGE=auto_20191128_14_47_22UTC} SELECT * FROM MyTable;
```

You can also use the **latest** literal to use the latest statistics package.

``` text
@{OPTIMIZER_STATISTICS_PACKAGE=latest} SELECT * FROM MyTable;
```

Both hints can be set in a single statement as shown in the following example.

The **default\_version** literal sets the optimizer version for a query to the default version, which might be different than the latest version. See [Default version](#default-version) for details.

### GoogleSQL

``` text
@{OPTIMIZER_VERSION=default_version, OPTIMIZER_STATISTICS_PACKAGE=auto_20191128_14_47_22UTC} SELECT * FROM MyTable;
```

### PostgreSQL

``` text
/*@OPTIMIZER_VERSION=default_version, OPTIMIZER_STATISTICS_PACKAGE=auto_20191128_14_47_22UTC*/ SELECT * FROM KeyValue;
```

### Set optimizer options with client libraries

When you are programmatically interacting with Spanner through client libraries, there are a number of ways to change query options for your client application.

You must be using the latest versions of the client libraries to set optimizer options.

#### For a database client

An application can set optimizer options globally on the client library by configuring the query options property as shown in the following code snippets. The optimizer settings are stored in the client instance and are applied to all queries run throughout the lifetime of the client. Even though the options apply at a database level in the backend, when the options are set at a client level, they apply to all databases connected to that client.

### C++

``` cpp
namespace spanner = ::google::cloud::spanner;
spanner::Client client(
    spanner::MakeConnection(db),
    google::cloud::Options{}
        .set<spanner::QueryOptimizerVersionOption>("1")
        .set<spanner::QueryOptimizerStatisticsPackageOption>(
            "auto_20191128_14_47_22UTC"));
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class CreateConnectionWithQueryOptionsAsyncSample
{
    public class Album
    {
        public int AlbumId { get; set; }
        public int SingerId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> CreateConnectionWithQueryOptionsAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString)
        {
            // Set query options on the connection.
            QueryOptions = QueryOptions.Empty
                .WithOptimizerVersion("1")
                // The list of available statistics packages for the database can
                // be found by querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS"
                // table.
                .WithOptimizerStatisticsPackage("latest")
        };

        var albums = new List<Album>();
        var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
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
 "time"

 "cloud.google.com/go/spanner"
 sppb "cloud.google.com/go/spanner/apiv1/spannerpb"
 "google.golang.org/api/iterator"
)

func createClientWithQueryOptions(w io.Writer, database string) error {
 ctx := context.Background()
 queryOptions := spanner.QueryOptions{
     Options: &sppb.ExecuteSqlRequest_QueryOptions{
         OptimizerVersion: "1",
         // The list of available statistics packages can be found by
         // querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
         OptimizerStatisticsPackage: "latest",
     },
 }
 client, err := spanner.NewClientWithConfig(
     ctx, database, spanner.ClientConfig{QueryOptions: queryOptions},
 )
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT VenueId, VenueName, LastUpdateTime FROM Venues`}
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
     var venueName string
     var lastUpdateTime time.Time
     if err := row.Columns(&venueID, &venueName, &lastUpdateTime); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %s\n", venueID, venueName, lastUpdateTime)
 }
}
```

### Java

``` java
static void clientWithQueryOptions(DatabaseId db) {
  SpannerOptions options =
      SpannerOptions.newBuilder()
          .setDefaultQueryOptions(
              db, QueryOptions
                  .newBuilder()
                  .setOptimizerVersion("1")
                  // The list of available statistics packages can be found by querying the
                  // "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
                  .setOptimizerStatisticsPackage("latest")
                  .build())
          .build();
  Spanner spanner = options.getService();
  DatabaseClient dbClient = spanner.getDatabaseClient(db);
  try (ResultSet resultSet =
      dbClient
          .singleUse()
          .executeQuery(Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"))) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s\n", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
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
const database = instance.database(
  databaseId,
  {},
  {
    optimizerVersion: '1',
    // The list of available statistics packages can be found by querying the
    // "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
    optimizerStatisticsPackage: 'latest',
  },
);

const query = {
  sql: `SELECT AlbumId, AlbumTitle, MarketingBudget
        FROM Albums
        ORDER BY AlbumTitle`,
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    const marketingBudget = json.MarketingBudget
      ? json.MarketingBudget
      : null; // This value is nullable
    console.log(
      `AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}, MarketingBudget: ${marketingBudget}`,
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

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Database;

/**
 * Create a client with query options.
 * Example:
 * ```
 * create_client_with_query_options($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function create_client_with_query_options(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient([
        'queryOptions' => [
            'optimizerVersion' => '1',
            // Pin the statistics package used for this client instance to the
            // latest version. The list of available statistics packages can be
            // found by querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS"
            // table.
            'optimizerStatisticsPackage' => 'latest'
        ]
    ]);
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT VenueId, VenueName, LastUpdateTime FROM Venues'
    );

    foreach ($results as $row) {
        printf('VenueId: %s, VenueName: %s, LastUpdateTime: %s' . PHP_EOL,
            $row['VenueId'], $row['VenueName'], $row['LastUpdateTime']);
    }
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client(
    query_options={
        "optimizer_version": "1",
        "optimizer_statistics_package": "latest",
    }
)
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName, LastUpdateTime FROM Venues"
    )

    for row in results:
        print("VenueId: {}, VenueName: {}, LastUpdateTime: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

query_options = {
  optimizer_version: "1",
  # The list of available statistics packages can be
  # found by querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS"
  # table.
  optimizer_statistics_package: "latest"
}

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id, query_options: query_options

sql_query = "SELECT VenueId, VenueName, LastUpdateTime FROM Venues"

client.execute(sql_query).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]} #{row[:LastUpdateTime]}"
end
```

#### With environment variables

To make it easier to try different optimizer settings without having to recompile your app, you can set the `  SPANNER_OPTIMIZER_VERSION  ` and `  SPANNER_OPTIMIZER_STATISTICS_PACKAGE  ` environment variables and run your app, as shown in the following snippet.

### Linux / macOS

``` text
export SPANNER_OPTIMIZER_VERSION="8"
export SPANNER_OPTIMIZER_STATISTICS_PACKAGE="auto_20191128_14_47_22UTC"
```

### Windows

``` text
set SPANNER_OPTIMIZER_VERSION="8"
  set SPANNER_OPTIMIZER_STATISTICS_PACKAGE="auto_20191128_14_47_22UTC"
```

The specified query optimizer options values are read and stored in the client instance at client initialization time and apply to all queries run throughout the lifetime of the client.

#### For a client query

You can specify a value for optimizer version or statistics package version at the query level in your client application by specifying a query options property when building your query.

### C++

``` cpp
void QueryWithQueryOptions(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto sql = spanner::SqlStatement("SELECT SingerId, FirstName FROM Singers");
  auto opts =
      google::cloud::Options{}
          .set<spanner::QueryOptimizerVersionOption>("1")
          .set<spanner::QueryOptimizerStatisticsPackageOption>("latest");
  auto rows = client.ExecuteQuery(std::move(sql), std::move(opts));

  using RowType = std::tuple<std::int64_t, std::string>;
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\t";
    std::cout << "FirstName: " << std::get<1>(*row) << "\n";
  }
  std::cout << "Read completed for [spanner_query_with_query_options]\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class RunCommandWithQueryOptionsAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> RunCommandWithQueryOptionsAsync(string projectId, string instanceId, string databaseId)
    {
        var connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

        cmd.QueryOptions = QueryOptions.Empty
            .WithOptimizerVersion("1")
            // The list of available statistics packages for the database can
            // be found by querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS"
            // table.
            .WithOptimizerStatisticsPackage("latest");
        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album()
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
 "time"

 "cloud.google.com/go/spanner"
 sppb "cloud.google.com/go/spanner/apiv1/spannerpb"
 "google.golang.org/api/iterator"
)

func queryWithQueryOptions(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT VenueId, VenueName, LastUpdateTime FROM Venues`}
 queryOptions := spanner.QueryOptions{
     Options: &sppb.ExecuteSqlRequest_QueryOptions{
         OptimizerVersion: "1",
         // The list of available statistics packages can be found by
         // querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
         OptimizerStatisticsPackage: "latest",
     },
 }
 iter := client.Single().QueryWithOptions(ctx, stmt, queryOptions)
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
     var venueName string
     var lastUpdateTime time.Time
     if err := row.Columns(&venueID, &venueName, &lastUpdateTime); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %s\n", venueID, venueName, lastUpdateTime)
 }
}
```

### Java

``` java
static void queryWithQueryOptions(DatabaseClient dbClient) {
  try (ResultSet resultSet =
      dbClient
          .singleUse()
          .executeQuery(
              Statement
                  .newBuilder("SELECT SingerId, AlbumId, AlbumTitle FROM Albums")
                  .withQueryOptions(QueryOptions
                      .newBuilder()
                      .setOptimizerVersion("1")
                      // The list of available statistics packages can be found by querying the
                      // "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
                      .setOptimizerStatisticsPackage("latest")
                      .build())
                  .build())) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s\n", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
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

const query = {
  sql: `SELECT AlbumId, AlbumTitle, MarketingBudget
        FROM Albums
        ORDER BY AlbumTitle`,
  queryOptions: {
    optimizerVersion: 'latest',
    // The list of available statistics packages can be found by querying the
    // "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
    optimizerStatisticsPackage: 'latest',
  },
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    const marketingBudget = json.MarketingBudget
      ? json.MarketingBudget
      : null; // This value is nullable
    console.log(
      `AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}, MarketingBudget: ${marketingBudget}`,
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

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Database;

/**
 * Queries sample data using SQL with query options.
 * Example:
 * ```
 * query_data_with_query_options($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_query_options(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT VenueId, VenueName, LastUpdateTime FROM Venues',
        [
            'queryOptions' => [
                'optimizerVersion' => '1',
                // Pin the statistics package to the latest version just for
                // this query.
                'optimizerStatisticsPackage' => 'latest'
            ]
        ]
    );

    foreach ($results as $row) {
        printf('VenueId: %s, VenueName: %s, LastUpdateTime: %s' . PHP_EOL,
            $row['VenueId'], $row['VenueName'], $row['LastUpdateTime']);
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

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName, LastUpdateTime FROM Venues",
        query_options={
            "optimizer_version": "1",
            "optimizer_statistics_package": "latest",
        },
    )

    for row in results:
        print("VenueId: {}, VenueName: {}, LastUpdateTime: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

sql_query = "SELECT VenueId, VenueName, LastUpdateTime FROM Venues"
query_options = {
  optimizer_version: "1",
  # The list of available statistics packagebs can be
  # found by querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS"
  # table.
  optimizer_statistics_package: "latest"
}

client.execute(sql_query, query_options: query_options).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]} #{row[:LastUpdateTime]}"
end
```

### Set optimizer options when using the Spanner JDBC driver

You can override the default value of the optimizer version and statistics package by specifying options in the JDBC connection string as shown in the following example.

These options are only supported in the latest versions of the [Spanner JDBC driver](/spanner/docs/open-source-jdbc) .

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

class ConnectionWithQueryOptionsExample {

  static void connectionWithQueryOptions() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    connectionWithQueryOptions(projectId, instanceId, databaseId);
  }

  static void connectionWithQueryOptions(String projectId, String instanceId, String databaseId)
      throws SQLException {
    String optimizerVersion = "1";
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s?optimizerVersion=%s",
            projectId, instanceId, databaseId, optimizerVersion);
    try (Connection connection = DriverManager.getConnection(connectionUrl);
        Statement statement = connection.createStatement()) {
      // Execute a query using the optimizer version '1'.
      try (ResultSet rs =
          statement.executeQuery(
              "SELECT SingerId, FirstName, LastName FROM Singers ORDER BY LastName")) {
        while (rs.next()) {
          System.out.printf("%d %s %s%n", rs.getLong(1), rs.getString(2), rs.getString(3));
        }
      }
      try (ResultSet rs = statement.executeQuery("SHOW VARIABLE OPTIMIZER_VERSION")) {
        while (rs.next()) {
          System.out.printf("Optimizer version: %s%n", rs.getString(1));
        }
      }
    }
  }
}
```

You can also set the query optimizer version using the `  SET OPTIMIZER_VERSION  ` statement as shown in the following example.

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

class SetQueryOptionsExample {

  static void setQueryOptions() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    setQueryOptions(projectId, instanceId, databaseId);
  }

  static void setQueryOptions(String projectId, String instanceId, String databaseId)
      throws SQLException {
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
            projectId, instanceId, databaseId);
    try (Connection connection = DriverManager.getConnection(connectionUrl);
        Statement statement = connection.createStatement()) {
      // Instruct the JDBC connection to use version '1' of the query optimizer.
      // NOTE: Use `SET SPANNER.OPTIMIZER_VERSION='1`` when connected to a PostgreSQL database.
      statement.execute("SET OPTIMIZER_VERSION='1'");
      // Execute a query using the latest optimizer version.
      try (ResultSet rs =
          statement.executeQuery(
              "SELECT SingerId, FirstName, LastName FROM Singers ORDER BY LastName")) {
        while (rs.next()) {
          System.out.printf("%d %s %s%n", rs.getLong(1), rs.getString(2), rs.getString(3));
        }
      }
      // NOTE: Use `SHOW SPANNER.OPTIMIZER_VERSION` when connected to a PostgreSQL database.
      try (ResultSet rs = statement.executeQuery("SHOW VARIABLE OPTIMIZER_VERSION")) {
        while (rs.next()) {
          System.out.printf("Optimizer version: %s%n", rs.getString(1));
        }
      }
    }
  }
}
```

For more details on using the open source driver, see [Using the open source JDBC driver](/spanner/docs/use-oss-jdbc) .

## How invalid optimizer versions are handled

Spanner supports a [range](#list-optimizer-versions) of optimizer versions. This range changes over time when the query optimizer is updated. If the version you specify is out of range, the query fails. For example, if you attempt to run a query with the statement hint `  @{OPTIMIZER_VERSION=9}  ` , but the most recent optimizer version number is only `  8  ` , Spanner responds with this error message:

`  Query optimizer version: 9 is not supported  `

### Handle an invalid optimizer statistics package setting

You can pin your database or query to any [available statistics package](#list-statistics-packages) using one of the methods described earlier on this page. A query fails if an invalid statistics package name is provided. A statistics package specified by a query needs to be either:

  - [set at the database level](#db-option) ; or
  - [marked as `  ALLOW_GC=false  `](#statement-hint)

## Determine the query optimizer version used to run a query

The optimizer version used for a query is visible through the Google Cloud console and in the Google Cloud CLI.

### Google Cloud console

To view the optimizer version used for a query, run your query in the **Spanner Studio** page of the Google Cloud console, and then select the **Explanation** tab. You should see a message similar to the following:

Query optimizer version: 8

### gcloud CLI

To see the version used when running a query in gcloud CLI, set the `  --query-mode  ` flag to `  PROFILE  ` as shown in the following snippet.

``` text
gcloud spanner databases execute-sql MyDatabase --instance=test-instance \
    --query-mode=PROFILE --sql='SELECT * FROM MyTable'
```

## Visualize query optimizer version in Metrics Explorer

Cloud Monitoring collects measurements to help you understand how your applications and system services are performing. One of the metrics collected for Spanner is [count of queries](/monitoring/api/metrics_gcp_p_z#gcp-spanner) , which measures the number of queries in an instance, sampled over time. While this metric is very useful to view queries grouped by error code, we can also use it to see what optimizer version was used to run each query.

You can use [Metrics Explorer](/monitoring/charts/metrics-explorer) in Google Cloud console to visualize **Count of queries** for your database instance. Figure 1 shows the count of queries for three databases. You can see which optimizer version is being used in each database.

The table below the chart in this figure shows that `  my-db-1  ` attempted to run a query with an invalid optimizer version, returning the status **Bad usage** and resulting in a query count of 0. The other databases ran queries using versions 1 and 2 of the optimizer respectively.

**Figure 1.** **Count of queries** displayed in Metrics Explorer with queries grouped by optimizer version.

To set up a similar chart for your instance:

1.  Navigate to the [Metrics Explorer](/monitoring/charts/metrics-explorer#find-me) in the Google Cloud console.
2.  In the **Resource type** field, select `  Cloud Spanner Instance  ` .
3.  In the **Metric** field, select `  Count of queries  ` .
4.  In the **Group By** field, select `  database  ` , `  optimizer_version  ` , and `  status  ` .

Not shown in this example is the case where a different optimizer version is being used for different queries in the same database. In that case, the chart would display a bar segment for each combination of database and optimizer version.

To learn how to use Cloud Monitoring to monitor your Spanner instances, see [Monitoring with Cloud Monitoring](/spanner/docs/monitoring-cloud)
