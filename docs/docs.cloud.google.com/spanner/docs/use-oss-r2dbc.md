[R2DBC](https://r2dbc.io/) is a specification for non-blocking access to relational databases, based on [Reactive Streams](https://www.reactive-streams.org/) . Your application can make use of the reactive database connectivity with Spanner by using the Spanner R2DBC driver.

## Add dependencies

Spring Data users should use the Spring Data R2DBC dialect for Spanner; all other users should bring in the Spanner R2DBC driver only.

### Use the Spanner R2DBC driver

To add only the Spanner R2DBC driver to your application, add the following dependency:

``` xml
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>cloud-spanner-r2dbc</artifactId>
  <version>1.3.0</version>
</dependency>
```

For more information, see the [Spanner R2DBC driver GitHub repository](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc) and the [sample code](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc/tree/main/cloud-spanner-r2dbc-samples/cloud-spanner-r2dbc-sample) .

### Use the Spring Data R2DBC dialect for Spanner

For users of the Spring Framework, Spring Data provides familiar abstractions to simplify interaction with common database operations.

To use [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc) features with Spanner, add the following dependency to your project. The driver is a transitive dependency of the dialect.

``` xml
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>cloud-spanner-spring-data-r2dbc</artifactId>
  <version>1.2.2</version>
</dependency>
```

To learn how to use its template and repositories, see the [Spring Data R2DBC reference](https://docs.spring.io/spring-data/r2dbc/docs/current/reference/html/) . To see which objects are automatically configured for your, see the [Spring Boot reference](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-r2dbc) .

For more information, see the [Spanner Spring Data R2DBC GitHub repository](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc/tree/main/cloud-spanner-spring-data-r2dbc) .

## Spring Boot configuration

Regardless of which R2DBC dependency you use, if your application is based on Spring Boot, the framework will attempt to automatically configure and provide a connection factory for you.

Provide a `  spring.r2dbc.url  ` property to let autoconfiguration take care of R2DBC connection factory configuration. The format is shown in the following sample `  application.properties  ` entry:

``` properties
spring.r2dbc.url=\
r2dbc:cloudspanner://spanner.googleapis.com:443/projects/${project}/instances/${instance}/databases/${database}
```

## What's next

  - Learn more about using Spanner through R2DBC with these [code examples](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc/tree/main/cloud-spanner-r2dbc-samples) .
  - Learn more about [R2DBC](https://r2dbc.io/) .
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc/issues) to report a bug or ask a question about Spanner R2DBC support.
