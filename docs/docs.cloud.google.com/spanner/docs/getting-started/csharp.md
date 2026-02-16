## Objectives

This tutorial walks you through the following steps using the Spanner client library for C\#:

  - Create a Spanner instance and database.
  - Write, read, and execute SQL queries on data in the database.
  - Update the database schema.
  - Update data using a read-write transaction.
  - Add a secondary index to the database.
  - Use the index to read and execute SQL queries on data.
  - Retrieve data using a read-only transaction.

## Costs

This tutorial uses Spanner, which is a billable component of the Google Cloud. For information on the cost of using Spanner, see [Pricing](https://cloud.google.com/spanner/pricing) .

## Before you begin

Complete the steps described in [Set up](/spanner/docs/getting-started/set-up#set_up_a_project) , which cover creating and setting a default Google Cloud project, enabling billing, enabling the Cloud Spanner API, and setting up OAuth 2.0 to get authentication credentials to use the Cloud Spanner API.

In particular, make sure that you run [`  gcloud auth application-default login  `](/sdk/gcloud/reference/auth/application-default/login) to set up your local development environment with authentication credentials.

**Note:** If you don't plan to keep the resources that you create in this tutorial, consider creating a new Google Cloud project instead of selecting an existing project. After you finish the tutorial, you can delete the project, removing all resources associated with the project.

## Prepare your local C\# environment

1.  Set the `  PROJECT_ID  ` environment variable to your Google Cloud project ID.
    
    1.  First, set `  PROJECT_ID  ` for the current PowerShell session:
        
        ``` text
        $env:PROJECT_ID = "MY_PROJECT_ID"
        ```
    
    2.  Then, set `  PROJECT_ID  ` for all processes created after this command:
        
        ``` text
        [Environment]::SetEnvironmentVariable("PROJECT_ID", "MY_PROJECT_ID", "User")
        ```

2.  Download credentials.
    
    1.  Go to the **Credentials** page in the Google Cloud console.
    
    2.  Click **Create credentials** and choose **Service account key** .
    
    3.  Under "Service account", choose **Compute Engine default service account** , and leave **JSON** selected under "Key type". Click **Create** . Your computer downloads a JSON file.

3.  Set up credentials. For a file named `  FILENAME .json  ` in `  CURRENT_USER  ` 's Downloads directory, located on the `  C  ` drive, run the following commands to set `  GOOGLE_APPLICATION_CREDENTIALS  ` to point to the JSON key:
    
    1.  First, to set `  GOOGLE_APPLICATION_CREDENTIALS  ` for this PowerShell session:
        
        ``` text
        $env:GOOGLE_APPLICATION_CREDENTIALS = "C:\Users\CURRENT_USER\Downloads\FILENAME.json"
        ```
    
    2.  Then, to set `  GOOGLE_APPLICATION_CREDENTIALS  ` for all processes created after this command:
        
        ``` text
        [Environment]::SetEnvironmentVariable("GOOGLE_APPLICATION_CREDENTIALS", "C:\Users\CURRENT_USER\Downloads\FILENAME.json", "User")
        ```

4.  Clone the sample app repository to your local machine:
    
    ``` text
    git clone https://github.com/GoogleCloudPlatform/dotnet-docs-samples
    ```
    
    Alternatively, you can [download the sample](https://github.com/GoogleCloudPlatform/dotnet-docs-samples/archive/main.zip) as a zip file and extract it.

5.  Open `  Spanner.sln  ` , located in the `  dotnet-docs-samples\spanner\api  ` directory of the downloaded repository, with Visual Studio 2017 or later, then build it.

6.  Change to the directory within the downloaded repository that contains the compiled application. For example:
    
    ``` text
    cd dotnet-docs-samples\spanner\api\Spanner
    ```

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with C\#.

Take a look through the [Spanner .NET GitHub repository](https://github.com/GoogleCloudPlatform/dotnet-docs-samples/tree/main/spanner/api) , which shows how to create a database and modify a database schema. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

You should see:

The following code creates a database and two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

``` csharp
using Google.Cloud.Spanner.Data;
using System.Threading.Tasks;

public class CreateDatabaseAsyncSample
{
    public async Task CreateDatabaseAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}";

        using var connection = new SpannerConnection(connectionString);
        var createDatabase = $"CREATE DATABASE `{databaseId}`";
        // Define create table statement for table #1.
        var createSingersTable =
            @"CREATE TABLE Singers (
                SingerId INT64 NOT NULL,
                FirstName STRING(1024),
                LastName STRING(1024),
                ComposerInfo BYTES(MAX),
                FullName STRING(2048) AS (ARRAY_TO_STRING([FirstName, LastName], "" "")) STORED
            ) PRIMARY KEY (SingerId)";
        // Define create table statement for table #2.
        var createAlbumsTable =
            @"CREATE TABLE Albums (
                SingerId INT64 NOT NULL,
                AlbumId INT64 NOT NULL,
                AlbumTitle STRING(MAX)
            ) PRIMARY KEY (SingerId, AlbumId),
            INTERLEAVE IN PARENT Singers ON DELETE CASCADE";

        using var createDbCommand = connection.CreateDdlCommand(createDatabase, createSingersTable, createAlbumsTable);
        await createDbCommand.ExecuteNonQueryAsync();
    }
}
```

### PostgreSQL

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Threading.Tasks;

public class CreateDatabaseAsyncPostgresSample
{
    public async Task CreateDatabaseAsyncPostgres(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = await DatabaseAdminClient.CreateAsync();

        // Create the CreateDatabaseRequest with PostgreSQL dialect and execute it.
        // There cannot be Extra DDL statements while creating PostgreSQL.
        var createDatabaseRequest = new CreateDatabaseRequest
        {
            ParentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId),
            CreateStatement = $"CREATE DATABASE \"{databaseId}\"",
            DatabaseDialect = DatabaseDialect.Postgresql
        };

        var createOperation = await databaseAdminClient.CreateDatabaseAsync(createDatabaseRequest);

        // Wait until the operation has finished.
        Console.WriteLine("Waiting for the database to be created.");
        var completedResponse = await createOperation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Error while creating PostgreSQL database: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        // PostgreSQL Database is created. Now, we can create the tables.
        // Define create table statement for table #1 in PostgreSQL syntax.
        var createSingersTable = @"CREATE TABLE Singers (
            SingerId bigint NOT NULL PRIMARY KEY,
            FirstName varchar(1024),
            LastName varchar(1024),
            Rating numeric,
            SingerInfo bytea,
            FullName character varying(2048) GENERATED ALWAYS AS (FirstName || ' ' || LastName) STORED)";

        // Define create table statement for table #2 in PostgreSQL syntax.
        var createAlbumsTable = @"CREATE TABLE Albums (
            AlbumId bigint NOT NULL PRIMARY KEY,
            SingerId bigint NOT NULL REFERENCES Singers (SingerId),
            AlbumTitle text,
            MarketingBudget BIGINT)";

        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);

        // Create UpdateDatabaseRequest to create the tables. 
        var updateDatabaseRequest = new UpdateDatabaseDdlRequest
        {
            DatabaseAsDatabaseName = databaseName,
            Statements = { createSingersTable, createAlbumsTable }
        };

        var updateOperation = await databaseAdminClient.UpdateDatabaseDdlAsync(updateDatabaseRequest);
        // Wait until the operation has finished.
        Console.WriteLine("Waiting for the tables to be created.");
        var updateResponse = await updateOperation.PollUntilCompletedAsync();
        if (updateResponse.IsFaulted)
        {
            Console.WriteLine($"Error while updating database: {updateResponse.Exception}");
            throw updateResponse.Exception;
        }
    }
}
```

The next step is to write data to your database.

## Create a database client

Before you can do reads or writes, you must create a [`  SpannerConnection  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection) :

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

