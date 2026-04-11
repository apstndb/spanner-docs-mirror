## Objectives

This tutorial walks you through the following steps using the Spanner ADO.NET driver:

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

Complete the steps described in [Set up](https://docs.cloud.google.com/spanner/docs/getting-started/set-up#set_up_a_project) , which cover creating and setting a default Google Cloud project, enabling billing, enabling the Cloud Spanner API, and setting up OAuth 2.0 to get authentication credentials to use the Cloud Spanner API.

In particular, make sure that you run [`gcloud auth application-default login`](https://docs.cloud.google.com/sdk/gcloud/reference/auth/application-default/login) to set up your local development environment with authentication credentials.

**Note:** If you don't plan to keep the resources that you create in this tutorial, consider creating a new Google Cloud project instead of selecting an existing project. After you finish the tutorial, you can delete the project, removing all resources associated with the project.

## Prepare your local ADO.NET environment

1.  Download and install [.NET](https://dotnet.microsoft.com/en-us/download) on your development machine if it isn't already installed.

2.  Clone the sample repository to your local machine:
    
        git clone https://github.com/googleapis/dotnet-spanner-entity-framework.git

3.  Change to the directory that contains the Spanner ADO.NET driver sample code:
    
        cd dotnet-spanner-entity-framework/spanner-ado-net/spanner-ado-net-getting-started-guide

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](https://docs.cloud.google.com/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `test-instance` to use it with other topics in this document that reference an instance named `test-instance` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with ADO.NET.

Take a look through the `SampleRunner.cs` file, which shows how to use Spanner. The code shows how to create and use a new database. The data uses the example schema shown in the [Schema and data model](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

### GoogleSQL

    gcloud spanner databases create example-db --instance=test-instance

### PostgreSQL

    gcloud spanner databases create example-db --instance=test-instance \
      --database-dialect=POSTGRESQL

You should see:

    Creating database...done.

### Create tables

The following code creates two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

    public static async Task CreateTables(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Create two tables in one batch on Spanner.
        var batch = connection.CreateBatch();
        batch.BatchCommands.Add("CREATE TABLE Singers (" +
                                "  SingerId   INT64 NOT NULL, " +
                                "  FirstName  STRING(1024), " +
                                "  LastName   STRING(1024), " +
                                "  SingerInfo BYTES(MAX) " +
                                ") PRIMARY KEY (SingerId)");
        batch.BatchCommands.Add("CREATE TABLE Albums ( " +
                                "  SingerId     INT64 NOT NULL, " +
                                "  AlbumId      INT64 NOT NULL, " +
                                "  AlbumTitle   STRING(MAX)" +
                                ") PRIMARY KEY (SingerId, AlbumId), " +
                                "INTERLEAVE IN PARENT Singers ON DELETE CASCADE");
        await batch.ExecuteNonQueryAsync();
        Console.WriteLine("Created Singers & Albums tables");
    }

### PostgreSQL

    public static async Task CreateTables(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Create two tables in one batch on Spanner.
        var batch = connection.CreateBatch();
        batch.BatchCommands.Add("create table singers (" +
                                "  singer_id   bigint not null primary key, " +
                                "  first_name  varchar(1024), " +
                                "  last_name   varchar(1024), " +
                                "  singer_info bytea" +
                                ")");
        batch.BatchCommands.Add("create table albums (" +
                                "  singer_id     bigint not null, " +
                                "  album_id      bigint not null, " +
                                "  album_title   varchar, " +
                                "  primary key (singer_id, album_id)" +
                                ") interleave in parent singers on delete cascade");
        await batch.ExecuteNonQueryAsync();
        Console.WriteLine("Created Singers & Albums tables");
    }

Run the sample with the following command:

### GoogleSQL

    dotnet run createtables projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run createtablespg projects/PROJECT_ID/instances/test-instance/databases/example-db

The next step is to write data to your database.

## Create a connection

Before you can do reads or writes, you must create a connection to interact with Spanner. The database name and other connection properties are specified in the ADO.NET connection string.

### GoogleSQL

    /// <summary>
    /// Create an ADO.NET connection to a Spanner database.
    /// </summary>
    /// <param name="connectionString">
    /// A connection string in the format
    /// 'Data Source=projects/my-project/instances/my-instance/databases/my-database'.
    /// </param>
    public static async Task CreateConnection(string connectionString)
    {
        // Use a SpannerConnectionStringBuilder to construct a connection string.
        // The SpannerConnectionStringBuilder contains properties for the most
        // used connection string variables.
        var builder = new SpannerConnectionStringBuilder(connectionString)
        {
            // Sets the default isolation level that should be used for all
            // read/write transactions on this connection.
            DefaultIsolationLevel = IsolationLevel.RepeatableRead,
    
            // The Options property can be used to set any connection property
            // as a key-value pair.
            Options = "statement_cache_size=2000"
        };
    
        await using var connection = new SpannerConnection(builder.ConnectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT 'Hello World' as Message";
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"Greeting from Spanner: {reader.GetString(0)}");
        }
    }

### PostgreSQL

    /// <summary>
    /// Create an ADO.NET connection to a Spanner PostgreSQL database.
    /// </summary>
    /// <param name="connectionString">
    /// A connection string in the format
    /// 'Data Source=projects/my-project/instances/my-instance/databases/my-database'.
    /// </param>
    public static async Task CreateConnection(string connectionString)
    {
        // Use a SpannerConnectionStringBuilder to construct a connection string.
        // The SpannerConnectionStringBuilder contains properties for the most
        // used connection string variables.
        var builder = new SpannerConnectionStringBuilder(connectionString)
        {
            // Sets the default isolation level that should be used for all
            // read/write transactions on this connection.
            DefaultIsolationLevel = IsolationLevel.RepeatableRead,
    
            // The Options property can be used to set any connection property
            // as a key-value pair.
            Options = "statement_cache_size=2000"
        };
    
        await using var connection = new SpannerConnection(builder.ConnectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT 'Hello World' as Message";
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"Greeting from Spanner: {reader.GetString(0)}");
        }
    }

<span id="write_data"></span>

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `DbCommand#ExecuteNonQuery` method to execute a DML statement.

### GoogleSQL

    public static async Task WriteDataWithDml(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Add 4 rows in one statement.
        // The ADO.NET driver supports positional query parameters.
        await using var command = connection.CreateCommand();
        command.CommandText = "INSERT INTO Singers (SingerId, FirstName, LastName) " +
                              "VALUES (?, ?, ?), (?, ?, ?), " +
                              "       (?, ?, ?), (?, ?, ?)";
        command.Parameters.Add(12);
        command.Parameters.Add("Melissa");
        command.Parameters.Add("Garcia");
    
        command.Parameters.Add(13);
        command.Parameters.Add("Russel");
        command.Parameters.Add("Morales");
    
        command.Parameters.Add(14);
        command.Parameters.Add("Jacqueline");
        command.Parameters.Add("Long");
    
        command.Parameters.Add(15);
        command.Parameters.Add("Dylan");
        command.Parameters.Add("Shaw");
    
        var affected = await command.ExecuteNonQueryAsync();
        Console.WriteLine($"{affected} record(s) inserted.");
    }

### PostgreSQL

    public static async Task WriteDataWithDml(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Add 4 rows in one statement.
        // The ADO.NET driver supports positional query parameters.
        await using var command = connection.CreateCommand();
        command.CommandText = "insert into singers (singer_id, first_name, last_name) " +
                              "VALUES (?, ?, ?), (?, ?, ?), " +
                              "       (?, ?, ?), (?, ?, ?)";
        command.Parameters.Add(12);
        command.Parameters.Add("Melissa");
        command.Parameters.Add("Garcia");
    
        command.Parameters.Add(13);
        command.Parameters.Add("Russel");
        command.Parameters.Add("Morales");
    
        command.Parameters.Add(14);
        command.Parameters.Add("Jacqueline");
        command.Parameters.Add("Long");
    
        command.Parameters.Add(15);
        command.Parameters.Add("Dylan");
        command.Parameters.Add("Shaw");
    
        var affected = await command.ExecuteNonQueryAsync();
        Console.WriteLine($"{affected} record(s) inserted.");
    }

Run the sample with the following command:

### GoogleSQL

    dotnet run dmlwrite projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run dmlwritepg projects/PROJECT_ID/instances/test-instance/databases/example-db

The result should show:

    4 records inserted.

**Note:** There are limits to commit size. See [CRUD limit](https://docs.cloud.google.com/spanner/quotas#limits-for) for more information.

<span id="write_data_with_mutations"></span>

## Write data with mutations

You can also insert data using [mutations](https://docs.cloud.google.com/spanner/docs/modify-mutation-api) .

You can insert data using the `batch.CreateInsertCommand()` method, which creates a new `SpannerBatchCommand` to insert rows into a table. The `SpannerBatchCommand.ExecuteNonQueryAsync()` method adds new rows to the table.

The following code shows how to write data using mutations:

### GoogleSQL

    struct Singer
    {
        internal long SingerId;
        internal string FirstName;
        internal string LastName;
    }
    
    struct Album
    {
        internal long SingerId;
        internal long AlbumId;
        internal string Title;
    }
    
    public static async Task WriteDataWithMutations(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        Singer[] singers =
        [
            new() {SingerId=1, FirstName = "Marc", LastName = "Richards"},
            new() {SingerId=2, FirstName = "Catalina", LastName = "Smith"},
            new() {SingerId=3, FirstName = "Alice", LastName = "Trentor"},
            new() {SingerId=4, FirstName = "Lea", LastName = "Martin"},
            new() {SingerId=5, FirstName = "David", LastName = "Lomond"},
        ];
        Album[] albums =
        [
            new() {SingerId = 1, AlbumId = 1, Title = "Total Junk"},
            new() {SingerId = 1, AlbumId = 2, Title = "Go, Go, Go"},
            new() {SingerId = 2, AlbumId = 1, Title = "Green"},
            new() {SingerId = 2, AlbumId = 2, Title = "Forever Hold Your Peace"},
            new() {SingerId = 2, AlbumId = 3, Title = "Terrified"},
        ];
        var batch = connection.CreateBatch();
        foreach (var singer in singers)
        {
            // The name of a parameter must correspond with a column name.
            var command = batch.CreateInsertCommand("Singers");
            command.AddParameter("SingerId", singer.SingerId);
            command.AddParameter("FirstName", singer.FirstName);
            command.AddParameter("LastName", singer.LastName);
            batch.BatchCommands.Add(command);
        }
        foreach (var album in albums)
        {
            // The name of a parameter must correspond with a column name.
            var command = batch.CreateInsertCommand("Albums");
            command.AddParameter("SingerId", album.SingerId);
            command.AddParameter("AlbumId", album.AlbumId);
            command.AddParameter("AlbumTitle", album.Title);
            batch.BatchCommands.Add(command);
        }
        var affected = await batch.ExecuteNonQueryAsync();
        Console.WriteLine($"Inserted {affected} rows.");
    }

### PostgreSQL

    struct Singer
    {
        internal long SingerId;
        internal string FirstName;
        internal string LastName;
    }
    
    struct Album
    {
        internal long SingerId;
        internal long AlbumId;
        internal string Title;
    }
    
    public static async Task WriteDataWithMutations(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        Singer[] singers =
        [
            new() {SingerId=1, FirstName = "Marc", LastName = "Richards"},
            new() {SingerId=2, FirstName = "Catalina", LastName = "Smith"},
            new() {SingerId=3, FirstName = "Alice", LastName = "Trentor"},
            new() {SingerId=4, FirstName = "Lea", LastName = "Martin"},
            new() {SingerId=5, FirstName = "David", LastName = "Lomond"},
        ];
        Album[] albums =
        [
            new() {SingerId = 1, AlbumId = 1, Title = "Total Junk"},
            new() {SingerId = 1, AlbumId = 2, Title = "Go, Go, Go"},
            new() {SingerId = 2, AlbumId = 1, Title = "Green"},
            new() {SingerId = 2, AlbumId = 2, Title = "Forever Hold Your Peace"},
            new() {SingerId = 2, AlbumId = 3, Title = "Terrified"},
        ];
        var batch = connection.CreateBatch();
        foreach (var singer in singers)
        {
            // The name of a parameter must correspond with a column name.
            var command = batch.CreateInsertCommand("singers");
            command.AddParameter("singer_id", singer.SingerId);
            command.AddParameter("first_name", singer.FirstName);
            command.AddParameter("last_name", singer.LastName);
            batch.BatchCommands.Add(command);
        }
        foreach (var album in albums)
        {
            // The name of a parameter must correspond with a column name.
            var command = batch.CreateInsertCommand("albums");
            command.AddParameter("singer_id", album.SingerId);
            command.AddParameter("album_id", album.AlbumId);
            command.AddParameter("album_title", album.Title);
            batch.BatchCommands.Add(command);
        }
        var affected = await batch.ExecuteNonQueryAsync();
        Console.WriteLine($"Inserted {affected} rows.");
    }

Run the following example using the `write` argument:

### GoogleSQL

    dotnet run write projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run writepg projects/PROJECT_ID/instances/test-instance/databases/example-db

**Note:** There are limits to commit size. See [CRUD limit](https://docs.cloud.google.com/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner ADO.NET driver.

### On the command line

Execute the following SQL statement to read the values of all columns from the `Albums` table:

### GoogleSQL

    gcloud spanner databases execute-sql example-db --instance=test-instance \
        --sql='SELECT SingerId, AlbumId, AlbumTitle FROM Albums'

### PostgreSQL

    gcloud spanner databases execute-sql example-db --instance=test-instance \
        --sql='SELECT singer_id, album_id, album_title FROM albums'

**Note:** For the GoogleSQL reference, see [Query syntax in GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax) and for PostgreSQL reference, see [PostgreSQL lexical structure and syntax](https://docs.cloud.google.com/spanner/docs/reference/postgresql/lexical) .

The result shows:

    SingerId AlbumId AlbumTitle
    1        1       Total Junk
    1        2       Go, Go, Go
    2        1       Green
    2        2       Forever Hold Your Peace
    2        3       Terrified

### Use the Spanner ADO.NET driver

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner ADO.NET driver.

The following methods are used to execute a SQL query:

  - The [`ExecuteReader`](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/retrieving-data-using-a-datareader) method in the `DbCommand` class: use this to execute a SQL statement that returns rows, such as a query or a DML statement with a `THEN RETURN` clause.
  - The [`DbDataReader`](https://learn.microsoft.com/en-us/dotnet/api/system.data.common.dbdatareader) class: use this to access the data returned by a SQL statement.

The following example uses the `ExecuteReaderAsync` method:

### GoogleSQL

    public static async Task QueryData(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT SingerId, AlbumId, AlbumTitle " +
                              "FROM Albums " +
                              "ORDER BY SingerId, AlbumId";
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"{reader["SingerId"]} {reader["AlbumId"]} {reader["AlbumTitle"]}");
        }
    }

### PostgreSQL

    public static async Task QueryData(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT singer_id, album_id, album_title " +
                              "FROM albums " +
                              "ORDER BY singer_id, album_id";
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"{reader["singer_id"]} {reader["album_id"]} {reader["album_title"]}");
        }
    }

Run the example with the following command:

### GoogleSQL

    dotnet run query projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run querypg projects/PROJECT_ID/instances/test-instance/databases/example-db

The result should show:

    1 1 Total Junk
    1 2 Go, Go, Go
    2 1 Green
    2 2 Forever Hold Your Peace
    2 3 Terrified

### Query using a SQL parameter

If your application has a frequently executed query, you can improve its performance by parameterizing it. The resulting parametric query can be cached and reused, which reduces compilation costs. For more information, see [Use query parameters to speed up frequently executed queries](https://docs.cloud.google.com/spanner/docs/sql-best-practices#query-parameters) .

Here is an example of using a parameter in the `WHERE` clause to query records containing a specific value for `LastName` .

The Spanner ADO.NET driver supports both positional and named query parameters. A `?` in a SQL statement indicates a positional query parameter. Add query parameter values to the `Parameters` of the `DbCommand` . For example:

### GoogleSQL

    public static async Task QueryDataWithParameter(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT SingerId, FirstName, LastName " +
                              "FROM Singers " +
                              "WHERE LastName = ?";
        command.Parameters.Add("Garcia");
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"{reader["SingerId"]} {reader["FirstName"]} {reader["LastName"]}");
        }
    }

### PostgreSQL

    public static async Task QueryDataWithParameter(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT singer_id, first_name, last_name " +
                              "FROM singers " +
                              "WHERE last_name = ?";
        command.Parameters.Add("Garcia");
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"{reader["singer_id"]} {reader["first_name"]} {reader["last_name"]}");
        }
    }

