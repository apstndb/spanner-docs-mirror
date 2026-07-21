---
name: documents/docs.cloud.google.com/spanner/docs/use-hibernate
uri: https://docs.cloud.google.com/spanner/docs/use-hibernate
title: Integrate Spanner with Hibernate ORM (GoogleSQL dialect)
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

Hibernate is an object-relational mapping tool for the Java programming language. It provides a framework for mapping an object-oriented domain model to a relational database.

You can integrate GoogleSQL-dialect databases with Hibernate. Spanner is compatible with [Hibernate ORM 6.x and 7.x](https://hibernate.org/orm/) . Both the built-in dialect and the open-source dialect produce SQL, DML, and DDL statements for most common entity types and relationships using standard Hibernate and Java Persistence annotations.

## Use built-in Hibernate integration (Hibernate ORM 7.4+)

Starting with Hibernate ORM 7.4, Hibernate includes a built-in dialect for Spanner ( `org.hibernate.dialect.SpannerDialect` ).

We recommend that new projects use this built-in dialect, as it does not require adding the external `google-cloud-spanner-hibernate-dialect` dependency. The built-in dialect lacks some features and optimizations that are available in the [open-source Spanner dialect](https://docs.cloud.google.com/spanner/docs/use-hibernate#setup-external) .

To use the built-in dialect, add the Maven dependencies for Hibernate ORM core and the Spanner JDBC driver to your project's `pom.xml` file. To find the latest versions of these dependencies, see the [Hibernate ORM releases](https://github.com/hibernate/hibernate-orm) and the [Spanner JDBC driver releases](https://github.com/googleapis/google-cloud-java/tree/main/java-spanner-jdbc) :

    <dependencies>
      <!-- Hibernate ORM Core -->
      <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>7.4.0.Final</version>
      </dependency>
    
      <!-- Cloud Spanner JDBC Driver -->
      <dependency>
        <groupId>com.google.cloud</groupId>
        <artifactId>google-cloud-spanner-jdbc</artifactId>
        <version>2.40.0</version>
      </dependency>
    </dependencies>

Configure your project's `hibernate.properties` file (typically located in the `src/main/resources` directory) to use the built-in Spanner Dialect:

    hibernate.dialect=org.hibernate.dialect.SpannerDialect
    hibernate.connection.driver_class=com.google.cloud.spanner.jdbc.JdbcDriver
    hibernate.connection.url=jdbc:cloudspanner:/projects/YOUR-PROJECT/instances/YOUR-INSTANCE/databases/YOUR-DATABASE

To authenticate with Spanner, the JDBC driver requires credentials. Set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to the path of your [service account JSON credentials file](https://docs.cloud.google.com/docs/authentication/getting-started) . Otherwise, the driver uses the default credentials set in the Google Cloud CLI `gcloud` application.

## Use open-source Spanner dialect

For projects using earlier versions of Hibernate (6.x or 7.x prior to 7.4), or if you need features and optimizations that are not yet supported by the built-in dialect, you can use the open-source [Spanner Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) .

To use the open-source dialect, add the Maven dependencies to your project's `pom.xml` file:

    <dependencies>
      <!-- The Spanner JDBC driver dependency -->
      <dependency>
        <groupId>com.google.cloud</groupId>
        <artifactId>google-cloud-spanner-jdbc</artifactId>
      </dependency>
    
      <!-- Hibernate core dependency -->
      <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>6.4.4.Final</version>
      </dependency>
    </dependencies>

Configure your project's `hibernate.properties` file to use the open-source Spanner Dialect and JDBC Driver:

    hibernate.dialect=com.google.cloud.spanner.hibernate.SpannerDialect
    hibernate.connection.driver_class=com.google.cloud.spanner.jdbc.JdbcDriver
    hibernate.connection.url=jdbc:cloudspanner:/projects/YOUR-PROJECT/instances/YOUR-INSTANCE/databases/YOUR-DATABASE

For more information about the features and recommendations for Hibernate when using this dialect, consult the [reference documentation](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/blob/-/README.adoc) on GitHub.

## What's next

  - Checkout [code examples](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/blob/-/google-cloud-spanner-hibernate-samples) using Hibernate with Spanner.
  - Try the Spanner with Hibernate ORM [codelab](https://codelabs.developers.google.com/codelabs/cloud-spanner-hibernate) .
  - Learn more about [Hibernate ORM](https://hibernate.org/orm/) .
  - View the repository for [Spanner Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) on GitHub.
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/issues) to report a bug or ask a question about Hibernate.
  - Learn more about [Apache Maven](https://maven.apache.org/) .
  - Learn more about [Integrate Spanner with Hibernate ORM (PostgreSQL dialect)](https://docs.cloud.google.com/spanner/docs/use-hibernate-postgresql) .