namespace GoogleCloudSamples.Spanner
{
    public class QuickStart
    {
        static async Task MainAsync()
        {
            string projectId = "YOUR-PROJECT-ID";
            string instanceId = "my-instance";
            string databaseId = "my-database";
            string connectionString =
                $"Data Source=projects/{projectId}/instances/{instanceId}/"
                + $"databases/{databaseId}";
            // Create connection to Cloud Spanner.
            using (var connection = new SpannerConnection(connectionString))
            {
                // Execute a simple SQL statement.
                var cmd = connection.CreateSelectCommand(
                    @"SELECT ""Hello World"" as test");
                using (var reader = await cmd.ExecuteReaderAsync())
                {
                    while (await reader.ReadAsync())
                    {
                        Console.WriteLine(
                            reader.GetFieldValue<string>("test"));
                    }
                }
            }
        }
        public static void Main(string[] args)
        {
            MainAsync().Wait();
        }
    }
}
```

You can think of a `  SpannerConnection  ` as a database connection: all of your interactions with Spanner must go through a `  SpannerConnection  ` .

Read more in the [`  SpannerConnection  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection) reference.

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  ExecuteNonQueryAsync()  ` method to execute a DML statement.

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class WriteUsingDmlCoreAsyncSample
{
    public async Task<int> WriteUsingDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        SpannerCommand cmd = connection.CreateDmlCommand(
            "INSERT Singers (SingerId, FirstName, LastName) VALUES "
               + "(12, 'Melissa', 'Garcia'), "
               + "(13, 'Russell', 'Morales'), "
               + "(14, 'Jacqueline', 'Long'), "
               + "(15, 'Dylan', 'Shaw')");
        int rowCount = await cmd.ExecuteNonQueryAsync();

        Console.WriteLine($"{rowCount} row(s) inserted...");
        return rowCount;
    }
}
```

