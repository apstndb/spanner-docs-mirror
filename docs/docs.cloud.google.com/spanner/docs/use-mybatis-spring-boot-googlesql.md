[MyBatis](https://mybatis.org/mybatis-3/) is a persistence framework with support for custom SQL and advanced mappings. MyBatis eliminates most of the JDBC code and manual setting of parameters and retrieval of results in your application.

## Set up MyBatis for Spanner GoogleSQL-dialect databases

You can integrate Spanner GoogleSQL-dialect databases with MyBatis and Spring Boot using the Spanner JDBC driver.

### Dependencies

In your project, add Apache Maven dependencies for [MyBatis](https://mybatis.org/mybatis-3/) , [Spring Boot](https://spring.io/projects/spring-boot) , and the [Spanner JDBC driver](https://github.com/googleapis/java-spanner-jdbc) .

``` text
<dependencies>
  <!-- MyBatis and Spring Boot -->
  <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
  </dependency>
  <dependency>
    <groupId>org.mybatis.dynamic-sql</groupId>
    <artifactId>mybatis-dynamic-sql</artifactId>
  </dependency>

  <!-- Spanner JDBC driver -->
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-spanner-jdbc</artifactId>
  </dependency>
<dependencies>
```

### Data source configuration

Configure `  application.properties  ` to use the Spanner JDBC driver and to connect to a Spanner GoogleSQL-dialect database.

``` text
spanner.project=my-project
spanner.instance=my-instance
spanner.database=mybatis-sample

spring.datasource.driver-class-name=com.google.cloud.spanner.jdbc.JdbcDriver
spring.datasource.url=jdbc:cloudspanner:/projects/${spanner.project}/instances/${spanner.instance}/databases/${spanner.database}
```

## Full sample application

To try this integration with a sample application, see [Spring Data MyBatis Sample Application with Spanner GoogleSQL](https://github.com/googleapis/java-spanner-jdbc/tree/main/samples/spring-data-mybatis/googlesql) .

## What's next

  - Learn more about [MyBatis](https://mybatis.org/mybatis-3/) .
  - Learn more about [MyBatis and Spring Boot](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html) .
  - Learn more about [Spring Boot](https://spring.io/projects/spring-boot) .
  - [File a GitHub issue](https://github.com/googleapis/java-spanner-jdbc/issues) to report a bug or ask a question about the Spanner JDBC driver.
