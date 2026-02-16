[Spring Data JDBC](https://spring.io/projects/spring-data-jdbc) , part of the larger Spring Data family, makes it easier to implement JDBC based repositories in your application. It adds an abstraction layer between your application and your database that makes your application easier to port from one database system to another.

## Set up Spring Data JDBC for Spanner GoogleSQL-dialect databases

You can integrate Spanner GoogleSQL-dialect databases with Spring Data JDBC by adding a dialect for Spanner GoogleSQL to your application.

### Dialect configuration

Spring Data JDBC selects the database dialect it uses based on the JDBC driver that has been configured as a data source in Spring Data. You must add an additional dialect provider to your application to instruct Spring Data JDBC to use the GoogleSQL dialect when the Spanner JDBC driver is used:

``` text
public class SpannerDialectProvider implements DialectResolver.JdbcDialectProvider {
  @Override
  public Optional<Dialect> getDialect(JdbcOperations operations) {
    return Optional.ofNullable(
        operations.execute((ConnectionCallback<Dialect>) SpannerDialectProvider::getDialect));
  }

  @Nullable
  private static Dialect getDialect(Connection connection) throws SQLException {
    DatabaseMetaData metaData = connection.getMetaData();
    String name = metaData.getDatabaseProductName().toLowerCase(Locale.ENGLISH);
    if (name.contains("spanner")) {
      return SpannerDialect.INSTANCE;
    }
    return null;
  }
}
```

This dialect provider must be added to the `  spring.factories  ` file in your application:

``` text
org.springframework.data.jdbc.repository.config.DialectResolver$JdbcDialectProvider=org.springframework.data.jdbc.repository.config.DialectResolver.DefaultDialectProvider,com.google.cloud.spanner.sample.SpannerDialectProvider
```

To see an example, refer to the full [working sample application](https://github.com/googleapis/java-spanner-jdbc/tree/main/samples/spring-data-jdbc/googlesql) on GitHub.

### Dependencies

In your project, add Apache Maven dependencies for [Spring Data JDBC](https://spring.io/projects/spring-data-jdbc) and the [Spanner JDBC driver](https://github.com/googleapis/java-spanner-jdbc) .

``` text
<dependencies>
  <!-- Spring Data JDBC -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
  </dependency>

  <!-- Spanner JDBC driver -->
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-spanner-jdbc</artifactId>
  </dependency>
<dependencies>
```

### Data source configuration

Configure `  application.properties  ` to use the Spanner JDBC driver and connect to a Spanner GoogleSQL-dialect database.

``` text
spanner.project=my-project
spanner.instance=my-instance
spanner.database=spring-data-jdbc

spring.datasource.driver-class-name=com.google.cloud.spanner.jdbc.JdbcDriver
spring.datasource.url=jdbc:cloudspanner:/projects/${spanner.project}/instances/${spanner.instance}/databases/${spanner.database}
```

## Full Sample Application

To try this integration with a sample application, see [Spring Data JDBC Sample Application with Spanner GoogleSQL](https://github.com/googleapis/java-spanner-jdbc/tree/main/samples/spring-data-jdbc/googlesql) .

## What's next

  - Learn more about [Spring Data JDBC](https://spring.io/projects/spring-data-jdbc) .
  - [File a GitHub issue](https://github.com/googleapis/java-spanner-jdbc/issues) to report a bug or ask a question about the Spanner JDBC driver.