Run the sample using the `  writeUsingDml  ` argument.

``` text
dotnet run writeUsingDml $env:PROJECT_ID test-instance example-db
```

You should see:

``` text
4 row(s) inserted...
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

You can insert data using the [`  connection.CreateInsertCommand()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_CreateInsertCommand_System_String_Google_Cloud_Spanner_Data_SpannerParameterCollection_) method, which creates a new `  SpannerCommand  ` to insert rows into a table. The [`  SpannerCommand.ExecuteNonQueryAsync()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerCommand#Google_Cloud_Spanner_Data_SpannerCommand_ExecuteNonQueryAsync_System_Threading_CancellationToken_) method adds new rows to the table.

This code shows how to insert data using mutations:

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class InsertDataAsyncSample
{
    public class Singer
    {
        public int SingerId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task InsertDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        List<Singer> singers = new List<Singer>
        {
            new Singer { SingerId = 1, FirstName = "Marc", LastName = "Richards" },
            new Singer { SingerId = 2, FirstName = "Catalina", LastName = "Smith" },
            new Singer { SingerId = 3, FirstName = "Alice", LastName = "Trentor" },
            new Singer { SingerId = 4, FirstName = "Lea", LastName = "Martin" },
            new Singer { SingerId = 5, FirstName = "David", LastName = "Lomond" },
        };
        List<Album> albums = new List<Album>
        {
            new Album { SingerId = 1, AlbumId = 1, AlbumTitle = "Total Junk" },
            new Album { SingerId = 1, AlbumId = 2, AlbumTitle = "Go, Go, Go" },
            new Album { SingerId = 2, AlbumId = 1, AlbumTitle = "Green" },
            new Album { SingerId = 2, AlbumId = 2, AlbumTitle = "Forever Hold your Peace" },
            new Album { SingerId = 2, AlbumId = 3, AlbumTitle = "Terrified" },
        };

        // Create connection to Cloud Spanner.
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        await connection.RunWithRetriableTransactionAsync(async transaction =>
        {
            await Task.WhenAll(singers.Select(singer =>
            {
                // Insert rows into the Singers table.
                using var cmd = connection.CreateInsertCommand("Singers", new SpannerParameterCollection
                {
                        { "SingerId", SpannerDbType.Int64, singer.SingerId },
                        { "FirstName", SpannerDbType.String, singer.FirstName },
                        { "LastName", SpannerDbType.String, singer.LastName }
                });
                cmd.Transaction = transaction;
                return cmd.ExecuteNonQueryAsync();
            }));

            await Task.WhenAll(albums.Select(album =>
            {
                // Insert rows into the Albums table.
                using var cmd = connection.CreateInsertCommand("Albums", new SpannerParameterCollection
                {
                        { "SingerId", SpannerDbType.Int64, album.SingerId },
                        { "AlbumId", SpannerDbType.Int64, album.AlbumId },
                        { "AlbumTitle", SpannerDbType.String,album.AlbumTitle }
                });
                cmd.Transaction = transaction;
                return cmd.ExecuteNonQueryAsync();
            }));
        });
        Console.WriteLine("Data inserted.");
    }
}
```

Run the sample using the `  insertSampleData  ` argument.

``` text
dotnet run insertSampleData $env:PROJECT_ID test-instance example-db
```

You should see:

``` text
Inserted data.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner client library for C\#.

### On the command line

Execute the following SQL statement to read the values of all columns from the `  Albums  ` table:

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql='SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
```

**Note:** For the GoogleSQL reference, see [Query syntax in GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax) and for PostgreSQL reference, see [PostgreSQL lexical structure and syntax](/spanner/docs/reference/postgresql/lexical) .

The result shows:

``` text
SingerId AlbumId AlbumTitle
1        1       Total Junk
1        2       Go, Go, Go
2        1       Green
2        2       Forever Hold Your Peace
2        3       Terrified
```

### Use the Spanner client library for C\#

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner client library for C\#.

Use [`  ExecuteReaderAsync()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerCommand#Google_Cloud_Spanner_Data_SpannerCommand_ExecuteReaderAsync) to run the SQL query.

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QuerySampleDataAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> QuerySampleDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

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

Here's how to issue the query and access the data:

``` text
dotnet run querySampleData $env:PROJECT_ID test-instance example-db
```

You should see the following result:

``` text
SingerId: 1 AlbumId: 1 AlbumTitle: Total Junk
SingerId: 1 AlbumId: 2 AlbumTitle: Go, Go, Go
SingerId: 2 AlbumId: 1 AlbumTitle: Green
SingerId: 2 AlbumId: 2 AlbumTitle: Forever Hold your Peace
SingerId: 2 AlbumId: 3 AlbumTitle: Terrified
```

### Query using a SQL parameter

If your application has a frequently executed query, you can improve its performance by parameterizing it. The resulting parametric query can be cached and reused, which reduces compilation costs. For more information, see [Use query parameters to speed up frequently executed queries](/spanner/docs/sql-best-practices#query-parameters) .

Here is an example of using a parameter in the `  WHERE  ` clause to query records containing a specific value for `  LastName  ` .

### GoogleSQL

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryWithParameterAsyncSample
{
    public class Singer
    {
        public int SingerId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public async Task<List<Singer>> QueryWithParameterAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            $"SELECT SingerId, FirstName, LastName FROM Singers WHERE LastName = @lastName",
            new SpannerParameterCollection { { "lastName", SpannerDbType.String, "Garcia" } });

        var singers = new List<Singer>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            singers.Add(new Singer
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                FirstName = reader.GetFieldValue<string>("FirstName"),
                LastName = reader.GetFieldValue<string>("LastName")
            });
        }
        return singers;
    }
}
```

### PostgreSQL

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryUsingParametersAsyncPostgresSample
{
    public async Task<List<Singer>> QueryUsingParametersAsyncPostgres(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateSelectCommand("SELECT SingerId, FirstName, LastName FROM Singers WHERE LastName LIKE $1",
            new SpannerParameterCollection
            {
                {"p1", SpannerDbType.String, "N%" }
            });

        var list = new List<Singer>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            list.Add(new Singer
            {
                // See https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS
                // to understand why column names are in lower case.
                Id = reader.GetFieldValue<long>("singerid"),
                FirstName = reader.GetFieldValue<string>("firstname"),
                LastName = reader.GetFieldValue<string>("lastname")
            });
        }

        return list;
    }

    public struct Singer
    {
        public long Id { get; set; }

        public string FirstName { get; set; }

        public string LastName { get; set; }
    }
}
```

Here's how to issue the query with a parameter and access the data:

``` text
dotnet run queryWithParameter $env:PROJECT_ID test-instance example-db
```

You should see the following result:

``` text
SingerId : 12 FirstName : Melissa LastName : Garcia
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner client library for C\#.

#### On the command line

Use the following [`  ALTER TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) command to add the new column to the table:

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget INT64'
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget BIGINT'
```

You should see:

``` text
Schema updating...done.
```

#### Use the Spanner client library for C\#

Use [`  CreateDdlCommand()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_CreateDdlCommand_System_String_System_String___) to modify the schema:

### GoogleSQL

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class AddColumnAsyncSample
{
    public async Task AddColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string alterStatement = "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64";

        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(alterStatement);
        await updateCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added the MarketingBudget column.");
    }
}
```

### PostgreSQL

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class AddColumnAsyncPostgresSample
{
    public async Task AddColumnAsyncPostgres(string projectId, string instanceId, string databaseId)
    {
        // PostgreSQL database with Singers table already exists.
        // Alter the Singers table.
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string alterStatement = "ALTER TABLE Singers ADD COLUMN Age INTEGER";

        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(alterStatement);
        await updateCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added the Age column in Singers table.");
    }
}
```

Run the sample using the `  addColumn  ` command.

``` text
dotnet run addColumn $env:PROJECT_ID test-instance example-db
```

You should see:

``` text
Added the MarketingBudget column.
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateDataAsyncSample
{
    public async Task<int> UpdateDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);

        var rowCount = 0;
        SpannerCommand cmd = connection.CreateDmlCommand(
            "UPDATE Albums SET MarketingBudget = @MarketingBudget "
            + "WHERE SingerId = 1 and AlbumId = 1");
        cmd.Parameters.Add("MarketingBudget", SpannerDbType.Int64, 100000);
        rowCount += await cmd.ExecuteNonQueryAsync();

        cmd = connection.CreateDmlCommand(
            "UPDATE Albums SET MarketingBudget = @MarketingBudget "
            + "WHERE SingerId = 2 and AlbumId = 2");
        cmd.Parameters.Add("MarketingBudget", SpannerDbType.Int64, 500000);
        rowCount += await cmd.ExecuteNonQueryAsync();

        Console.WriteLine("Data Updated.");
        return rowCount;
    }
}
```

Run the sample using the `  writeDataToNewColumn  ` command.

``` text
dotnet run writeDataToNewColumn $env:PROJECT_ID test-instance example-db
```

You should see:

``` text
Updated data.
```

You can also execute a SQL query to fetch the values that you just wrote.

Here's the code to execute the query:

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryNewColumnAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public long MarketingBudget { get; set; }
    }

    public async Task<List<Album>> QueryNewColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT * FROM Albums");
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                MarketingBudget = reader.IsDBNull(reader.GetOrdinal("MarketingBudget")) ? 0 : reader.GetFieldValue<long>("MarketingBudget")
            });
        }
        return albums;
    }
}
```

To execute this query, run the sample using the `  queryNewColumn  ` argument.

``` text
dotnet run queryNewColumn $env:PROJECT_ID test-instance example-db
```

You should see:

``` text
SingerId : 1 AlbumId : 1 MarketingBudget : 100000
SingerId : 1 AlbumId : 2 MarketingBudget :
SingerId : 2 AlbumId : 1 MarketingBudget :
SingerId : 2 AlbumId : 2 MarketingBudget : 500000
SingerId : 2 AlbumId : 3 MarketingBudget :
```

## Update data

You can update data using DML in a read-write transaction.

You use the `  ExecuteNonQueryAsync()  ` method to execute a DML statement.

### GoogleSQL

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class WriteWithTransactionUsingDmlCoreAsyncSample
{
    public async Task<int> WriteWithTransactionUsingDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        // This sample transfers 200,000 from the MarketingBudget
        // field of the second Album to the first Album. Make sure to run
        // the AddColumnAsyncSample and WriteDataToNewColumnAsyncSample first,
        // in that order.
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        decimal transferAmount = 200000;
        decimal secondBudget = 0;

        // Create connection to Cloud Spanner.
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        // Create a readwrite transaction that we'll assign
        // to each SpannerCommand.
        using var transaction = await connection.BeginTransactionAsync();
        // Create statement to select the second album's data.
        var cmdLookup = connection.CreateSelectCommand("SELECT * FROM Albums WHERE SingerId = 2 AND AlbumId = 2");
        cmdLookup.Transaction = transaction;
        // Execute the select query.
        using var reader1 = await cmdLookup.ExecuteReaderAsync();
        while (await reader1.ReadAsync())
        {
            // Read the second album's budget.
            secondBudget = reader1.GetFieldValue<decimal>("MarketingBudget");
            // Confirm second Album's budget is sufficient and
            // if not raise an exception. Raising an exception
            // will automatically roll back the transaction.
            if (secondBudget < transferAmount)
            {
                throw new Exception($"The second album's budget {secondBudget} is less than the amount to transfer.");
            }
        }

        // Update second album to remove the transfer amount.
        secondBudget -= transferAmount;
        SpannerCommand cmd = connection.CreateDmlCommand("UPDATE Albums SET MarketingBudget = @MarketingBudget  WHERE SingerId = 2 and AlbumId = 2");
        cmd.Parameters.Add("MarketingBudget", SpannerDbType.Int64, secondBudget);
        cmd.Transaction = transaction;
        var rowCount = await cmd.ExecuteNonQueryAsync();

        // Update first album to add the transfer amount.
        cmd = connection.CreateDmlCommand("UPDATE Albums SET MarketingBudget = MarketingBudget + @MarketingBudgetIncrement WHERE SingerId = 1 and AlbumId = 1");
        cmd.Parameters.Add("MarketingBudgetIncrement", SpannerDbType.Int64, transferAmount);
        cmd.Transaction = transaction;
        rowCount += await cmd.ExecuteNonQueryAsync();

        await transaction.CommitAsync();

        Console.WriteLine("Transaction complete.");
        return rowCount;
    }
}
```

### PostgreSQL

``` csharp
using Google.Cloud.Spanner.Data;
using Google.Cloud.Spanner.V1;
using System;
using System.Threading.Tasks;

