**PostgreSQL interface note:** The [PostgreSQL interface for Spanner](/spanner/docs/postgresql-interface) doesn't support Spring Data Spanner. You can use [Spring Data JPA](/spanner/docs/use-spring-data-jpa-postgresql) instead.

The Spring Data Spanner module helps you use Spanner in any Java application that's built with the [Spring Framework](https://spring.io/projects/spring-framework) .

Like all [Spring Data](https://spring.io/projects/spring-data) modules, Spring Data Spanner provides a Spring-based programming model that retains the consistency guarantees and scalability of Spanner. Its features are similar to [Spring Data JPA](https://spring.io/projects/spring-data-jpa) and [Hibernate ORM](https://hibernate.org/orm/) , with annotations designed for Spanner. For more information about how to use Spring Data JPA with Spanner, see [Integrate Spanner with Spring Data JPA (GoogleSQL dialect)](/spanner/docs/use-spring-data-jpa) .

If you're already familiar with Spring, then Spring Data Spanner can make it easier to work with Spanner in your application and reduce the amount of code that you need to write.

This page explains how to add Spring Data Spanner to a Java application. For detailed information about the module, see the [Spring Data Spanner reference](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/spanner.html) .

## Install the module

If you use Maven, add the [Spring Cloud GCP Bill of Materials (BOM)](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/index.html#bill-of-materials) and Spring Data Spanner to your `  pom.xml  ` file. These dependencies provide the Spring Data Spanner components to your Spring [`  ApplicationContext  `](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html) :

``` xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>spring-cloud-gcp-dependencies</artifactId>
      <version>3.7.7</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>${spring.boot.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-data-spanner</artifactId>
  </dependency>
</dependencies>
```

You must also [create a service account](/docs/authentication/getting-started) and use the service account key to authenticate with Google Cloud.

For more information, see the instructions for [setting up a Java development environment](/java/docs/setup) . You do not need to install the Google Cloud Client Library for Java; the Spring Boot starter installs the client library automatically.

## Configure the module

This section describes some of the most commonly used configuration settings for Spring Data Spanner. For a complete list of settings, see the [reference documentation](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/#cloud-spanner-settings) .

### Specify an instance and database

To specify the default instance and database, set the following configuration properties for your application:

<table>
<thead>
<tr class="header">
<th>Property</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spring.cloud.gcp.spanner.project-id      </code></td>
<td>Optional. The Google Cloud project ID. Overrides the value of <code dir="ltr" translate="no">       spring.cloud.gcp.config.project-id      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spring.cloud.gcp.spanner.instance-id      </code></td>
<td>The Spanner instance ID.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spring.cloud.gcp.spanner.database      </code></td>
<td>The database to connect to.</td>
</tr>
</tbody>
</table>

## Model Spanner data

With Spring Data Spanner, you can use plain old Java objects (POJOs) to model the data you store in your Spanner tables.

For each table in your database, declare an entity that represents a record in that table. Use annotations to map the entity and its properties to a table and its columns.

You can use the following annotations to model simple relationships between entities and tables:

Entity annotations

`  @Column(name = " columnName ")  `

Optional. Maps the property to a specific column in the Spanner table, overriding the naming strategy that automatically maps the names.

When you omit this property, the default naming strategy for Spring Data Spanner maps Java `  camelCase  ` property names to `  PascalCase  ` column names. For example, the property `  singerId  ` maps to the column name `  SingerId  ` .

`  @Embedded  `

Indicates that the property is an embedded object that can hold components of a primary key. If the property is actually used in the primary key, you must also include the `  @PrimaryKey  ` annotation.

`  @Interleaved  `

`  @Interleaved(lazy = true )  `

Indicates that a property contains a list of rows that are [interleaved](/spanner/docs/schema-and-data-model#creating-interleaved-tables) with the current row.

By default, Spring Data Spanner fetches the interleaved rows at instance creation. To fetch the rows lazily, when you access the property, use `  @Interleaved(lazy = true)  ` .

Example: If a `  Singer  ` entity can have interleaved `  Album  ` entries as children, add a `  List<Album>  ` property to the `  Singer  ` entity. Also, add an `  @Interleaved  ` annotation to the property.

`  @NotMapped  `

Indicates that a property is not stored in the database and should be ignored.

`  @PrimaryKey  `

`  @PrimaryKey(keyOrder = N )  `

Indicates that the property is a component of the primary key, and identifies the position of the property within the primary key, starting at 1. The default `  keyOrder  ` is `  1  ` .

Example: `  @PrimaryKey(keyOrder = 3)  `

`  @Table(name = " TABLE_NAME ")  `

The table that the entity models. Each instance of the entity represents a record in the table. Replace `  TABLE_NAME  ` with the name of your table.

Example: `  @Table(name = "Singers")  `

If you need to model more complex relationships, see the [Spring Data Spanner reference](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/spanner.html) for details about other annotations that the module supports.

The following examples show one way to model the `  Singers  ` and `  Albums  ` tables for Spring Data Spanner:

  - For `  Singer  ` entities, the example includes an `  albums  ` property, with an `  @Interleaved  ` annotation. This property contains a list of albums that are interleaved with the `  Singer  ` entity. Spring Data Spanner populates this property automatically.
  - For `  Album  ` entities, the example includes a `  relatedAlbums  ` property that is not stored in Spanner.

<!-- end list -->

``` java
import com.google.cloud.spring.data.spanner.core.mapping.Interleaved;
import com.google.cloud.spring.data.spanner.core.mapping.PrimaryKey;
import com.google.cloud.spring.data.spanner.core.mapping.Table;
import java.util.Date;
import java.util.List;


/**
 * An entity and table holding singers.
 */
@Table(name = "Singers")
public class Singer {
  @PrimaryKey
  long singerId;

  String firstName;

  String lastName;

  Date birthDate;

  @Interleaved
  List<Album> albums;
}
```

``` java
import com.google.cloud.spring.data.spanner.core.mapping.NotMapped;
import com.google.cloud.spring.data.spanner.core.mapping.PrimaryKey;
import com.google.cloud.spring.data.spanner.core.mapping.Table;
import java.util.List;

/**
 * An entity class representing an Album.
 */
@Table(name = "Albums")
public class Album {

  @PrimaryKey
  long singerId;

  @PrimaryKey(keyOrder = 2)
  long albumId;

  String albumTitle;

  long marketingBudget;

  @NotMapped
  List<Album> relatedAlbums;

  public Album(long singerId, long albumId, String albumTitle, long marketingBudget) {
    this.singerId = singerId;
    this.albumId = albumId;
    this.albumTitle = albumTitle;
    this.marketingBudget = marketingBudget;
  }
}
```

## Query and modify data

To query and modify data with Spring Data Spanner, you can acquire a [`  SpannerTemplate  `](https://github.com/GoogleCloudPlatform/spring-cloud-gcp/blob/master/spring-cloud-gcp-data-spanner/src/main/java/com/google/cloud/spring/data/spanner/core/SpannerTemplate.java) bean, which implements [`  SpannerOperations  `](https://github.com/GoogleCloudPlatform/spring-cloud-gcp/blob/master/spring-cloud-gcp-data-spanner/src/main/java/com/google/cloud/spring/data/spanner/core/SpannerOperations.java) . `  SpannerTemplate  ` provides methods for performing [SQL queries](/spanner/docs/reference/standard-sql/query-syntax) and modifying data with [Data Manipulation Language (DML) statements](/spanner/docs/dml-tasks) . You can also use this bean to access the [read API](/spanner/docs/reads) and [mutation API](/spanner/docs/modify-mutation-api) for Spanner.

In addition, you can extend the [`  SpannerRepository  `](https://github.com/GoogleCloudPlatform/spring-cloud-gcp/blob/master/spring-cloud-gcp-data-spanner/src/main/java/com/google/cloud/spring/data/spanner/repository/SpannerRepository.java) interface to encapsulate all of the application logic that queries and modifies data in Spanner.

The following sections explain how to work with `  SpannerTemplate  ` and `  SpannerRepository  ` .

### Acquire a template bean

Use the `  @Autowired  ` annotation to acquire a `  SpannerTemplate  ` bean automatically. You can then use the `  SpannerTemplate  ` throughout your class.

The following example shows a class that acquires and uses the bean:

``` java
import com.google.cloud.spanner.KeySet;
import com.google.cloud.spanner.Statement;
import com.google.cloud.spring.data.spanner.core.SpannerQueryOptions;
import com.google.cloud.spring.data.spanner.core.SpannerTemplate;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * A quick start code for Spring Data Cloud Spanner. It demonstrates how to use SpannerTemplate to
 * execute DML and SQL queries, save POJOs, and read entities.
 */
@Component
public class SpannerTemplateSample {

  @Autowired
  SpannerTemplate spannerTemplate;

  public void runTemplateExample(Singer singer) {
    // Delete all of the rows in the Singer table.
    this.spannerTemplate.delete(Singer.class, KeySet.all());

    // Insert a singer into the Singers table.
    this.spannerTemplate.insert(singer);

    // Read all of the singers in the Singers table.
    List<Singer> allSingers = this.spannerTemplate
        .query(Singer.class, Statement.of("SELECT * FROM Singers"),
                new SpannerQueryOptions().setAllowPartialRead(true));
  }

}
```

You can use the `  SpannerTemplate  ` bean to execute [read-only transactions](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/#read-only-transaction) and [read-write transactions](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/#readwrite-transaction) . In addition, you can use the [`  @Transactional  `](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/#declarative-transactions-with-transactional-annotation) annotation to create declarative transactions.

### Acquire a repository bean

If you use a `  SpannerRepository  ` , you can use the `  @Autowired  ` annotation to acquire a bean that implements your repository's interface. A repository includes methods for running Java functions as [read-only transactions](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/#read-only-transaction) and [read-write transactions](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/#readwrite-transaction) . For lower-level operations, you can get the template bean that the repository uses.

The following examples show the interface for a repository and a class that acquires and uses the bean:

``` java
import com.google.cloud.spanner.Key;
import com.google.cloud.spring.data.spanner.repository.SpannerRepository;
import com.google.cloud.spring.data.spanner.repository.query.Query;
import java.util.List;
import org.springframework.data.repository.query.Param;


/**
 * An interface of various Query Methods. The behavior of the queries is defined only by
 * their names, arguments, or annotated SQL strings. The implementation of these functions
 * is generated by Spring Data Cloud Spanner.
 */
public interface SingerRepository extends SpannerRepository<Singer, Key> {
  List<Singer> findByLastName(String lastName);

  int countByFirstName(String firstName);

  int deleteByLastName(String lastName);

  List<Singer> findTop3DistinctByFirstNameAndSingerIdIgnoreCaseOrLastNameOrderByLastNameDesc(
      String firstName, String lastName, long singerId);

  @Query("SELECT * FROM Singers WHERE firstName LIKE '%@fragment';")
  List<Singer> getByQuery(@Param("fragment") String firstNameFragment);
}
```

``` java
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * A quick start code for Spring Data Cloud Spanner.
 * It demonstrates how to use a SpannerRepository to execute read-write queries
 * generated from interface definitions.
 *
 */
@Component
public class SpannerRepositorySample {

  @Autowired
  SingerRepository singerRepository;

  public void runRepositoryExample() {
    List<Singer> lastNameSingers = this.singerRepository.findByLastName("a last name");

    int fistNameCount = this.singerRepository.countByFirstName("a first name");

    int deletedLastNameCount = this.singerRepository.deleteByLastName("a last name");
  }

}
```

## Manage Spanner

To get information about your Spanner databases, update a schema with a Data Definition Language (DDL) statement, or complete other administrative tasks, you can acquire a [`  SpannerDatabaseAdminTemplate  `](https://github.com/GoogleCloudPlatform/spring-cloud-gcp/blob/master/spring-cloud-gcp-data-spanner/src/main/java/com/google/cloud/spring/data/spanner/core/admin/SpannerDatabaseAdminTemplate.java) bean.

Use the `  @Autowired  ` annotation to acquire the bean automatically. You can then use the `  SpannerDatabaseAdminTemplate  ` throughout your class.

The following example shows a class that acquires and uses the bean:

``` java
import com.google.cloud.spring.data.spanner.core.admin.SpannerDatabaseAdminTemplate;
import com.google.cloud.spring.data.spanner.core.admin.SpannerSchemaUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * This sample demonstrates how to generate schemas for interleaved tables from POJOs and how to
 * execute DDL.
 */
@Component
public class SpannerSchemaToolsSample {

  @Autowired
  SpannerDatabaseAdminTemplate spannerDatabaseAdminTemplate;

  @Autowired
  SpannerSchemaUtils spannerSchemaUtils;

  /**
   * Creates the Singers table. Also creates the Albums table, because Albums is interleaved with
   * Singers.
   */
  public void createTableIfNotExists() {
    if (!this.spannerDatabaseAdminTemplate.tableExists("Singers")) {
      this.spannerDatabaseAdminTemplate.executeDdlStrings(
          this.spannerSchemaUtils
              .getCreateTableDdlStringsForInterleavedHierarchy(Singer.class),
          true);
    }
  }

  /**
   * Drops both the Singers and Albums tables using just a reference to the Singer entity type ,
   * because they are interleaved.
   */
  public void dropTables() {
    if (this.spannerDatabaseAdminTemplate.tableExists("Singers")) {
      this.spannerDatabaseAdminTemplate.executeDdlStrings(
          this.spannerSchemaUtils.getDropTableDdlStringsForInterleavedHierarchy(Singer.class),
          false);
    }
  }
}
```

## What's next

  - Get started with [Spring Cloud GCP](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/#getting-started) .
  - Learn more about [using Spring Data Spanner in your applications](https://docs.spring.io/spring-cloud-gcp/docs/current/reference/html/spanner.html) .
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/spring-cloud-gcp/issues) to report a bug or ask a question about the module.
  - Get more information about [Spring Framework support on Google Cloud](/java/docs/reference/spring) .
  - Try a codelab to [deploy and run an application that uses Spring Cloud GCP](https://codelabs.developers.google.com/spring/) .
