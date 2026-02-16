Spring Data JPA, part of the larger Spring Data family, makes it easier to implement JPA based repositories. Spring Data JPA supports Spanner and a wide range of other database systems. It adds an abstraction layer between your application and your database that makes your application easier to port from one database system to another.

## Set up Spring Data JPA for Spanner GoogleSQL-dialect databases

You can integrate Spanner GoogleSQL-dialect databases with Spring Data JPA using the open source [Spanner Hibernate Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) ( `  SpannerDialect  ` ).

To see an example, refer to the full [working sample application](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/blob/-/google-cloud-spanner-hibernate-samples/spring-data-jpa-full-sample) on GitHub.

### Dependencies

In your project, add Apache Maven dependencies for [Spring Data JPA](https://spring.io/projects/spring-data-jpa) , [Spanner Hibernate Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) , and the Spanner officially supported [Open Source JDBC driver](/spanner/docs/use-oss-jdbc) .

``` text
  <dependencies>
    <!-- Spring Data JPA -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!-- Hibernate Dialect and JDBC Driver dependencies-->
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-hibernate-dialect</artifactId>
    </dependency>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-jdbc</artifactId>
    </dependency>
  </dependencies>
```

### Configuration

Configure `  application.properties  ` to use the Spanner Hibernate Dialect and the Spanner JDBC Driver.

``` text
# Spanner connection URL.
# - ${PROJECT_ID} Replace with your Google Cloud project ID
# - ${INSTANCE_ID} Replace with your Spanner instance ID
# - ${DATABASE_NAME} Replace with the name of your Spanner database that you created inside your Spanner instance

spring.datasource.url=jdbc:cloudspanner:/projects/${PROJECT_ID}/instances/${INSTANCE_ID}/databases/${DATABASE_NAME}

# Specify the Spanner JDBC driver.
spring.datasource.driver-class-name=com.google.cloud.spanner.jdbc.JdbcDriver

# Specify the Spanner Hibernate dialect.
spring.jpa.properties.hibernate.dialect=com.google.cloud.spanner.hibernate.SpannerDialect

spring.jpa.hibernate.ddl-auto=update

# Settings to enable batching statements for efficiency
spring.jpa.properties.hibernate.jdbc.batch_size=100
spring.jpa.properties.hibernate.order_inserts=true

# You can display SQL statements and stats for debugging if needed.
spring.jpa.properties.hibernate.show_sql=true
spring.jpa.properties.hibernate.format_sql=true
```

## Full Sample Application

A [working sample application](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/blob/-/google-cloud-spanner-hibernate-samples/spring-data-jpa-full-sample) is available on GitHub.

## What's next

  - Learn more about [Spring Data JPA](https://spring.io/projects/spring-data-jpa) .
  - Learn more about [Hibernate ORM](https://hibernate.org/orm/) .
  - View the repository for [Spanner Dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) on GitHub.
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/issues) to report a bug or ask a question about Hibernate.
  - [Integrate Spanner with Spring Data JPA (PostgreSQL dialect)](/spanner/docs/use-spring-data-jpa-postgresql) .