public class UpdateUsingDmlAsyncPostgresSample
{
    public async Task<int> UpdateUsingDmlAsyncPostgres(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("UPDATE Singers SET Rating = $1 WHERE SingerId = 11", 
            new SpannerParameterCollection 
            {
                { "p1", SpannerDbType.PgNumeric, PgNumeric.Parse("4.0") }
            });

        var rowCount = await cmd.ExecuteNonQueryAsync();

        Console.WriteLine($"{rowCount} row(s) updated...");
        return rowCount;
    }
}
```

Run the sample using the `  writeWithTransactionUsingDml  ` argument.

``` text
dotnet run writeWithTransactionUsingDml $env:PROJECT_ID test-instance example-db
```

You should see:

``` text
Transaction complete.
```

**Note:** You can also [update data using mutations](/spanner/docs/modify-mutation-api#updating_rows_in_a_table) .

## Use a secondary index

Suppose you wanted to fetch all rows of `  Albums  ` that have `  AlbumTitle  ` values in a certain range. You could read all values from the `  AlbumTitle  ` column using a SQL statement or a read call, and then discard the rows that don't meet the criteria, but doing this full table scan is expensive, especially for tables with a lot of rows. Instead you can speed up the retrieval of rows when searching by non-primary key columns by creating a [secondary index](/spanner/docs/secondary-indexes) on the table.

Adding a secondary index to an existing table requires a schema update. Like other schema updates, Spanner supports adding an index while the database continues to serve traffic. Spanner automatically backfills the index with your existing data. Backfills might take a few minutes to complete, but you don't need to take the database offline or avoid writing to the indexed table during this process. For more details, see [Add a secondary index](/spanner/docs/secondary-indexes#adding_an_index) .

After you add a secondary index, Spanner automatically uses it for SQL queries that are likely to run faster with the index. If you use the read interface, you must specify the index that you want to use.

### Add a secondary index

You can add an index on the command line using the gcloud CLI or programmatically using the Spanner client library for C\#.

#### On the command line

Use the following [`  CREATE INDEX  `](/spanner/docs/reference/standard-sql/data-definition-language#create_index) command to add an index to the database:

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)'
```

