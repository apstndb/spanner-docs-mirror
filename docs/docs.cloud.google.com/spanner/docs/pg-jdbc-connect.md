You can use either the PostgreSQL JDBC driver or the Spanner JDBC driver with a Spanner PostgreSQL-dialect database. This page explains how to connect to your database with these drivers.

### PostgreSQL JDBC driver

This section explains how to connect the PostgreSQL JDBC driver to a PostgreSQL-dialect database in Spanner. JDBC is the standard Java driver for PostgreSQL.

If you use the PostgreSQL JDBC driver, you must use PGAdapter to translate between the PostgreSQL network protocol and the Spanner network protocol. You can add PGAdapter as a dependency and run it in-process with your application.

1.  Add PGAdapter and the PostgreSQL JDBC driver as dependencies to your application.  
      
    
    ``` markdown
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <version>0.53.1</version>
    </dependency>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-pgadapter</artifactId>
      <version>0.53.1</version>
    </dependency>
    ```

2.  Start PGAdapter in-process with your application.
    
    ``` text
    OptionsMetadata.Builder builder =
      OptionsMetadata.newBuilder()
        .setProject("PROJECT_NAME")
        .setInstance("INSTANCE_NAME")
        .setPort(PORT);
    ProxyServer server = new ProxyServer(builder.build());
    server.startServer();
    server.awaitRunning();
          
    ```
    
    Replace the following:
    
      - PORT : the port number where PGAdapter is running. Set to `  5432  ` in most cases or `  0  ` to use a dynamically assigned port.

3.  Make sure the PostgreSQL JDBC driver driver is loaded.
    
    ``` text
    Class.forName("org.postgresql.Driver");
    
    try (Connection connection =
        DriverManager.getConnection("jdbc:postgresql://APPLICATION_HOST:PORT/DATABASE_NAME")) {
      try (ResultSet resultSet =
          connection.createStatement().executeQuery("select 'Hello world!' as hello")) {
        while (resultSet.next()) {
          System.out.printf(
            "Greetings from Cloud Spanner PostgreSQL: %s\n", resultSet.getString(1));
          }
        }
      }
        
    ```
    
    Replace the following:
    
      - APPLICATION\_HOST : the hostname or IP address of the machine where PGAdapter is running. If running locally, use `  localhost  ` .
    
    The PGAdapter GitHub repository contains a [sample application.](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/jdbc)

### Unix domain sockets

This section explains how to use Unix domain sockets to connect PostgreSQL JDBC driver to a PostgreSQL-dialect database. Use Unix domain sockets for the lowest possible latency.

To use Unix domain sockets, PGAdapter must be running on the same host as the client application.

``` text
// Make sure the PG JDBC driver is loaded.
Class.forName("org.postgresql.Driver");

try (Connection connection = DriverManager.getConnection("jdbc:postgresql://APPLICATION_HOST/DATABASE_NAME"
    + "?socketFactory=org.newsclub.net.unix.AFUNIXSocketFactory$FactoryArg"
    + "&socketFactoryArg=DIRECTORY_NAME.s.PGSQL.PORT")) {
  try (ResultSet resultSet = connection.createStatement().executeQuery("select 'Hello world!' as hello")) {
    while (resultSet.next()) {
      System.out.printf("Greeting from Cloud Spanner PostgreSQL: %s\n", resultSet.getString(1));
      }
    }
  }
  
```

Replace the following:

  - APPLICATION\_HOST : the hostname or IP address of the machine where PGAdapter is running. If running locally, use `  localhost  ` .
  - PORT : the port number where PGAdapter is running. Change this in the connection string if PGAdapter is running on a custom port. Otherwise, use the default port, `  5432  ` .

### Spanner JDBC driver

This section explains how to use the Spanner JDBC driver to connect to a PostgreSQL-dialect database database.

1.  Add the Spanner JDBC driver as a dependency to your application.  
      
    
    ``` xml
    <dependencyManagement>
      <dependencies>
        <dependency>
          <groupId>com.google.cloud</groupId>
          <artifactId>libraries-bom</artifactId>
          <version>26.76.0</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
      </dependencies>
    </dependencyManagement>
    
    <dependencies>
      <dependency>
        <groupId>com.google.cloud</groupId>
        <artifactId>google-cloud-spanner-jdbc</artifactId>
        <exclusions>
          <exclusion>
            <groupId>com.google.api.grpc</groupId>
            <artifactId>proto-google-cloud-spanner-executor-v1</artifactId>
          </exclusion>
        </exclusions>
      </dependency>
    ```

2.  Use a Spanner JDBC connection URL to connect to the PostgreSQL-dialect database.  
      
    
    ``` text
        // Make sure the PostgreSQL JDBC driver is loaded.
        Class.forName("org.postgresql.Driver");
    
        try (Connection connection = DriverManager.getConnection(
            "jdbc:cloudspanner:/projects/PROJECT_NAME/instances/INSTANCE_NAME/databases/DATABASE_NAME")) {
          try (ResultSet resultSet =
              connection.createStatement().executeQuery("select 'Hello world!' as hello")) {
            while (resultSet.next()) {
              System.out.printf(
                "Greetings from Cloud Spanner PostgreSQL: %s\n", resultSet.getString(1));
            }
          }
        }
        
    ```
    
    The driver automatically detects the SQL dialect of the specified database. A dialect parameter in the connection URL is not required.

## What's next

  - Learn more about [PGAdapter](/spanner/docs/pgadapter) .
  - For more information about PostgreSQL JDBC driver connection options, see [PGAdapter - JDBC Connection Options](https://github.com/GoogleCloudPlatform/pgadapter/blob/postgresql-dialect/docs/jdbc.md) in the PGAdapter GitHub repository.
