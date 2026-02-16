GORM is an object-relational mapping tool for the Go programming language. It provides a framework for mapping an object-oriented domain model to a relational database.

You can integrate Spanner PostgreSQL databases with GORM using the standard PostgreSQL pgx driver and PGAdapter.

## Set up GORM with Spanner PostgreSQL-dialect databases

1.  Ensure that PGAdapter is running on the same machine as the application that is connecting using GORM with Spanner.
    
    For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

2.  Add an import statement for the PostgreSQL GORM dialect to your application. This is the same driver as you would normally use with a PostgreSQL database.

3.  Specify `  localhost  ` and `  5432  ` as the database server host and port in the GORM connection string. GORM requires a username and password in the connection string. PGAdapter ignores these.
    
      - Optionally, specify a different port number if PGAdapter is configured to listen on a port other than the default PostgreSQL port (5432).
      - PGAdapter does not support SSL. GORM by default first tries to connect with SSL enabled. Disabling SSL in the connection request speeds up the connection process, because it requires one fewer round trip.
    
    <!-- end list -->
    
    ``` text
    import (
      "gorm.io/driver/postgres"
      "gorm.io/gorm"
    )
    
    dsn := "host=localhost user=gorm password=gorm dbname=gorm port=5432 sslmode=disable"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    ```

See the [GORM with PostgreSQL documentation](https://gorm.io/docs/connecting_to_the_database.html#PostgreSQL) for more connection options for PostgreSQL.

## Use GORM with Spanner PostgreSQL-dialect databases

For more information about the features and recommendations for using GORM with Spanner, consult the [reference documentation](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/golang/gorm) on GitHub.

## What's next

  - Checkout the [sample application](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/golang/gorm/sample.go) using GORM with PGAdapter and Spanner.
  - Learn more about [GORM](https://gorm.io/) .
  - Learn more about [PGAdapter](/spanner/docs/pgadapter) .
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/pgadapter/issues) to report a bug or ask a question about using GORM with Spanner with PGAdapter.