You should see:

``` text
Schema updating...done.
```

#### Using the Spanner client library for C\#

Use [`  CreateDdlCommand()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_CreateDdlCommand_System_String_System_String___) to add an index:

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class AddIndexAsyncSample
{
    public async Task AddIndexAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string createStatement = "CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)";

        using var connection = new SpannerConnection(connectionString);
        using var createCmd = connection.CreateDdlCommand(createStatement);
        await createCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added the AlbumsByAlbumTitle index.");
    }
}
```

Run the sample using the `  addIndex  ` command.

``` text
  dotnet run addIndex $env:PROJECT_ID test-instance example-db
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
  Added the AlbumsByAlbumTitle index.
```

### Add an index for index-only reads

You might have noticed that the previous read example doesn't include reading the `  MarketingBudget  ` column. This is because Spanner's read interface doesn't support the ability to join an index with a data table to look up values that are not stored in the index.

Create an alternate definition of `  AlbumsByAlbumTitle  ` that stores a copy of `  MarketingBudget  ` in the index.

#### On the command line

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget)
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) INCLUDE (MarketingBudget)
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Schema updating...done.
```

#### Using the Spanner client library for C\#

Use [`  CreateDdlCommand()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_CreateDdlCommand_System_String_System_String___) to add an index with a `  STORING  ` clause:

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class AddStoringIndexAsyncSample
{
    public async Task AddStoringIndexAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string createStatement = "CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget)";

        using var connection = new SpannerConnection(connectionString);
        using var createCmd = connection.CreateDdlCommand(createStatement);
        await createCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added the AlbumsByAlbumTitle2 index.");
    }
}
```

Run the sample using the `  addStoringIndex  ` command.

``` text
dotnet run addStoringIndex $env:PROJECT_ID test-instance example-db
```

You should see:

``` text
Added the AlbumsByAlbumTitle2 index.
```

Now you can execute a read that fetches all `  AlbumId  ` , `  AlbumTitle  ` , and `  MarketingBudget  ` columns from the `  AlbumsByAlbumTitle2  ` index:

Read data using the storing index you created by executing a query that explicitly specifies the index:

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithStoringIndexAsyncSample
{
    public class Album
    {
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
        public long? MarketingBudget { get; set; }
    }

    public async Task<List<Album>> QueryDataWithStoringIndexAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        var cmd = connection.CreateSelectCommand(
            "SELECT AlbumId, AlbumTitle, MarketingBudget FROM Albums@ "
            + "{FORCE_INDEX=AlbumsByAlbumTitle2}");

        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album
            {
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                AlbumTitle = reader.GetFieldValue<string>("AlbumTitle"),
                MarketingBudget = reader.IsDBNull(reader.GetOrdinal("MarketingBudget")) ? 0 : reader.GetFieldValue<long>("MarketingBudget")
            });
        }
        return albums;
    }
}
```

