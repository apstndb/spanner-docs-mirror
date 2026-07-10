---
name: documents/docs.cloud.google.com/spanner/docs/use-hibernate-postgresql
uri: https://docs.cloud.google.com/spanner/docs/use-hibernate-postgresql
title: Integrate Spanner with Hibernate ORM (PostgreSQL dialect)
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

Hibernate is an object-relational mapping tool for the Java programming language. It provides a framework for mapping an object-oriented domain model to a relational database.

You can integrate PostgreSQL-dialect databases with Hibernate using the open source Spanner JDBC driver. [Hibernate ORM 7.4](https://hibernate.org/orm/) is supported with PostgreSQL-dialect databases.

## Set up Hibernate with PostgreSQL

In your Google Cloud project, add Apache Maven dependencies for the Hibernate ORM core and Spanner JDBC driver.

    <!-- Hibernate core dependency -->
    <dependency>
      <groupId>org.hibernate.orm</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>7.4.0.Final</version>
    </dependency>
    
    <!-- Spanner JDBC Driver dependency -->
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-jdbc</artifactId>
      <version>2.40.0</version>
    </dependency>

## Set up Hibernate properties

Configure `hibernate.properties` to use the `SpannerPostgreSQLDialect` , Spanner JDBC driver, and required connection properties in the URL.

    hibernate.dialect=org.hibernate.dialect.SpannerPostgreSQLDialect
    hibernate.connection.driver_class=com.google.cloud.spanner.jdbc.JdbcDriver
    
    hibernate.connection.url=jdbc:cloudspanner:/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID?defaultsequencekind=bit_reversed_positive
    
    hibernate.show_sql=true
    hibernate.format_sql=true
    
    # hibernate.hbm2ddl.auto validate
    hibernate.hbm2ddl.auto=update

## Use Hibernate

For more information about the features and recommendations for integrating Hibernate with PostgreSQL-dialect databases, please consult the [reference documentation](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/blob/main/README.md) on GitHub.

## What's next

  - Learn more about [Spanner JDBC driver](https://github.com/googleapis/google-cloud-java/tree/main/java-spanner-jdbc) .
  - Learn more about [Hibernate ORM](https://hibernate.org/orm/) .
  - For more information about Spanner JDBC driver connection options, see [JDBC Connection Options](https://github.com/googleapis/google-cloud-java/blob/main/java-spanner-jdbc/documentation/connection_properties.md) in the Google Cloud Java GitHub repository.
  - Learn more about [Integrate Spanner with Hibernate ORM (GoogleSQL dialect)](https://docs.cloud.google.com/spanner/docs/use-hibernate) .
  - See an [overview of drivers and ORMs](https://docs.cloud.google.com/spanner/docs/drivers-overview) supported for Spanner.
