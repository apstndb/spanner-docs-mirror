> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes how to connect to Spanner Omni using PGAdapter. You configure PGAdapter to establish secure connections. PGAdapter supports plain text, [Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) , and [mutual TLS (mTLS)](https://docs.cloud.google.com/load-balancing/docs/mtls) security modes. These modes protect your data during transmission by providing varying levels of encryption and authentication. Each security mode requires specific client configurations to ensure data integrity and confidentiality.

You can run PGAdapter as a standalone process or integrate it directly into your application. For interactive management and manual query execution, connect to your database using standard PostgreSQL tools like `psql` . For building automated applications, use PostgreSQL-compatible drivers like the following:

  - [Java Database Connectivity (JDBC)](https://en.wikipedia.org/wiki/Java_Database_Connectivity)

  - [`pgx`](https://pkg.go.dev/github.com/jackc/pgx/v5) for Go

  - [`psycopg2`](https://pypi.org/project/psycopg2/)

  - [`psycopg3`](https://pypi.org/project/psycopg/) for Python

  - [`node-postgres`](https://node-postgres.com/) for Node.js

For code samples using some of these drivers, see [sample code](https://docs.cloud.google.com/spanner-omni/pgadapter#sample-code) in this document.

## Security modes

Spanner Omni PGAdapter supports three security modes, which define how communication is encrypted and authenticated between PGAdapter and the database. To use these modes, configure the client options as described in the following table:

| Security mode | Description                                                                                                                                                      |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plain text    | Communication is not encrypted.                                                                                                                                  |
| TLS           | Communication is encrypted using Transport Layer Security (TLS). This mode requires that you add the Spanner Omni CA certificate to the default Java truststore. |
| mTLS          | Communication is encrypted using mutual TLS (mTLS). This mode requires you to provide both a client certificate and a client private key.                        |

## Run as a standalone process

Run PGAdapter as a standalone process for non-Java applications and for standard PostgreSQL tools, for example, `psql` , when you need manual database interaction. This approach decouples the proxy from your application lifecycle, which lets you manage and update it independently. To start PGAdapter as a standalone process, use the following configuration methods based on your selected security mode.

### Use plain text mode

To start PGAdapter in plain text communication mode, run the following command:

    java -jar pgadapter.jar \
         -d DATABASE_ID \
         -e ENDPOINT \
         -r "isExperimentalHost=true;usePlainText=true"

Replace the following:

  - `  DATABASE_ID  ` : the ID of your Spanner Omni database, for example, `test-db` .

  - `  ENDPOINT  ` : the endpoint of your Spanner Omni instance, for example, `localhost:15000` .

### Use TLS mode

Before you can start PGAdapter in-process in TLS mode with your Java application, you must add your Spanner Omni CA certificate to the default Java truststore. To add an existing CA certificate to the Java truststore, run the following command:

    sudo keytool -import -trustcacerts -file ~/.spanner/certs/ca.crt -alias spanner-ca -keystore $JAVA_HOME/lib/security/cacerts

To start PGAdapter in TLS mode, run the following command:

    java -Djavax.net.ssl.trustStore=$JAVA_HOME/lib/security/cacerts \
         -Djavax.net.ssl.trustStoreType=JKS \
         -jar pgadapter.jar \
         -d DATABASE_ID \
         -e ENDPOINT \
         -r "isExperimentalHost=true"

### Use mTLS mode

Before you can start PGAdapter in mTLS mode, you must ensure that your client key is in the in PKCS\#8 format. To convert an existing key in your keystore into a PKCS\#8 format, run the following command:

    openssl pkcs8 -topk8 -in ~/.spanner/certs/client.key -out ~/.spanner/certs/java-client.key -nocrypt

Alternatively, when you create your client certificate and key using the Spanner Omni CLI, provide the `--generate-pkcs8-key` parameter to generate the key in PKCS\#8 format.

To start PGAdapter in mTLS mode, run the following command:

    java -Djavax.net.ssl.trustStore=$JAVA_HOME/lib/security/cacerts \
        -Djavax.net.ssl.trustStoreType=JKS \
        -jar pgadapter.jar \
        -d DATABASE_ID \
        -e ENDPOINT \
        -r "isExperimentalHost=true;clientCertificate=PATH_TO_CLIENT_CERT;clientKey=PATH_TO_CLIENT_KEY"

Replace the following:

  - `  PATH_TO_CLIENT_CERT  ` : the path to your client certificate file.

  - `  PATH_TO_CLIENT_KEY  ` : the path to your client key file.

## Connect with `psql`

After you establish a connection using one of the preceding methods, run `psql` to manage your database and execute queries. To connect to `psql` , use the following command:

    psql -h PG_HOST -p PG_PORT -U USERNAME -d DATABASE_ID

Replace the following:

  - `  PG_HOST  ` : the hostname or IP address of the machine where PGAdapter is running. If running locally, use `localhost` .

  - `  PG_PORT  ` : the port number where PGAdapter is running. If you haven't specified a custom port, PGAdapter uses port `5432` by default.

  - `  USERNAME  ` : your PostgreSQL username.

## Run in-process with your application

You can also start PGAdapter in-process with your application. To establish security, configure the `OptionsMetadata` object.

### Use plain text mode

For plain text communication, such as local development or testing in secure environments where encryption overhead is unnecessary, use the following configuration:

    OptionsMetadata.Builder builder =
        OptionsMetadata.newBuilder()
        .setEndpoint("ENDPOINT")
        .setUsePlainText();
    
    ProxyServer server = new ProxyServer(builder.build());
    server.startServer();
    server.awaitRunning();

### Use TLS mode

For plain text communication for use in environments such as local development or testing, use the following configuration:

    sudo keytool -import -trustcacerts -file /.spanner/certs/ca.crt -alias spanner-ca -keystore $JAVA_HOME/lib/security/cacerts

To establish a TLS connection in-process, use the following configuration:

    OptionsMetadata.Builder builder =
        OptionsMetadata.newBuilder()
        .setEndpoint("ENDPOINT");
    
    ProxyServer server = new ProxyServer(builder.build());
    server.startServer();
    server.awaitRunning();

### Use mTLS mode

Before you can start PGAdapter in-process mode with your Java application, you must ensure that your client key is in the PKCS\#8 format. To convert an existing key in your keystore into a PKCS\#8 format, run the following command:

    openssl pkcs8 -topk8 -in ~/.spanner/certs/client.key -out ~/.spanner/certs/java-client.key -nocrypt

Alternatively, when you create your client certificate and key using the Spanner Omni CLI, provide the `--generate-pkcs8-key` parameter to generate the key in PKCS\#8 format.

To establish an mTLS connection in-process, use this configuration:

    OptionsMetadata.Builder builder =
        OptionsMetadata.newBuilder()
        .setEndpoint("ENDPOINT")
        .useClientCert("PATH_TO_CLIENT_CERT", "PATH_TO_CLIENT_KEY");
    
    ProxyServer server = new ProxyServer(builder.build());
    server.startServer();
    server.awaitRunning();

## Sample code

This section provides sample code for connecting to a Spanner Omni database using the following PostgreSQL-compatible drivers:

  - [JDBC](https://docs.cloud.google.com/spanner-omni/pgadapter#jdbc)

  - [Go (pgx)](https://docs.cloud.google.com/spanner-omni/pgadapter#go-pgx)

  - [Python (psycopg2 or psycopg3)](https://docs.cloud.google.com/spanner-omni/pgadapter#python)

  - [Node.js (node-postgres)](https://docs.cloud.google.com/spanner-omni/pgadapter#node-js)

Replace the following placeholder in your connection strings:

  - PASSWORD : the password for your PostgreSQL user.

### JDBC

You can connect to PGAdapter using the PostgreSQL `JDBC` driver as if you were connecting to a PostgreSQL database. To connect and query a table in a Spanner Omni database, use the following sample code:

    String jdbcUrl =
        "jdbc:postgresql://PG_HOST:PG_PORT/DATABASE_ID";
    
    try (Connection connection = DriverManager.getConnection(jdbcUrl)) {
      // Example: Query data
      try (Statement statement = connection.createStatement();
          ResultSet resultSet = statement.executeQuery("SELECT * FROM Singers")) {
    
        System.out.println("Query Results:");
        while (resultSet.next()) {
          long id = resultSet.getLong("id");
          String name = resultSet.getString("name");
          System.out.printf("ID: %d, Name: %s\n", id, name);
        }
      } catch (SQLException e) {
        throw new RuntimeException(e);
      }
    }

### Go (pgx)

You can connect to PGAdapter using `pgx` as if you were connecting to a PostgreSQL database. Use the following sample code:

    // Database connection string
    connString := "postgres://USERNAME:PASSWORD@PG_HOST:PG_PORT/DATABASE_ID?sslmode=disable"
    ctx := context.Background()
    
    // Connect to PGAdapter
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
      log.Fatalf("Connection error: %s", err.Error())
    }
    defer conn.Close(ctx)
    
    // Query all rows from the Singers table
    rows, err := conn.Query(ctx, "SELECT id, name FROM Singers")
    if err != nil {
      log.Fatalf("Query error: %s", err.Error())
    }
    defer rows.Close()
    
    // Iterate over the result set
    fmt.Println("Singers Table Data:")
    for rows.Next() {
      var id int
      var name string
      if err := rows.Scan(&id, &name); err != nil {
        log.Fatalf("Scan error: %s", err.Error())
      }
      fmt.Printf("ID: %d, Name: %s\n", id, name)
    }

### Python (psycopg2 or psycopg3)

You can connect to PGAdapter using `psycopg2` or `psycopg3` as if you were connecting to a PostgreSQL database. To connect and query a table in a Spanner Omni database, use the following sample code:

    # psycopg2
    import psycopg2
    
    connection = psycopg2.connect(database="DATABASE_ID",
                                  host="PG_HOST",
                                  port=PG_PORT)
    
    cursor = connection.cursor()
    cursor.execute('SELECT * FROM Singers')
    for row in cursor:
      print(row)
    
    cursor.close()
    connection.close()
    
    
    # psycopg3
    import psycopg
    
    with psycopg.connect("host=PG_HOST port=PG_PORT dbname=DATABASE_ID sslmode=disable") as conn:
      conn.autocommit = True
      with conn.cursor() as cur:
        cur.execute("SELECT * FROM Singers")
        for row in cur:
          print(row)

### Node.js (node-postgres)

You can connect to PGAdapter using [`node-postgres`](https://docs.cloud.google.com/spanner/docs/pg-node-postgres-connect) as if you were connecting to a PostgreSQL database. To connect and query a table in a Spanner Omni database, use the following sample code:

    const { Client } = require('pg');
    const client = new Client({
      host: 'PG_HOST',
      port: PG_PORT,
      database: 'DATABASE_ID',
    });
    await client.connect();
    const res = await client.query("SELECT * FROM Singers");
    console.log(res.rows);
    await client.end();