Run the sample using the `  queryDataWithStoringIndex  ` command.

``` text
dotnet run queryDataWithStoringIndex $env:PROJECT_ID test-instance example-db
```

You should see output similar to:

``` text
AlbumId : 2 AlbumTitle : Forever Hold your Peace MarketingBudget : 300000
AlbumId : 2 AlbumTitle : Go, Go, Go MarketingBudget : 300000
```

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data. Use the .NET framework's [`  TransactionScope()  `](https://msdn.microsoft.com/en-us/library/system.transactions.transactionscope) along with [`  OpenAsReadOnlyAsync()  `](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/Google.Cloud.Spanner.Data.SpannerConnection#Google_Cloud_Spanner_Data_SpannerConnection_OpenAsReadOnlyAsync_Google_Cloud_Spanner_Data_TimestampBound_System_Threading_CancellationToken_) for executing read-only transactions.

The following shows how to run a query and perform a read in the same read-only transaction:

### .NET Standard 2.0

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Transactions;

public class QueryDataWithTransactionAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> QueryDataWithTransactionAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();
        using TransactionScope scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
        using var connection = new SpannerConnection(connectionString);

        // Opens the connection so that the Spanner transaction included in the TransactionScope
        // is read-only TimestampBound.Strong.
        await connection.OpenAsync(SpannerTransactionCreationOptions.ReadOnly, options: null, cancellationToken: default);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

        // Read #1.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine("SingerId : " + reader.GetFieldValue<string>("SingerId")
                    + " AlbumId : " + reader.GetFieldValue<string>("AlbumId")
                    + " AlbumTitle : " + reader.GetFieldValue<string>("AlbumTitle"));
            }
        }

        // Read #2. Even if changes occur in-between the reads,
        // the transaction ensures that Read #1 and Read #2
        // return the same data.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                albums.Add(new Album
                {
                    AlbumId = reader.GetFieldValue<int>("AlbumId"),
                    SingerId = reader.GetFieldValue<int>("SingerId"),
                    AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
                });
            }
        }
        scope.Complete();
        Console.WriteLine("Transaction complete.");
        return albums;
    }
}
```

### .NET Standard 1.5

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithTransactionCoreAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> QueryDataWithTransactionCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var albums = new List<Album>();

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        // Open a new read only transaction.
        using var transaction = await connection.BeginTransactionAsync(
            SpannerTransactionCreationOptions.ReadOnly,
            transactionOptions: null,
            cancellationToken: default);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
        cmd.Transaction = transaction;

        // Read #1.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine("SingerId : " + reader.GetFieldValue<string>("SingerId")
                    + " AlbumId : " + reader.GetFieldValue<string>("AlbumId")
                    + " AlbumTitle : " + reader.GetFieldValue<string>("AlbumTitle"));
            }
        }

        // Read #2. Even if changes occur in-between the reads,
        // the transaction ensures that Read #1 and Read #2
        // return the same data.
        using (var reader = await cmd.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                albums.Add(new Album
                {
                    AlbumId = reader.GetFieldValue<int>("AlbumId"),
                    SingerId = reader.GetFieldValue<int>("SingerId"),
                    AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
                });
            }
        }

        Console.WriteLine("Transaction complete.");
        return albums;
    }
}
```

