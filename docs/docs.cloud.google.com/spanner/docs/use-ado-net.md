[ADO.NET](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview) is a generic interface for access to data sources for .NET. To use ADO.NET with your application, use the [Spanner ADO.NET driver](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net) .

The SpannerADO.NET driver supports both GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Install the Spanner ADO.NET driver

To use the Spanner ADO.NET driver in your application, add the following package to your .NET project:

``` 
  Google.Cloud.Spanner.DataProvider
```

## Use the Spanner ADO.NET driver

To create a ADO.NET connection to a Spanner database, create a `SpannerConnectionStringBuilder` with a fully qualified database name as the connection string:

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

For more information, see the [Spanner ADO.NET driver GitHub repository](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net) .

## Supported features

The [Spanner ADO.NET driver examples code directory](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net-samples/Snippets) contains ready-to-run examples for commonly used Spanner features.

## Performance tips

To get the best possible performance when using the Spanner ADO.NET driver, follow these best practices:

  - Query parameters: [Use query parameters](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net-samples/Snippets/QueryParametersSample.cs) instead of inline values in SQL statements. This lets Spanner cache and reuse the execution plan for frequently used SQL statements.
  - Database Definition Language (DDL): [Group multiple DDL statements into one batch](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net-samples/Snippets/DdlBatchSample.cs) instead of executing them one by one.
  - Data Manipulation Language (DML): [Group multiple DML statements into one batch](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net-samples/Snippets/DmlBatchSample.cs) instead of executing them one by one.
  - Read-only transactions: [Use read-only transactions](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net-samples/Snippets/ReadOnlyTransactionSample.cs) for workloads that only read data. Read-only transactions don't take locks.
  - Tags: [Use request and transaction tags](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/main/spanner-ado-net/spanner-ado-net-samples/Snippets/TagsSample.cs) to [troubleshoot](https://docs.cloud.google.com/spanner/docs/introspection/troubleshooting-with-tags) .

## What's next

  - Learn more about using Spanner with the ADO.NET driver [code examples](https://github.com/googleapis/dotnet-spanner-entity-framework/blob/-/spanner-ado-net/spanner-ado-net-samples/Snippets) .
  - Learn more about [ADO.NET](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview) .
  - [File a GitHub issue](https://github.com/googleapis/dotnet-spanner-entity-framework/issues) to report a feature request or bug, or to ask a question about the Spanner ADO.NET driver.