Run the example with the following command:

### GoogleSQL

    dotnet run querywithparameter projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run querywithparameterpg projects/PROJECT_ID/instances/test-instance/databases/example-db

The result shows:

    12 Melissa Garcia

## Update the database schema

Assume you need to add a new column called `MarketingBudget` to the `Albums` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](https://docs.cloud.google.com/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner ADO.NET driver.

#### On the command line

Use the following [`ALTER TABLE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#alter_table) command to add the new column to the table:

### GoogleSQL

    gcloud spanner databases ddl update example-db --instance=test-instance \
        --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget INT64'

### PostgreSQL

    gcloud spanner databases ddl update example-db --instance=test-instance \
        --ddl='alter table albums add column marketing_budget bigint'

You should see:

    Schema updating...done.

#### Use the Spanner ADO.NET driver

Use the `ExecuteNonQueryAsync` method to modify the schema:

### GoogleSQL

    public static async Task AddColumn(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64";
        await command.ExecuteNonQueryAsync();
    
        Console.WriteLine("Added MarketingBudget column");
    }

### PostgreSQL

    public static async Task AddColumn(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "alter table albums add column marketing_budget bigint";
        await command.ExecuteNonQueryAsync();
    
        Console.WriteLine("Added marketing_budget column");
    }

Run the example with the following command:

### GoogleSQL

    dotnet run addcolumn projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run addcolumnpg projects/PROJECT_ID/instances/test-instance/databases/example-db

The result shows:

    Added MarketingBudget column.

### Execute a DDL batch

We recommend that you execute multiple schema modifications in one batch. Use the ADO.NET `CreateBatch` method to create a batch. The following example creates two tables in one batch:

### GoogleSQL

    public static async Task DdlBatch(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Executing multiple DDL statements as one batch is
        // more efficient than executing each statement individually.
        var batch = connection.CreateBatch();
        batch.BatchCommands.Add(
            "CREATE TABLE Venues (" +
            "  VenueId     INT64 NOT NULL, " +
            "  Name        STRING(1024), " +
            "  Description JSON, " +
            ") PRIMARY KEY (VenueId)");
        batch.BatchCommands.Add(
            "CREATE TABLE Concerts (" +
            "  ConcertId INT64 NOT NULL, " +
            "  VenueId   INT64 NOT NULL, " +
            "  SingerId  INT64 NOT NULL, " +
            "  StartTime TIMESTAMP, " +
            "  EndTime   TIMESTAMP, " +
            "  CONSTRAINT Fk_Concerts_Venues " +
            "    FOREIGN KEY (VenueId) REFERENCES Venues (VenueId), " +
            "  CONSTRAINT Fk_Concerts_Singers " +
            "    FOREIGN KEY (SingerId) REFERENCES Singers (SingerId), " +
            ") PRIMARY KEY (ConcertId)");
        await batch.ExecuteNonQueryAsync();
    
        Console.WriteLine("Added Venues and Concerts tables");
    }

### PostgreSQL

    public static async Task DdlBatch(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Executing multiple DDL statements as one batch is
        // more efficient than executing each statement individually.
        var batch = connection.CreateBatch();
        batch.BatchCommands.Add(
            "create table venues (" +
            "  venue_id    bigint not null primary key, " +
            "  name        varchar(1024), " +
            "  description jsonb" +
            ")");
        batch.BatchCommands.Add(
            "create table concerts (" +
            "  concert_id bigint not null primary key, " +
            "  venue_id   bigint not null, " +
            "  singer_id  bigint not null, " +
            "  start_time timestamptz, " +
            "  end_time   timestamptz, " +
            "  constraint fk_concerts_venues foreign key " +
            "    (venue_id) references venues (venue_id), " +
            "  constraint fk_concerts_singers foreign key " +
            "    (singer_id) references singers (singer_id)" +
            ")");
        await batch.ExecuteNonQueryAsync();
    
        Console.WriteLine("Added Venues and Concerts tables");
    }

Run the example with the following command:

### GoogleSQL

    dotnet run ddlbatch projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run ddlbatchpg projects/PROJECT_ID/instances/test-instance/databases/example-db

The result shows:

    Added Venues and Concerts tables.

### Write data to the new column

The following code writes data to the new column. It sets `MarketingBudget` to `100000` for the row keyed by `Albums(1, 1)` and to `500000` for the row keyed by `Albums(2, 2)` .

### GoogleSQL

    public static async Task UpdateDataWithMutations(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        (long SingerId, long AlbumId, long MarketingBudget)[] albums = [
            (1L, 1L, 100000L),
            (2L, 2L, 500000L),
        ];
        // Use a batch to update two rows in one round-trip.
        var batch = connection.CreateBatch();
        foreach (var album in albums)
        {
            // This creates a command that will use a mutation to update the row.
            var command = batch.CreateUpdateCommand("Albums");
            command.AddParameter("SingerId", album.SingerId);
            command.AddParameter("AlbumId", album.AlbumId);
            command.AddParameter("MarketingBudget", album.MarketingBudget);
            batch.BatchCommands.Add(command);
        }
        var affected = await batch.ExecuteNonQueryAsync();
        Console.WriteLine($"Updated {affected} albums.");
    }

### PostgreSQL

    public static async Task UpdateDataWithMutations(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        (long SingerId, long AlbumId, long MarketingBudget)[] albums = [
            (1L, 1L, 100000L),
            (2L, 2L, 500000L),
        ];
        // Use a batch to update two rows in one round-trip.
        var batch = connection.CreateBatch();
        foreach (var album in albums)
        {
            // This creates a command that will use a mutation to update the row.
            var command = batch.CreateUpdateCommand("albums");
            command.AddParameter("singer_id", album.SingerId);
            command.AddParameter("album_id", album.AlbumId);
            command.AddParameter("marketing_budget", album.MarketingBudget);
            batch.BatchCommands.Add(command);
        }
        var affected = await batch.ExecuteNonQueryAsync();
        Console.WriteLine($"Updated {affected} albums.");
    }

Run the example with the following command:

### GoogleSQL

    dotnet run update projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run updatepg projects/PROJECT_ID/instances/test-instance/databases/example-db

The result shows:

    Updated 2 albums

You can also execute a SQL query to fetch the values that you just wrote.

The following example uses the `ExecuteReaderAsync` method to execute a query:

### GoogleSQL

    public static async Task QueryNewColumn(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT SingerId, AlbumId, MarketingBudget " +
                              "FROM Albums " +
                              "ORDER BY SingerId, AlbumId";
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"{reader["SingerId"]} {reader["AlbumId"]} {reader["MarketingBudget"]}");
        }
    }

### PostgreSQL

    public static async Task QueryNewColumn(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "select singer_id, album_id, marketing_budget " +
                              "from albums " +
                              "order by singer_id, album_id";
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"{reader["singer_id"]} {reader["album_id"]} {reader["marketing_budget"]}");
        }
    }

To execute this query, run the following command:

### GoogleSQL

    dotnet run querymarketingbudget projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run querymarketingbudgetpg projects/PROJECT_ID/instances/test-instance/databases/example-db

You should see:

    1 1 100000
    1 2 null
    2 1 null
    2 2 500000
    2 3 null

## Update data

You can update data using DML in a read-write transaction.

Call `connection.BeginTransactionAsync()` to execute read-write transactions in ADO.NET.

### GoogleSQL

    public static async Task WriteDataWithTransaction(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Transfer marketing budget from one album to another. We do it in a
        // transaction to ensure that the transfer is atomic.
        await using var transaction = await connection.BeginTransactionAsync();
    
        // The Spanner ADO.NET driver supports both positional and named
        // query parameters. This query uses named query parameters.
        const string selectSql =
            "SELECT MarketingBudget " +
            "FROM Albums " +
            "WHERE SingerId = @singerId and AlbumId = @albumId";
        // Get the marketing_budget of singer 2 / album 2.
        await using var command = connection.CreateCommand();
        command.CommandText = selectSql;
        command.Transaction = transaction;
        command.Parameters.AddWithValue("singerId", 2);
        command.Parameters.AddWithValue("albumId", 2);
        var budget2 = (long) (await command.ExecuteScalarAsync() ?? 0L);
    
        const long transfer = 20000L;
        // The transaction will only be committed if this condition still holds
        // at the time of commit. Otherwise, the transaction will be aborted.
        if (budget2 >= transfer)
        {
            // Get the marketing_budget of singer 1 / album 1.
            command.Parameters["singerId"].Value = 1;
            command.Parameters["albumId"].Value = 1;
            var budget1 = (long) (await command.ExecuteScalarAsync() ?? 0L);
    
            // Transfer part of the marketing budget of Album 2 to Album 1.
            budget1 += transfer;
            budget2 -= transfer;
            const string updateSql =
                "UPDATE Albums " +
                "SET MarketingBudget = @budget " +
                "WHERE SingerId = @singerId and AlbumId = @albumId";
            // Create a DML batch and execute it as part of the current transaction.
            var batch = connection.CreateBatch();
            batch.Transaction = transaction;
    
            // Update the marketing budgets of both Album 1 and Album 2 in a batch.
            (long SingerId, long AlbumId, long MarketingBudget)[] budgets = [
                new (1L, 1L, budget1),
                new (2L, 2L, budget2),
            ];
            foreach (var budget in budgets)
            {
                var batchCommand = batch.CreateBatchCommand();
                batchCommand.CommandText = updateSql;
                var singerIdParameter = batchCommand.CreateParameter();
                singerIdParameter.ParameterName = "singerId";
                singerIdParameter.Value = budget.SingerId;
                batchCommand.Parameters.Add(singerIdParameter);
                var albumIdParameter = batchCommand.CreateParameter();
                albumIdParameter.ParameterName = "albumId";
                albumIdParameter.Value = budget.AlbumId;
                batchCommand.Parameters.Add(albumIdParameter);
                var marketingBudgetParameter = batchCommand.CreateParameter();
                marketingBudgetParameter.ParameterName = "budget";
                marketingBudgetParameter.Value = budget.MarketingBudget;
                batchCommand.Parameters.Add(marketingBudgetParameter);
                batch.BatchCommands.Add(batchCommand);
            }
            var affected = await batch.ExecuteNonQueryAsync();
            // The batch should update 2 rows.
            if (affected != 2)
            {
                await transaction.RollbackAsync();
                throw new InvalidOperationException($"Unexpected num affected: {affected}");
            }
        }
        // Commit the transaction.
        await transaction.CommitAsync();
        Console.WriteLine("Transferred marketing budget from Album 2 to Album 1");
    }

### PostgreSQL

    public static async Task WriteDataWithTransaction(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Transfer marketing budget from one album to another. We do it in a
        // transaction to ensure that the transfer is atomic.
        await using var transaction = await connection.BeginTransactionAsync();
    
        // The Spanner ADO.NET driver supports both positional and named
        // query parameters. This query uses named query parameters.
        const string selectSql =
            "SELECT marketing_budget " +
            "FROM albums " +
            "WHERE singer_id = $1 and album_id = $2";
        // Get the marketing_budget of singer 2 / album 2.
        await using var command = connection.CreateCommand();
        command.CommandText = selectSql;
        command.Transaction = transaction;
        command.Parameters.AddWithValue("p1", 2);
        command.Parameters.AddWithValue("p2", 2);
        var budget2 = (long) (await command.ExecuteScalarAsync() ?? 0L);
    
        const long transfer = 20000L;
        // The transaction will only be committed if this condition still holds
        // at the time of commit. Otherwise, the transaction will be aborted.
        if (budget2 >= transfer)
        {
            // Get the marketing_budget of singer 1 / album 1.
            command.Parameters["p1"].Value = 1;
            command.Parameters["p2"].Value = 1;
            var budget1 = (long) (await command.ExecuteScalarAsync() ?? 0L);
    
            // Transfer part of the marketing budget of Album 2 to Album 1.
            budget1 += transfer;
            budget2 -= transfer;
            const string updateSql =
                "UPDATE albums " +
                "SET marketing_budget = $1 " +
                "WHERE singer_id = $2 and album_id = $3";
            // Create a DML batch and execute it as part of the current transaction.
            var batch = connection.CreateBatch();
            batch.Transaction = transaction;
    
            // Update the marketing budgets of both Album 1 and Album 2 in a batch.
            (long SingerId, long AlbumId, long MarketingBudget)[] budgets = [
                new (1L, 1L, budget1),
                new (2L, 2L, budget2),
            ];
            foreach (var budget in budgets)
            {
                var batchCommand = batch.CreateBatchCommand();
                batchCommand.CommandText = updateSql;
                var marketingBudgetParameter = batchCommand.CreateParameter();
                marketingBudgetParameter.ParameterName = "p1";
                marketingBudgetParameter.Value = budget.MarketingBudget;
                batchCommand.Parameters.Add(marketingBudgetParameter);
                var singerIdParameter = batchCommand.CreateParameter();
                singerIdParameter.ParameterName = "p2";
                singerIdParameter.Value = budget.SingerId;
                batchCommand.Parameters.Add(singerIdParameter);
                var albumIdParameter = batchCommand.CreateParameter();
                albumIdParameter.ParameterName = "p3";
                albumIdParameter.Value = budget.AlbumId;
                batchCommand.Parameters.Add(albumIdParameter);
                batch.BatchCommands.Add(batchCommand);
            }
            var affected = await batch.ExecuteNonQueryAsync();
            // The batch should update 2 rows.
            if (affected != 2)
            {
                await transaction.RollbackAsync();
                throw new InvalidOperationException($"Unexpected num affected: {affected}");
            }
        }
        // Commit the transaction.
        await transaction.CommitAsync();
        Console.WriteLine("Transferred marketing budget from Album 2 to Album 1");
    }

Run the example with the following command:

### GoogleSQL

    dotnet run writewithtransactionusingdml projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run writewithtransactionusingdmlpg projects/PROJECT_ID/instances/test-instance/databases/example-db

### Transaction tags and request tags

Use [transaction tags and request tags](https://docs.cloud.google.com/spanner/docs/introspection/troubleshooting-with-tags) to troubleshoot transactions and queries in Spanner. You can set tags on Transaction objects to send transaction tags, and DbCommand objects to send request tags to Spanner. For example:

### GoogleSQL

    public static async Task Tags(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        const long singerId = 1L;
        const long albumId = 1L;
    
        await using var transaction = await connection.BeginTransactionAsync();
        // Set a tag on the transaction before executing any statements.
        transaction.Tag = "example-tx-tag";
    
        await using var command = connection.CreateCommand();
        command.Transaction = transaction;
        command.Tag = "query-marketing-budget";
        command.CommandText =
            "SELECT MarketingBudget " +
            "FROM Albums " +
            "WHERE SingerId=? and AlbumId=?";
        command.Parameters.Add(singerId);
        command.Parameters.Add(albumId);
        var budget = (long)(await command.ExecuteScalarAsync() ?? 0L);
    
        // Reduce the marketing budget by 10% if it is more than 1,000.
        if (budget > 1000)
        {
            budget -= budget / 10;
            await using var updateCommand = connection.CreateCommand();
            updateCommand.Transaction = transaction;
            updateCommand.Tag = "reduce-marketing-budget";
            updateCommand.CommandText =
                "UPDATE Albums SET MarketingBudget=@budget WHERE SingerId=@singerId AND AlbumId=@albumId";
            updateCommand.Parameters.AddWithValue("budget", budget);
            updateCommand.Parameters.AddWithValue("singerId", singerId);
            updateCommand.Parameters.AddWithValue("albumId", albumId);
            await updateCommand.ExecuteNonQueryAsync();
        }
        // Commit the transaction.
        await transaction.CommitAsync();
        Console.WriteLine("Reduced marketing budget");
    }

### PostgreSQL

    public static async Task Tags(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        const long singerId = 1L;
        const long albumId = 1L;
    
        await using var transaction = await connection.BeginTransactionAsync();
        // Set a tag on the transaction before executing any statements.
        transaction.Tag = "example-tx-tag";
    
        await using var command = connection.CreateCommand();
        command.Transaction = transaction;
        command.Tag = "query-marketing-budget";
        command.CommandText =
            "select marketing_budget " +
            "from albums " +
            "where singer_id=? and album_id=?";
        command.Parameters.Add(singerId);
        command.Parameters.Add(albumId);
        var budget = (long)(await command.ExecuteScalarAsync() ?? 0L);
    
        // Reduce the marketing budget by 10% if it is more than 1,000.
        if (budget > 1000)
        {
            budget -= budget / 10;
            await using var updateCommand = connection.CreateCommand();
            updateCommand.Transaction = transaction;
            updateCommand.Tag = "reduce-marketing-budget";
            updateCommand.CommandText =
                "update albums set marketing_budget=$1 where singer_id=$2 and album_id=$3";
            updateCommand.Parameters.Add(budget);
            updateCommand.Parameters.Add(singerId);
            updateCommand.Parameters.Add(albumId);
            await updateCommand.ExecuteNonQueryAsync();
        }
        // Commit the transaction.
        await transaction.CommitAsync();
        Console.WriteLine("Reduced marketing budget");
    }

Run the example with the following command:

### GoogleSQL

    dotnet run tags projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run tagspg projects/PROJECT_ID/instances/test-instance/databases/example-db

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](https://docs.cloud.google.com/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data. Call `connection.BeginReadOnlyTransactionAsync()` to execute a read-only transaction.

The following shows how to run a query and perform a read in the same read-only transaction:

### GoogleSQL

    public static async Task ReadOnlyTransaction(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Start a read-only transaction on this connection.
        await using var transaction = await connection.BeginReadOnlyTransactionAsync();
    
        await using var command = connection.CreateCommand();
        command.Transaction = transaction;
        command.CommandText = "SELECT SingerId, AlbumId, AlbumTitle " +
                              "FROM Albums " +
                              "ORDER BY SingerId, AlbumId";
        await using (var reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine(
                    $"{reader["SingerId"]} {reader["AlbumId"]} {reader["AlbumTitle"]}");
            }
        }
    
        // Execute another query using the same read-only transaction.
        command.CommandText = "SELECT SingerId, AlbumId, AlbumTitle " +
                              "FROM Albums " +
                              "ORDER BY AlbumTitle";
        await using (var reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine(
                    $"{reader["SingerId"]} {reader["AlbumId"]} {reader["AlbumTitle"]}");
            }
        }
    
        // End the read-only transaction by calling Commit.
        await transaction.CommitAsync();
    }

### PostgreSQL

    public static async Task ReadOnlyTransaction(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Start a read-only transaction on this connection.
        await using var transaction = await connection.BeginReadOnlyTransactionAsync();
    
        await using var command = connection.CreateCommand();
        command.Transaction = transaction;
        command.CommandText = "SELECT SingerId, AlbumId, AlbumTitle " +
                              "FROM Albums " +
                              "ORDER BY SingerId, AlbumId";
        await using (var reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine(
                    $"{reader["SingerId"]} {reader["AlbumId"]} {reader["AlbumTitle"]}");
            }
        }
    
        // Execute another query using the same read-only transaction.
        command.CommandText = "SELECT SingerId, AlbumId, AlbumTitle " +
                              "FROM Albums " +
                              "ORDER BY AlbumTitle";
        await using (var reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine(
                    $"{reader["SingerId"]} {reader["AlbumId"]} {reader["AlbumTitle"]}");
            }
        }
    
        // End the read-only transaction by calling Commit.
        await transaction.CommitAsync();
    }

Run the example with the following command:

### GoogleSQL

    dotnet run readonlytransaction projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run readonlytransactionpg projects/PROJECT_ID/instances/test-instance/databases/example-db

The result shows:

``` 
    1 1 Total Junk
    1 2 Go, Go, Go
    2 1 Green
    2 2 Forever Hold Your Peace
    2 3 Terrified
    2 2 Forever Hold Your Peace
    1 2 Go, Go, Go
    2 1 Green
    2 3 Terrified
    1 1 Total Junk
```

## Partitioned DML

[Partitioned Data Manipulation Language (DML)](https://docs.cloud.google.com/spanner/docs/dml-partitioned) is designed for the following types of bulk updates and deletes:

  - Periodic cleanup and garbage collection.
  - Backfilling new columns with default values.

### GoogleSQL

    public static async Task PartitionedDml(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Enable Partitioned DML on this connection.
        await using var command = connection.CreateCommand();
        command.CommandText = "SET AUTOCOMMIT_DML_MODE='PARTITIONED_NON_ATOMIC'";
        await command.ExecuteNonQueryAsync();
    
        // Back-fill a default value for the MarketingBudget column.
        command.CommandText = "UPDATE Albums SET MarketingBudget=0 WHERE MarketingBudget IS NULL";
        var affected = await command.ExecuteNonQueryAsync();
    
        // Partitioned DML returns the minimum number of records that were affected.
        Console.WriteLine($"Updated at least {affected} albums");
    
        // Reset the value for AUTOCOMMIT_DML_MODE to its default.
        command.CommandText = "RESET AUTOCOMMIT_DML_MODE";
        await command.ExecuteNonQueryAsync();
    }

### PostgreSQL

    public static async Task PartitionedDml(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Enable Partitioned DML on this connection.
        await using var command = connection.CreateCommand();
        command.CommandText = "set autocommit_dml_mode='partitioned_non_atomic'";
        await command.ExecuteNonQueryAsync();
    
        // Back-fill a default value for the MarketingBudget column.
        command.CommandText = "update albums set marketing_budget=0 where marketing_budget is null";
        var affected = await command.ExecuteNonQueryAsync();
    
        // Partitioned DML returns the minimum number of records that were affected.
        Console.WriteLine($"Updated at least {affected} albums");
    
        // Reset the value for autocommit_dml_mode to its default.
        command.CommandText = "reset autocommit_dml_mode";
        await command.ExecuteNonQueryAsync();
    }

Run the example with the following command:

### GoogleSQL

    dotnet run pdml projects/PROJECT_ID/instances/test-instance/databases/example-db

### PostgreSQL

    dotnet run pdmlpg projects/PROJECT_ID/instances/test-instance/databases/example-db

## Cleanup

To avoid incurring additional charges to your Cloud Billing account for the resources used in this tutorial, drop the database and delete the instance that you created.

### Delete the database

If you delete an instance, all databases within it are automatically deleted. This step shows how to delete a database without deleting an instance (you would still incur charges for the instance).

#### On the command line

    gcloud spanner databases delete example-db --instance=test-instance

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.
    
    [Go to the Instances page](https://console.cloud.google.com/spanner/instances)

2.  Click the instance.

3.  Click the database that you want to delete.

4.  In the **Database details** page, click **Delete** .

5.  Confirm that you want to delete the database and click **Delete** .

### Delete the instance

Deleting an instance automatically drops all databases created in that instance.

#### On the command line

    gcloud spanner instances delete test-instance

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.
    
    [Go to the Instances page](https://console.cloud.google.com/spanner/instances)

2.  Click your instance.

3.  Click **Delete** .

4.  Confirm that you want to delete the instance and click **Delete** .

## What's next

  - Learn how to [access Spanner with a virtual machine instance](https://docs.cloud.google.com/spanner/docs/configure-virtual-machine-instance) .

  - Learn about authorization and authentication credentials in [Authenticate to Cloud services using client libraries](https://docs.cloud.google.com/docs/authentication/getting-started) .

  - Learn more about Spanner [Schema design best practices](https://docs.cloud.google.com/spanner/docs/schema-design) .