Run the sample using the `  queryDataWithTransaction  ` command.

``` text
dotnet run queryDataWithTransaction $env:PROJECT_ID test-instance example-db
```

You should see output similar to:

``` text
SingerId : 2 AlbumId : 2 AlbumTitle : Forever Hold your Peace
SingerId : 1 AlbumId : 2 AlbumTitle : Go, Go, Go
SingerId : 2 AlbumId : 1 AlbumTitle : Green
SingerId : 2 AlbumId : 3 AlbumTitle : Terrified
SingerId : 1 AlbumId : 1 AlbumTitle : Total Junk
SingerId : 2 AlbumId : 2 AlbumTitle : Forever Hold your Peace
SingerId : 1 AlbumId : 2 AlbumTitle : Go, Go, Go
SingerId : 2 AlbumId : 1 AlbumTitle : Green
SingerId : 2 AlbumId : 3 AlbumTitle : Terrified
SingerId : 1 AlbumId : 1 AlbumTitle : Total Junk
```

## Cleanup

To avoid incurring additional charges to your Cloud Billing account for the resources used in this tutorial, drop the database and delete the instance that you created.

### Delete the database

If you delete an instance, all databases within it are automatically deleted. This step shows how to delete a database without deleting an instance (you would still incur charges for the instance).

#### On the command line

``` text
gcloud spanner databases delete example-db --instance=test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click the instance.

3.  Click the database that you want to delete.

4.  In the **Database details** page, click **Delete** .

5.  Confirm that you want to delete the database and click **Delete** .

### Delete the instance

Deleting an instance automatically drops all databases created in that instance.

#### On the command line

``` text
gcloud spanner instances delete test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click your instance.

3.  Click **Delete** .

4.  Confirm that you want to delete the instance and click **Delete** .

## What's next

  - Try the preview release of [Spanner database provider](https://github.com/cloudspannerecosystem/dotnet-spanner-entity-framework#googlecloudentityframeworkcorespanner) for [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/) .

<!-- end list -->

  - Learn how to [access Spanner with a virtual machine instance](/spanner/docs/configure-virtual-machine-instance) .

  - Learn about authorization and authentication credentials in [Authenticate to Cloud services using client libraries](/docs/authentication/getting-started) .

  - Learn more about Spanner [Schema design best practices](/spanner/docs/schema-design) .
