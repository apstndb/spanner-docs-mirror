This page explains how to connect the PostgreSQL psycopg2 driver to a PostgreSQL-dialect database in Spanner. `  psycopg2  ` is a Python driver for PostgreSQL.

Verify that PGAdapter is running on the same machine as the application that is connecting using the PostgreSQL psycopg2 driver.

For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

``` text
connection = psycopg2.connect(database="DATABASE_NAME",
                              host="APPLICATION_HOST",
                              port=PORT)

cursor = connection.cursor()
cursor.execute('select \'Hello World\'')
for row in cursor:
  print(row)

cursor.close()
connection.close()
```

Replace the following:

  - APPLICATION\_HOST : the hostname or IP address of the machine where PGAdapter is running. If running locally, you can use `  localhost  ` .
  - PORT : the port number where PGAdapter is running. Change this in the connection string if PGAdapter is running on a custom port. Otherwise, use the default port, `  5432  ` .

### Unix domain sockets

This section explains how to use Unix domain sockets to connect to a PostgreSQL-dialect database database. Use Unix domain socket connections when you need to have the lowest possible latency.

To use Unix domain sockets, PGAdapter must be running on the same host as the client application.

``` text
connection = psycopg2.connect(database="DATABASE_NAME",
                              host="/tmp",
                              port=PORT)

cursor = connection.cursor()
cursor.execute('select \'Hello World\'')
for row in cursor:
  print(row)

cursor.close()
connection.close()
```

Replace the following:

  - /tmp : the default domain socket directory for PGAdapter. This can be changed using the `  -dir  ` command line argument.
  - PORT : the port number where PGAdapter is running. Change this in the connection string if PGAdapter is running on a custom port. Otherwise, use the default port, `  5432  ` .

## What's next

  - Learn more about [PGAdapter](/spanner/docs/pgadapter) .
  - For more information about PostgreSQL psycopg2 driver connection options, see [psycopg2 Connection Options](https://github.com/GoogleCloudPlatform/pgadapter/blob/postgresql-dialect/docs/psycopg2.md) in the PGAdapter GitHub repository.
