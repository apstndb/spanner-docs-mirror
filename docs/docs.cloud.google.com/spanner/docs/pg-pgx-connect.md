This page explains how to connect the PostgreSQL pgx driver to a PostgreSQL-dialect database in Spanner. `  pgx  ` is a Golang driver for PostgreSQL.

Verify that PGAdapter is running on the same machine as the application that is connecting using the PostgreSQL pgx driver.

For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

  - `  pgx  ` requires a username and password in the connection string. PGAdapter ignores these.
  - By default, PGAdapter disables SSL. `  pgx  ` by default first tries to connect with SSL enabled. Disabling SSL in the connection request speeds up the connection process, as it requires one fewer round trip.

<!-- end list -->

``` text
connString := "postgres://uid:pwd@APPLICATION_HOST:PORT/DATABASE_NAME?sslmode=disable"
ctx := context.Background()
conn, err := pgx.Connect(ctx, connString)
if err != nil {
  return err
}
defer conn.Close(ctx)

var greeting string
err = conn.QueryRow(ctx, "select 'Hello world!' as hello").Scan(&greeting)
if err != nil {
  return err
}
fmt.Printf("Greeting from Cloud Spanner PostgreSQL: %v\n", greeting)
```

Replace the following:

  - APPLICATION\_HOST : the hostname or IP address of the machine where PGAdapter is running. If running locally, you can use `  localhost  ` .
  - PORT : the port number where PGAdapter is running. Change this in the connection string if PGAdapter is running on a custom port. Otherwise, use the default port, `  5432  ` .

### Unix domain sockets

This section explains how to use Unix domain sockets to connect the PostgreSQL pgx driver to a PostgreSQL-dialect database. Use Unix domain sockets for the lowest possible latency.

To use Unix domain sockets, PGAdapter must be running on the same host as the client application.

``` text
connString := "host=/tmp port=PORT database=DATABASE_NAME"
ctx := context.Background()
conn, err := pgx.Connect(ctx, connString)
if err != nil {
    return err
}
defer conn.Close(ctx)

var greeting string
err = conn.QueryRow(ctx, "select 'Hello world!' as hello").Scan(&greeting)
if err != nil {
    return err
}
fmt.Printf("Greeting from Cloud Spanner PostgreSQL: %v\n", greeting)
```

Replace the following:

  - /tmp : the default domain socket directory for PGAdapter. This can be changed using the `  -dir  ` command line argument.
  - PORT : the port number where PGAdapter is running. Change this in the connection string if PGAdapter is running on a custom port. Otherwise, use the default port, `  5432  ` .

## What's next

  - Learn more about [PGAdapter](/spanner/docs/pgadapter) .
  - Learn more about [pgx Connection Options](https://github.com/GoogleCloudPlatform/pgadapter/blob/postgresql-dialect/docs/pgx.md) in the PGAdapter GitHub repository.
