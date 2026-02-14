Hibernate is an object-relational mapping tool for the Java programming language. It provides a framework for mapping an object-oriented domain model to a relational database.

You can integrate GoogleSQL-dialect databases with Hibernate using the open source [Spanner Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) ( `  SpannerDialect  ` ). Spanner is compatible with [Hibernate ORM 6.x](https://hibernate.org/orm/) . Spanner Dialect produces SQL, DML, and DDL statements for most common entity types and relationships using standard Hibernate and Java Persistence annotations.

## Set up Hibernate

In your project, add Apache Maven dependencies for Hibernate ORM core, [Spanner Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) , and the Spanner officially supported [Open Source JDBC driver](/spanner/docs/use-oss-jdbc) .

``` xml
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
```

Configure `  hibernate.cfg.xml  ` to use Spanner Dialect and Spanner JDBC Driver.

``` xml
<!-- Connection settings -->
<property name="hibernate.dialect">org.hibernate.dialect.SpannerDialect</property>
<property name="hibernate.connection.driver_class">com.google.cloud.spanner.jdbc.JdbcDriver</property>
<property name="hibernate.connection.url">jdbc:cloudspanner:/projects/{YOUR_PROJECT_ID}/instances/{YOUR_INSTANCE_ID}/databases/{YOUR_DATABASE_ID}</property>
```

The [service account JSON credentials](/docs/authentication/getting-started) file location should be in the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable. The driver will use default credentials set in the Google Cloud CLI `  gcloud  ` application otherwise.

## Use Hibernate with Spanner GoogleSQL

For more information about the features and recommendations for Hibernate, consult the [reference documentation](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/blob/-/README.adoc) on GitHub.

## What's next

  - Checkout [code examples](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/blob/-/google-cloud-spanner-hibernate-samples) using Hibernate with Spanner.
  - Try the Spanner with Hibernate ORM [codelab](https://codelabs.developers.google.com/codelabs/cloud-spanner-hibernate) .
  - Learn more about [Hibernate ORM](https://hibernate.org/orm/) .
  - View the repository for [Spanner Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) on GitHub.
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/issues) to report a bug or ask a question about Hibernate.
  - Learn more about [Apache Maven](https://maven.apache.org/) .
  - Learn more about [Integrate Spanner with Hibernate ORM (PostgreSQL dialect)](/spanner/docs/use-hibernate-postgresql) .
