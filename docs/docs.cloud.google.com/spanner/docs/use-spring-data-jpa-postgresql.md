Spring Data JPA, part of the larger Spring Data family, makes it easier to implement JPA based repositories. Spring Data JPA supports PostgreSQL and a wide range of other database systems. It adds an abstraction layer between your application and your database that makes your application easier to port from one database system to another.

## Set up Spring Data JPA for Spanner PostgreSQL-dialect databases

You can integrate Spanner PostgreSQL-dialect databases with Spring Data JPA using the standard PostgreSQL Hibernate dialect and [PGAdapter](/spanner/docs/pgadapter) .

To see an example, refer to the full [working sample application](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/spring-data-jpa) on GitHub.

### Dependencies

In your project, add Apache Maven dependencies for [Spring Data JPA](https://spring.io/projects/spring-data-jpa) , the [PostgreSQL JDBC driver](https://github.com/pgjdbc/pgjdbc) , and [PGAdapter](/spanner/docs/pgadapter) .

``` text
  <dependencies>
    <!-- Spring Data JPA -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!-- Add the PostgreSQL JDBC driver -->
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
    </dependency>
    <!-- Add PGAdapter as a dependency, so we can start it in-process -->
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-pgadapter</artifactId>
    </dependency>
  </dependencies>
```

### Start PGAdapter in-process

Add the following method to your application to start PGAdapter directly from your Java application. PGAdapter runs in the same JVM as your application, and the application connects to PGAdapter on `  localhost:port  ` .

``` text
  /** Starts PGAdapter in-process and returns a reference to the server. */
  static ProxyServer startPGAdapter() {
    // Start PGAdapter using the default credentials of the runtime environment on port 9432.
    OptionsMetadata options = OptionsMetadata.newBuilder().setPort(9432).build();
    ProxyServer server = new ProxyServer(options);
    server.startServer();
    server.awaitRunning();

    return server;
  }
```

### Configuration

Configure `  application.properties  ` to use the PostgreSQL Hibernate Dialect and the PostgreSQL JDBC Driver. Configure the PostgreSQL JDBC Driver to connect to a PostgreSQL-dialect database through PGAdapter.

``` text
# The example uses the standard PostgreSQL Hibernate dialect.
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

# Defining these properties here makes it a bit easier to build the connection string.
# Change these to match your Cloud Spanner PostgreSQL-dialect database.
spanner.project=my-project
spanner.instance=my-instance
spanner.database=my-database
# This setting ensures that PGAdapter automatically commits the current transaction if it encounters
# a DDL statement in a read/write transaction, and then executes the DDL statements as a single DDL
# batch.
spanner.ddl_transaction_mode=options=-c%20spanner.ddl_transaction_mode=AutocommitExplicitTransaction

# This is the connection string to PGAdapter running in-process.
spring.datasource.url=jdbc:postgresql://localhost:9432/projects%2F${spanner.project}%2Finstances%2F${spanner.instance}%2Fdatabases%2F${spanner.database}?${spanner.ddl_transaction_mode}

# You can display SQL statements and stats for debugging if needed.
spring.jpa.properties.hibernate.show_sql=true
spring.jpa.properties.hibernate.format_sql=true

# Enable JDBC batching.
spring.jpa.properties.hibernate.jdbc.batch_size=100
spring.jpa.properties.hibernate.order_inserts=true
```

## Full Sample Application

A [working sample application](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/spring-data-jpa) is available on GitHub.

## What's next

  - Learn more about [Spring Data JPA](https://spring.io/projects/spring-data-jpa) .
  - Learn more about [Hibernate ORM](https://hibernate.org/orm/) .
  - View the repository for [PGAdapter](https://github.com/GoogleCloudPlatform/pgadapter) on GitHub.
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/pgadapter/issues) to report a bug or ask a question about PGAdapter.
  - [Integrate Spanner with Spring Data JPA (GoogleSQL dialect)](/spanner/docs/use-spring-data-jpa) .
