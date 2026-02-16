Hibernate is an object-relational mapping tool for the Java programming language. It provides a framework for mapping an object-oriented domain model to a relational database.

You can integrate PostgreSQL-dialect databases with Hibernate using the open source PostgreSQL JDBC Driver. [Hibernate ORM 6.3](https://hibernate.org/orm/) is supported with PostgreSQL-dialect databases.

## Set up PGAdapter

Ensure that PGAdapter is running on the same machine as the application that is using Hibernate.

For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

## Set up Hibernate with PostgreSQL

In your project, add Apache Maven dependencies for the Hibernate ORM core and PostgreSQL JDBC Driver.

``` text
<!-- Hibernate core dependency -->
<dependency>
  <groupId>org.hibernate.orm</groupId>
  <artifactId>hibernate-core</artifactId>
  <version>6.3.1.Final</version>
</dependency>

<!-- Postgresql JDBC driver dependency -->
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <version>42.7.1</version>
</dependency>
```

## Set up Hibernate properties

Configure `  hibernate.properties  ` to use the PostgreSQL dialect and PostgreSQL JDBC Driver.

``` text
hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
hibernate.connection.driver_class=org.postgresql.Driver

hibernate.connection.url=jdbc:postgresql://localhost:5432/test-database
hibernate.connection.username=pratick

hibernate.connection.pool_size=5

hibernate.show_sql=true
hibernate.format_sql=true

# hibernate.hbm2ddl.auto validate
hibernate.hbm2ddl.auto=update
```

## Use Hibernate

For more information about the features and recommendations for integrating Hibernate with PostgreSQL-dialect databases, please consult the [reference documentation](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/hibernate/README.md) on GitHub.

## What's next

  - Checkout [code examples](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/hibernate) using Hibernate with PostgreSQL.
  - Learn more about [PGAdapter](/spanner/docs/pgadapter) .
  - Learn more about [Hibernate ORM](https://hibernate.org/orm/) .
  - For more information about PostgreSQL JDBC driver connection options, see [PGAdapter - JDBC Connection Options](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/jdbc.md) in the PGAdapter GitHub repository.
  - Learn more about [Integrate Spanner with Hibernate ORM (GoogleSQL dialect)](/spanner/docs/use-hibernate) .
