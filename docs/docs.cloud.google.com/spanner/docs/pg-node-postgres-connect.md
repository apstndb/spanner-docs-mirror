This page explains how to connect the PostgreSQL node-postgres driver to a PostgreSQL-dialect database in Spanner. node-postgres is a Node.js driver for PostgreSQL.

1.  Verify that PGAdapter is running on the same machine as the application that is connecting using the PostgreSQL node-postgres driver.
    
    For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

2.  Specify the database server host and port in the `  node-postgres  ` connection properties:
    
    ``` text
    const { Client } = require('pg');
    const client = new Client({
      host: 'APPLICATION_HOST',
      port: PORT,
      database: 'DATABASE_NAME',
    });
    await client.connect();
    const res = await client.query("select 'Hello world!' as hello");
    console.log(res.rows[0].hello);
    await client.end();
    ```
    
    Replace the following:
    
      - APPLICATION\_HOST : the hostname or IP address of the machine where PGAdapter is running. If running locally, you can use `  localhost  ` .
      - PORT : the port number where PGAdapter is running. Change this in the connection string if PGAdapter is running on a custom port. Otherwise, use the default port, `  5432  ` .

## Unix domain sockets

This section explains how to use Unix domain sockets to connect a PostgreSQL node-postgres driver to a PostgreSQL-dialect database. Use Unix domain socket connections when you need to have the lowest possible latency.

To use Unix domain sockets, PGAdapter must be running on the same host as the client application.

``` text
const client = new Client({
  host: '/tmp',
  port: PORT,
  database: 'DATABASE_NAME',
});
await client.connect();
const res = await client.query("select 'Hello world!' as hello");
console.log(res.rows[0].hello);
await client.end();
```

Replace the following:

  - /tmp : the default domain socket directory for PGAdapter. This can be changed using the `  -dir  ` command line argument.
  - PORT : the port number where PGAdapter is running. Change this in the connection string if PGAdapter is running on a custom port. Otherwise, use the default port, `  5432  ` .

## What's next

  - Learn more about [PGAdapter](/spanner/docs/pgadapter) .
  - For more information about PostgreSQL node-postgres driver connection options, see [node-postgres Connection Options](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/node-postgres.md) in the PGAdapter GitHub repository.
