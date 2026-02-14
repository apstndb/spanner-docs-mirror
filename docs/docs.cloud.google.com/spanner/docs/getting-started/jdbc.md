## Objectives

This tutorial walks you through the following steps using the Spanner JDBC driver:

  - Create a Spanner instance and database.
  - Write, read, and execute SQL queries on data in the database.
  - Update the database schema.
  - Update data using a read-write transaction.
  - Add a secondary index to the database.
  - Use the index to read and execute SQL queries on data.
  - Retrieve data using a read-only transaction.

## Costs

This tutorial uses Spanner, which is a billable component of the Google Cloud. For information on the cost of using Spanner, see [Pricing](https://cloud.google.com/spanner/pricing) .

## Before you begin

Complete the steps described in [Set up](/spanner/docs/getting-started/set-up#set_up_a_project) , which cover creating and setting a default Google Cloud project, enabling billing, enabling the Cloud Spanner API, and setting up OAuth 2.0 to get authentication credentials to use the Cloud Spanner API.

In particular, make sure that you run [`  gcloud auth application-default login  `](/sdk/gcloud/reference/auth/application-default/login) to set up your local development environment with authentication credentials.

**Note:** If you don't plan to keep the resources that you create in this tutorial, consider creating a new Google Cloud project instead of selecting an existing project. After you finish the tutorial, you can delete the project, removing all resources associated with the project.

## Prepare your local JDBC environment

1.  Install the following on your development machine if they are not already installed:
    
      - Java 8 JDK ( [download](http://openjdk.java.net/) ).
      - Maven 3 ( [download](https://maven.apache.org/download.cgi) ).

2.  Clone the sample app repository to your local machine:
    
    ``` text
    git clone https://github.com/googleapis/java-spanner-jdbc.git
    ```

3.  Change to the directory that contains the Spanner sample code:
    
    ``` text
    cd java-spanner-jdbc/samples/snippets
    ```

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with JDBC.

The `  pom.xml  ` adds the Spanner JDBC driver to the project's dependencies and configures the assembly plugin to build an executable JAR file with the Java class defined in this tutorial.

Build the sample from the [`  samples/snippets  ` directory](https://github.com/googleapis/java-spanner-jdbc/blob/-/samples/snippets) :

``` text
mvn package -DskipTests
```

## Create a database

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
createdatabase test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
createpgdatabase test-instance example-db
```

You should see:

``` text
Created database [projects/my-project/instances/test-instance/databases/example-db]
```

The following code creates a database and two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

``` java
static void createDatabase(DatabaseAdminClient dbAdminClient,
    InstanceName instanceName, String databaseId) {
  CreateDatabaseRequest createDatabaseRequest =
      CreateDatabaseRequest.newBuilder()
          .setCreateStatement("CREATE DATABASE `" + databaseId + "`")
          .setParent(instanceName.toString())
          .addAllExtraStatements(Arrays.asList(
              "CREATE TABLE Singers ("
                  + "  SingerId   INT64 NOT NULL,"
                  + "  FirstName  STRING(1024),"
                  + "  LastName   STRING(1024),"
                  + "  SingerInfo BYTES(MAX),"
                  + "  FullName STRING(2048) AS "
                  + "  (ARRAY_TO_STRING([FirstName, LastName], \" \")) STORED"
                  + ") PRIMARY KEY (SingerId)",
              "CREATE TABLE Albums ("
                  + "  SingerId     INT64 NOT NULL,"
                  + "  AlbumId      INT64 NOT NULL,"
                  + "  AlbumTitle   STRING(MAX)"
                  + ") PRIMARY KEY (SingerId, AlbumId),"
                  + "  INTERLEAVE IN PARENT Singers ON DELETE CASCADE")).build();
  try {
    // Initiate the request which returns an OperationFuture.
    com.google.spanner.admin.database.v1.Database db =
        dbAdminClient.createDatabaseAsync(createDatabaseRequest).get();
    System.out.println("Created database [" + db.getName() + "]");
  } catch (ExecutionException e) {
    // If the operation failed during execution, expose the cause.
    throw (SpannerException) e.getCause();
  } catch (InterruptedException e) {
    // Throw when a thread is waiting, sleeping, or otherwise occupied,
    // and the thread is interrupted, either before or during the activity.
    throw SpannerExceptionFactory.propagateInterrupt(e);
  }
}
```

### PostgreSQL

``` java
static void createPostgreSqlDatabase(
    DatabaseAdminClient dbAdminClient, String projectId, String instanceId, String databaseId) {
  final CreateDatabaseRequest request =
      CreateDatabaseRequest.newBuilder()
          .setCreateStatement("CREATE DATABASE \"" + databaseId + "\"")
          .setParent(InstanceName.of(projectId, instanceId).toString())
          .setDatabaseDialect(DatabaseDialect.POSTGRESQL).build();

  try {
    // Initiate the request which returns an OperationFuture.
    Database db = dbAdminClient.createDatabaseAsync(request).get();
    System.out.println("Created database [" + db.getName() + "]");
  } catch (ExecutionException e) {
    // If the operation failed during execution, expose the cause.
    throw (SpannerException) e.getCause();
  } catch (InterruptedException e) {
    // Throw when a thread is waiting, sleeping, or otherwise occupied,
    // and the thread is interrupted, either before or during the activity.
    throw SpannerExceptionFactory.propagateInterrupt(e);
  }
}
static void createTableUsingDdl(DatabaseAdminClient dbAdminClient, DatabaseName databaseName) {
  try {
    // Initiate the request which returns an OperationFuture.
    dbAdminClient.updateDatabaseDdlAsync(
        databaseName,
        Arrays.asList(
            "CREATE TABLE Singers ("
                + "  SingerId   bigint NOT NULL,"
                + "  FirstName  character varying(1024),"
                + "  LastName   character varying(1024),"
                + "  SingerInfo bytea,"
                + "  FullName character varying(2048) GENERATED "
                + "  ALWAYS AS (FirstName || ' ' || LastName) STORED,"
                + "  PRIMARY KEY (SingerId)"
                + ")",
            "CREATE TABLE Albums ("
                + "  SingerId     bigint NOT NULL,"
                + "  AlbumId      bigint NOT NULL,"
                + "  AlbumTitle   character varying(1024),"
                + "  PRIMARY KEY (SingerId, AlbumId)"
                + ") INTERLEAVE IN PARENT Singers ON DELETE CASCADE")).get();
    System.out.println("Created Singers & Albums tables in database: [" + databaseName + "]");
  } catch (ExecutionException e) {
    // If the operation failed during execution, expose the cause.
    throw SpannerExceptionFactory.asSpannerException(e);
  } catch (InterruptedException e) {
    // Throw when a thread is waiting, sleeping, or otherwise occupied,
    // and the thread is interrupted, either before or during the activity.
    throw SpannerExceptionFactory.propagateInterrupt(e);
  }
}
```

The next step is to write data to your database.

## Create a JDBC connection

Before you can do reads or writes, you must create a [`  Connection  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html) . All of your interactions with Spanner must go through a `  Connection  ` . The database name and other properties are specified in the JDBC connection URL and the `  java.util.Properties  ` set.

### GoogleSQL

``` java
static void createConnection(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  // Connection properties can be specified both with in a Properties object
  // and in the connection URL.
  properties.put("numChannels", "8");
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s"
                  + ";minSessions=400;maxSessions=400",
              project, instance, database),
          properties)) {
    try (ResultSet resultSet =
        connection.createStatement().executeQuery("select 'Hello World!'")) {
      while (resultSet.next()) {
        System.out.println(resultSet.getString(1));
      }
    }
  }
}
```

### PostgreSQL

``` java
static void createConnection(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  // Connection properties can be specified both with in a Properties object
  // and in the connection URL.
  properties.put("numChannels", "8");
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s"
                  + ";minSessions=400;maxSessions=400",
              project, instance, database),
          properties)) {
    try (ResultSet resultSet =
        connection.createStatement().executeQuery("select 'Hello World!'")) {
      while (resultSet.next()) {
        System.out.println(resultSet.getString(1));
      }
    }
  }
}
```

For a full list of supported properties, see [Connection URL Properties](https://github.com/googleapis/java-spanner-jdbc?tab=readme-ov-file#connection-url-properties) .

Each `  Connection  ` uses resources, so it is good practice to either close connections when they are no longer needed, or to use a connection pool to re-use connections throughout your application.

Read more in the [`  Connection  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html) Javadoc reference.

### Connect the JDBC driver to the emulator

You can connect the JDBC driver to the Spanner emulator in two ways:

  - Set the `  SPANNER_EMULATOR_HOST  ` environment variable: This instructs the JDBC driver to connect to the emulator. The Spanner instance and database in the JDBC connection URL must already exist on the emulator.
  - Add `  autoConfigEmulator=true  ` to the connection URL: This instructs the JDBC driver to connect to the emulator, and to automatically create the Spanner instance and database in the JDBC connection URL if these don't exist.

This example shows how to use the `  autoConfigEmulator=true  ` connection URL option.

### GoogleSQL

``` java
static void createConnectionWithEmulator(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  // Add autoConfigEmulator=true to the connection URL to instruct the JDBC
  // driver to connect to the Spanner emulator on localhost:9010.
  // The Spanner instance and database are automatically created if these
  // don't already exist.
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s"
                  + ";autoConfigEmulator=true",
              project, instance, database),
          properties)) {
    try (ResultSet resultSet =
        connection.createStatement().executeQuery("select 'Hello World!'")) {
      while (resultSet.next()) {
        System.out.println(resultSet.getString(1));
      }
    }
  }
}
```

### PostgreSQL

``` java
static void createConnectionWithEmulator(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  // Add autoConfigEmulator=true to the connection URL to instruct the JDBC
  // driver to connect to the Spanner emulator on localhost:9010.
  // The Spanner instance and database are automatically created if these
  // don't already exist.
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s"
                  + ";autoConfigEmulator=true",
              project, instance, database),
          properties)) {
    try (ResultSet resultSet =
        connection.createStatement().executeQuery("select 'Hello World!'")) {
      while (resultSet.next()) {
        System.out.println(resultSet.getString(1));
      }
    }
  }
}
```

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  PreparedStatement.executeUpdate()  ` method to execute a DML statement.

### GoogleSQL

``` java
static void writeDataWithDml(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Add 4 rows in one statement.
    // JDBC always uses '?' as a parameter placeholder.
    try (PreparedStatement preparedStatement =
        connection.prepareStatement(
            "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES "
                + "(?, ?, ?), "
                + "(?, ?, ?), "
                + "(?, ?, ?), "
                + "(?, ?, ?)")) {

      final ImmutableList<Singer> singers =
          ImmutableList.of(
              new Singer(/* SingerId = */ 12L, "Melissa", "Garcia"),
              new Singer(/* SingerId = */ 13L, "Russel", "Morales"),
              new Singer(/* SingerId = */ 14L, "Jacqueline", "Long"),
              new Singer(/* SingerId = */ 15L, "Dylan", "Shaw"));

      // Note that JDBC parameters start at index 1.
      int paramIndex = 0;
      for (Singer singer : singers) {
        preparedStatement.setLong(++paramIndex, singer.singerId);
        preparedStatement.setString(++paramIndex, singer.firstName);
        preparedStatement.setString(++paramIndex, singer.lastName);
      }

      int updateCount = preparedStatement.executeUpdate();
      System.out.printf("%d records inserted.\n", updateCount);
    }
  }
}
```

### PostgreSQL

``` java
static void writeDataWithDmlPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Add 4 rows in one statement.
    // JDBC always uses '?' as a parameter placeholder.
    try (PreparedStatement preparedStatement =
        connection.prepareStatement(
            "INSERT INTO singers (singer_id, first_name, last_name) VALUES "
                + "(?, ?, ?), "
                + "(?, ?, ?), "
                + "(?, ?, ?), "
                + "(?, ?, ?)")) {

      final ImmutableList<Singer> singers =
          ImmutableList.of(
              new Singer(/* SingerId = */ 12L, "Melissa", "Garcia"),
              new Singer(/* SingerId = */ 13L, "Russel", "Morales"),
              new Singer(/* SingerId = */ 14L, "Jacqueline", "Long"),
              new Singer(/* SingerId = */ 15L, "Dylan", "Shaw"));

      // Note that JDBC parameters start at index 1.
      int paramIndex = 0;
      for (Singer singer : singers) {
        preparedStatement.setLong(++paramIndex, singer.singerId);
        preparedStatement.setString(++paramIndex, singer.firstName);
        preparedStatement.setString(++paramIndex, singer.lastName);
      }

      int updateCount = preparedStatement.executeUpdate();
      System.out.printf("%d records inserted.\n", updateCount);
    }
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
writeusingdml test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
writeusingdmlpg test-instance example-db
```

You should see:

``` text
4 records inserted.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with a DML batch

You use the `  PreparedStatement#addBatch()  ` and `  PreparedStatement#executeBatch()  ` methods to execute multiple DML statements in one batch.

**Tip:** You can also execute DML batches with the [`  START BATCH DML  ` command](/spanner/docs/jdbc-session-mgmt-commands#start_batch_dml) .

### GoogleSQL

``` java
static void writeDataWithDmlBatch(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Add multiple rows in one DML batch.
    // JDBC always uses '?' as a parameter placeholder.
    try (PreparedStatement preparedStatement =
        connection.prepareStatement(
            "INSERT INTO Singers (SingerId, FirstName, LastName) "
                + "VALUES (?, ?, ?)")) {
      final ImmutableList<Singer> singers =
          ImmutableList.of(
              new Singer(/* SingerId = */ 16L, "Sarah", "Wilson"),
              new Singer(/* SingerId = */ 17L, "Ethan", "Miller"),
              new Singer(/* SingerId = */ 18L, "Maya", "Patel"));

      for (Singer singer : singers) {
        // Note that JDBC parameters start at index 1.
        int paramIndex = 0;
        preparedStatement.setLong(++paramIndex, singer.singerId);
        preparedStatement.setString(++paramIndex, singer.firstName);
        preparedStatement.setString(++paramIndex, singer.lastName);
        preparedStatement.addBatch();
      }

      int[] updateCounts = preparedStatement.executeBatch();
      System.out.printf(
          "%d records inserted.\n",
          Arrays.stream(updateCounts).sum());
    }
  }
}
```

### PostgreSQL

``` java
static void writeDataWithDmlBatchPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Add multiple rows in one DML batch.
    // JDBC always uses '?' as a parameter placeholder.
    try (PreparedStatement preparedStatement =
        connection.prepareStatement(
            "INSERT INTO singers (singer_id, first_name, last_name)"
                + " VALUES (?, ?, ?)")) {
      final ImmutableList<Singer> singers =
          ImmutableList.of(
              new Singer(/* SingerId = */ 16L, "Sarah", "Wilson"),
              new Singer(/* SingerId = */ 17L, "Ethan", "Miller"),
              new Singer(/* SingerId = */ 18L, "Maya", "Patel"));

      for (Singer singer : singers) {
        // Note that JDBC parameters start at index 1.
        int paramIndex = 0;
        preparedStatement.setLong(++paramIndex, singer.singerId);
        preparedStatement.setString(++paramIndex, singer.firstName);
        preparedStatement.setString(++paramIndex, singer.lastName);
        preparedStatement.addBatch();
      }

      int[] updateCounts = preparedStatement.executeBatch();
      System.out.printf(
          "%d records inserted.\n",
          Arrays.stream(updateCounts).sum());
    }
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
writeusingdmlbatch test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
writeusingdmlbatchpg test-instance example-db
```

You should see:

``` text
3 records inserted.
```

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

You can write data using a [`  Mutation  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation) object. A `  Mutation  ` object is a container for mutation operations. A `  Mutation  ` represents a sequence of inserts, updates, and deletes that Spanner applies atomically to different rows and tables in a Spanner database.

The [`  newInsertBuilder()  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation#com_google_cloud_spanner_Mutation_newInsertBuilder_java_lang_String_) method in the `  Mutation  ` class constructs an `  INSERT  ` mutation, which inserts a new row in a table. If the row already exists, the write fails. Alternatively, you can use the [`  newInsertOrUpdateBuilder  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.Mutation#com_google_cloud_spanner_Mutation_newInsertOrUpdateBuilder_java_lang_String_) method to construct an `  INSERT_OR_UPDATE  ` mutation, which updates column values if the row already exists.

The [`  write()  `](/java/docs/reference/google-cloud-spanner-jdbc/latest/com.google.cloud.spanner.jdbc.CloudSpannerJdbcConnection#com_google_cloud_spanner_jdbc_CloudSpannerJdbcConnection_write_com_google_cloud_spanner_Mutation_) method in the `  CloudSpannerJdbcConnection  ` interface writes the mutations. All mutations in a single batch are applied atomically.

You can unwrap the `  CloudSpannerJdbcConnection  ` interface from a Spanner JDBC `  Connection  ` .

This code shows how to write the data using mutations:

### GoogleSQL

``` java
/** The list of Singers to insert. */
static final List<Singer> SINGERS =
    Arrays.asList(
        new Singer(1, "Marc", "Richards"),
        new Singer(2, "Catalina", "Smith"),
        new Singer(3, "Alice", "Trentor"),
        new Singer(4, "Lea", "Martin"),
        new Singer(5, "David", "Lomond"));

/** The list of Albums to insert. */
static final List<Album> ALBUMS =
    Arrays.asList(
        new Album(1, 1, "Total Junk"),
        new Album(1, 2, "Go, Go, Go"),
        new Album(2, 1, "Green"),
        new Album(2, 2, "Forever Hold Your Peace"),
        new Album(2, 3, "Terrified"));

static void writeDataWithMutations(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Unwrap the CloudSpannerJdbcConnection interface
    // from the java.sql.Connection.
    CloudSpannerJdbcConnection cloudSpannerJdbcConnection =
        connection.unwrap(CloudSpannerJdbcConnection.class);

    List<Mutation> mutations = new ArrayList<>();
    for (Singer singer : SINGERS) {
      mutations.add(
          Mutation.newInsertBuilder("Singers")
              .set("SingerId")
              .to(singer.singerId)
              .set("FirstName")
              .to(singer.firstName)
              .set("LastName")
              .to(singer.lastName)
              .build());
    }
    for (Album album : ALBUMS) {
      mutations.add(
          Mutation.newInsertBuilder("Albums")
              .set("SingerId")
              .to(album.singerId)
              .set("AlbumId")
              .to(album.albumId)
              .set("AlbumTitle")
              .to(album.albumTitle)
              .build());
    }
    // Apply the mutations atomically to Spanner.
    cloudSpannerJdbcConnection.write(mutations);
    System.out.printf("Inserted %d rows.\n", mutations.size());
  }
}
```

### PostgreSQL

``` java
/** The list of Singers to insert. */
static final List<Singer> SINGERS =
    Arrays.asList(
        new Singer(1, "Marc", "Richards"),
        new Singer(2, "Catalina", "Smith"),
        new Singer(3, "Alice", "Trentor"),
        new Singer(4, "Lea", "Martin"),
        new Singer(5, "David", "Lomond"));

/** The list of Albums to insert. */
static final List<Album> ALBUMS =
    Arrays.asList(
        new Album(1, 1, "Total Junk"),
        new Album(1, 2, "Go, Go, Go"),
        new Album(2, 1, "Green"),
        new Album(2, 2, "Forever Hold Your Peace"),
        new Album(2, 3, "Terrified"));

static void writeDataWithMutationsPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Unwrap the CloudSpannerJdbcConnection interface
    // from the java.sql.Connection.
    CloudSpannerJdbcConnection cloudSpannerJdbcConnection =
        connection.unwrap(CloudSpannerJdbcConnection.class);

    List<Mutation> mutations = new ArrayList<>();
    for (Singer singer : SINGERS) {
      mutations.add(
          Mutation.newInsertBuilder("singers")
              .set("singer_id")
              .to(singer.singerId)
              .set("first_name")
              .to(singer.firstName)
              .set("last_name")
              .to(singer.lastName)
              .build());
    }
    for (Album album : ALBUMS) {
      mutations.add(
          Mutation.newInsertBuilder("albums")
              .set("singer_id")
              .to(album.singerId)
              .set("album_id")
              .to(album.albumId)
              .set("album_title")
              .to(album.albumTitle)
              .build());
    }
    // Apply the mutations atomically to Spanner.
    cloudSpannerJdbcConnection.write(mutations);
    System.out.printf("Inserted %d rows.\n", mutations.size());
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
write test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
writepg test-instance example-db
```

You should see:

``` text
Inserted 10 rows.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner JDBC driver.

### On the command line

Execute the following SQL statement to read the values of all columns from the `  Albums  ` table:

### GoogleSQL

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql='SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
```

### PostgreSQL

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql='SELECT singer_id, album_id, album_title FROM albums'
```

**Note:** For the GoogleSQL reference, see [Query syntax in GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax) and for PostgreSQL reference, see [PostgreSQL lexical structure and syntax](/spanner/docs/reference/postgresql/lexical) .

The result shows:

``` text
SingerId AlbumId AlbumTitle
1        1       Total Junk
1        2       Go, Go, Go
2        1       Green
2        2       Forever Hold Your Peace
2        3       Terrified
```

### Use the Spanner JDBC driver

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner JDBC driver.

The following methods and classes are used to run the SQL query:

  - The [`  createStatement()  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html#createStatement--) method in the `  Connection  ` interface: use this to create a new statement object for running a SQL statement.
  - The [`  executeQuery(String)  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Statement.html#executeQuery-java.lang.String-) method of the `  Statement  ` class: use this method to execute a query against a database.
  - The [`  Statement  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Statement.html) class: use this to execute a SQL string.
  - The [`  ResultSet  `](https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSet.html) class: use this to access the data returned by a SQL statement.

Here's how to issue the query and access the data:

### GoogleSQL

``` java
static void queryData(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "SELECT SingerId, AlbumId, AlbumTitle "
                + "FROM Albums")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            resultSet.getLong("SingerId"),
            resultSet.getLong("AlbumId"),
            resultSet.getString("AlbumTitle"));
      }
    }
  }
}
```

### PostgreSQL

``` java
static void queryDataPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "SELECT singer_id, album_id, album_title "
                    + "FROM albums")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            resultSet.getLong("singer_id"),
            resultSet.getLong("album_id"),
            resultSet.getString("album_title"));
      }
    }
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
query test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
querypg test-instance example-db
```

You should see the following result:

``` text
1 1 Total Junk
1 2 Go, Go, Go
2 1 Green
2 2 Forever Hold Your Peace
2 3 Terrified
```

### Query using a SQL parameter

If your application has a frequently executed query, you can improve its performance by parameterizing it. The resulting parametric query can be cached and reused, which reduces compilation costs. For more information, see [Use query parameters to speed up frequently executed queries](/spanner/docs/sql-best-practices#query-parameters) .

Here is an example of using a parameter in the `  WHERE  ` clause to query records containing a specific value for `  LastName  ` .

Use a [`  java.sql.PreparedStatement  `](https://docs.oracle.com/javase/8/docs/api/java/sql/PreparedStatement.html) to execute a query with a parameter.

### GoogleSQL

``` java
static void queryWithParameter(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    try (PreparedStatement statement =
        connection.prepareStatement(
            "SELECT SingerId, FirstName, LastName "
                + "FROM Singers "
                + "WHERE LastName = ?")) {
      statement.setString(1, "Garcia");
      try (ResultSet resultSet = statement.executeQuery()) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %s %s\n",
              resultSet.getLong("SingerId"),
              resultSet.getString("FirstName"),
              resultSet.getString("LastName"));
        }
      }
    }
  }
}
```

### PostgreSQL

``` java
static void queryWithParameterPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    try (PreparedStatement statement =
        connection.prepareStatement(
            "SELECT singer_id, first_name, last_name "
                + "FROM singers "
                + "WHERE last_name = ?")) {
      statement.setString(1, "Garcia");
      try (ResultSet resultSet = statement.executeQuery()) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %s %s\n",
              resultSet.getLong("singer_id"),
              resultSet.getString("first_name"),
              resultSet.getString("last_name"));
        }
      }
    }
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
querywithparameter test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
querywithparameterpg test-instance example-db
```

You should see the following result:

``` text
12 Melissa Garcia
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner JDBC driver driver.

#### On the command line

Use the following [`  ALTER TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) command to add the new column to the table:

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget INT64'
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE albums ADD COLUMN marketing_budget BIGINT'
```

You should see:

``` text
Schema updating...done.
```

#### Use the Spanner JDBC driver

Use the [`  execute(String)  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Statement.html#execute-java.lang.String-) method of the `  java.sql.Statement  ` class to modify the schema:

### GoogleSQL

``` java
static void addColumn(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    connection
        .createStatement()
        .execute("ALTER TABLE Albums ADD COLUMN MarketingBudget INT64");
    System.out.println("Added MarketingBudget column");
  }
}
```

### PostgreSQL

``` java
static void addColumnPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    connection
        .createStatement()
        .execute("alter table albums add column marketing_budget bigint");
    System.out.println("Added marketing_budget column");
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
addmarketingbudget test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
addmarketingbudgetpg test-instance example-db
```

You should see:

``` text
Added MarketingBudget column.
```

### Execute a DDL batch

We recommend that you execute multiple schema modifications in one batch. Use the [`  addBatch(String)  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Statement.html#addBatch-java.lang.String-) method of `  java.sql.Statement  ` to add multiple DDL statements to a batch.

**Tip:** You can also execute DDL batches with the [`  START BATCH DDL  ` command](/spanner/docs/jdbc-session-mgmt-commands#start_batch_ddl) .

### GoogleSQL

``` java
static void ddlBatch(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    try (Statement statement = connection.createStatement()) {
      // Create two new tables in one batch.
      statement.addBatch(
          "CREATE TABLE Venues ("
              + "  VenueId     INT64 NOT NULL,"
              + "  Name        STRING(1024),"
              + "  Description JSON"
              + ") PRIMARY KEY (VenueId)");
      statement.addBatch(
          "CREATE TABLE Concerts ("
              + "  ConcertId INT64 NOT NULL,"
              + "  VenueId   INT64 NOT NULL,"
              + "  SingerId  INT64 NOT NULL,"
              + "  StartTime TIMESTAMP,"
              + "  EndTime   TIMESTAMP,"
              + "  CONSTRAINT Fk_Concerts_Venues FOREIGN KEY"
              + "    (VenueId) REFERENCES Venues (VenueId),"
              + "  CONSTRAINT Fk_Concerts_Singers FOREIGN KEY"
              + "    (SingerId) REFERENCES Singers (SingerId),"
              + ") PRIMARY KEY (ConcertId)");
      statement.executeBatch();
    }
    System.out.println("Added Venues and Concerts tables");
  }
}
```

### PostgreSQL

``` java
static void ddlBatchPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    try (Statement statement = connection.createStatement()) {
      // Create two new tables in one batch.
      statement.addBatch(
          "CREATE TABLE venues ("
              + "  venue_id    bigint not null primary key,"
              + "  name        varchar(1024),"
              + "  description jsonb"
              + ")");
      statement.addBatch(
          "CREATE TABLE concerts ("
              + "  concert_id bigint not null primary key ,"
              + "  venue_id   bigint not null,"
              + "  singer_id  bigint not null,"
              + "  start_time timestamptz,"
              + "  end_time   timestamptz,"
              + "  constraint fk_concerts_venues foreign key"
              + "    (venue_id) references venues (venue_id),"
              + "  constraint fk_concerts_singers foreign key"
              + "    (singer_id) references singers (singer_id)"
              + ")");
      statement.executeBatch();
    }
    System.out.println("Added venues and concerts tables");
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
ddlbatch test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
ddlbatchpg test-instance example-db
```

You should see:

``` text
Added Venues and Concerts tables.
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

### GoogleSQL

``` java
static void updateDataWithMutations(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Unwrap the CloudSpannerJdbcConnection interface
    // from the java.sql.Connection.
    CloudSpannerJdbcConnection cloudSpannerJdbcConnection =
        connection.unwrap(CloudSpannerJdbcConnection.class);

    final long marketingBudgetAlbum1 = 100000L;
    final long marketingBudgetAlbum2 = 500000L;
    // Mutation can be used to update/insert/delete a single row in a table.
    // Here we use newUpdateBuilder to create update mutations.
    List<Mutation> mutations =
        Arrays.asList(
            Mutation.newUpdateBuilder("Albums")
                .set("SingerId")
                .to(1)
                .set("AlbumId")
                .to(1)
                .set("MarketingBudget")
                .to(marketingBudgetAlbum1)
                .build(),
            Mutation.newUpdateBuilder("Albums")
                .set("SingerId")
                .to(2)
                .set("AlbumId")
                .to(2)
                .set("MarketingBudget")
                .to(marketingBudgetAlbum2)
                .build());
    // This writes all the mutations to Cloud Spanner atomically.
    cloudSpannerJdbcConnection.write(mutations);
    System.out.println("Updated albums");
  }
}
```

### PostgreSQL

``` java
static void updateDataWithMutationsPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Unwrap the CloudSpannerJdbcConnection interface
    // from the java.sql.Connection.
    CloudSpannerJdbcConnection cloudSpannerJdbcConnection =
        connection.unwrap(CloudSpannerJdbcConnection.class);

    final long marketingBudgetAlbum1 = 100000L;
    final long marketingBudgetAlbum2 = 500000L;
    // Mutation can be used to update/insert/delete a single row in a table.
    // Here we use newUpdateBuilder to create update mutations.
    List<Mutation> mutations =
        Arrays.asList(
            Mutation.newUpdateBuilder("albums")
                .set("singer_id")
                .to(1)
                .set("album_id")
                .to(1)
                .set("marketing_budget")
                .to(marketingBudgetAlbum1)
                .build(),
            Mutation.newUpdateBuilder("albums")
                .set("singer_id")
                .to(2)
                .set("album_id")
                .to(2)
                .set("marketing_budget")
                .to(marketingBudgetAlbum2)
                .build());
    // This writes all the mutations to Cloud Spanner atomically.
    cloudSpannerJdbcConnection.write(mutations);
    System.out.println("Updated albums");
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
update test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
updatepg test-instance example-db
```

You should see output similar to this:

``` text
Updated albums
```

You can also execute a SQL query or a read call to fetch the values that you just wrote.

Here's the code to execute the query:

### GoogleSQL

``` java
static void queryDataWithNewColumn(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Rows without an explicit value for MarketingBudget will have a
    // MarketingBudget equal to null.
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "SELECT SingerId, AlbumId, MarketingBudget "
                + "FROM Albums")) {
      while (resultSet.next()) {
        // Use the ResultSet#getObject(String) method to get data
        // of any type from the ResultSet.
        System.out.printf(
            "%s %s %s\n",
            resultSet.getObject("SingerId"),
            resultSet.getObject("AlbumId"),
            resultSet.getObject("MarketingBudget"));
      }
    }
  }
}
```

### PostgreSQL

``` java
static void queryDataWithNewColumnPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Rows without an explicit value for marketing_budget will have a
    // marketing_budget equal to null.
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "select singer_id, album_id, marketing_budget "
                    + "from albums")) {
      while (resultSet.next()) {
        // Use the ResultSet#getObject(String) method to get data
        // of any type from the ResultSet.
        System.out.printf(
            "%s %s %s\n",
            resultSet.getObject("singer_id"),
            resultSet.getObject("album_id"),
            resultSet.getObject("marketing_budget"));
      }
    }
  }
}
```

To execute this query, run the following command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
querymarketingbudget test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
querymarketingbudgetpg test-instance example-db
```

The result shows:

``` text
1 1 100000
1 2 null
2 1 null
2 2 500000
2 3 null
```

## Update data

You can update data using DML in a read-write transaction.

Set `  AutoCommit=false  ` to execute read-write transactions in JDBC.

### GoogleSQL

``` java
static void writeWithTransactionUsingDml(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Set AutoCommit=false to enable transactions.
    connection.setAutoCommit(false);

    // Transfer marketing budget from one album to another.
    // We do it in a transaction to ensure that the transfer is atomic.
    // There is no need to explicitly start the transaction. The first
    // statement on the connection will start a transaction when
    // AutoCommit=false.
    String selectMarketingBudgetSql =
        "SELECT MarketingBudget "
        + "FROM Albums "
        + "WHERE SingerId = ? AND AlbumId = ?";
    long album2Budget = 0;
    try (PreparedStatement selectMarketingBudgetStatement =
        connection.prepareStatement(selectMarketingBudgetSql)) {
      // Bind the query parameters to SingerId=2 and AlbumId=2.
      selectMarketingBudgetStatement.setLong(1, 2);
      selectMarketingBudgetStatement.setLong(2, 2);
      try (ResultSet resultSet =
          selectMarketingBudgetStatement.executeQuery()) {
        while (resultSet.next()) {
          album2Budget = resultSet.getLong("MarketingBudget");
        }
      }
      // The transaction will only be committed if this condition still holds
      // at the time of commit. Otherwise, the transaction will be aborted.
      final long transfer = 200000;
      if (album2Budget >= transfer) {
        long album1Budget = 0;
        // Re-use the existing PreparedStatement for selecting the
        // MarketingBudget to get the budget for Album 1.
        // Bind the query parameters to SingerId=1 and AlbumId=1.
        selectMarketingBudgetStatement.setLong(1, 1);
        selectMarketingBudgetStatement.setLong(2, 1);
        try (ResultSet resultSet =
            selectMarketingBudgetStatement.executeQuery()) {
          while (resultSet.next()) {
            album1Budget = resultSet.getLong("MarketingBudget");
          }
        }

        // Transfer part of the marketing budget of Album 2 to Album 1.
        album1Budget += transfer;
        album2Budget -= transfer;
        String updateSql =
            "UPDATE Albums "
                + "SET MarketingBudget = ? "
                + "WHERE SingerId = ? and AlbumId = ?";
        try (PreparedStatement updateStatement =
            connection.prepareStatement(updateSql)) {
          // Update Album 1.
          int paramIndex = 0;
          updateStatement.setLong(++paramIndex, album1Budget);
          updateStatement.setLong(++paramIndex, 1);
          updateStatement.setLong(++paramIndex, 1);
          // Create a DML batch by calling addBatch on
          // the current PreparedStatement.
          updateStatement.addBatch();

          // Update Album 2 in the same DML batch.
          paramIndex = 0;
          updateStatement.setLong(++paramIndex, album2Budget);
          updateStatement.setLong(++paramIndex, 2);
          updateStatement.setLong(++paramIndex, 2);
          updateStatement.addBatch();

          // Execute both DML statements in one batch.
          updateStatement.executeBatch();
        }
      }
    }
    // Commit the current transaction.
    connection.commit();
    System.out.println(
        "Transferred marketing budget from Album 2 to Album 1");
  }
}
```

### PostgreSQL

``` java
static void writeWithTransactionUsingDmlPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Set AutoCommit=false to enable transactions.
    connection.setAutoCommit(false);

    // Transfer marketing budget from one album to another. We do it in a
    // transaction to ensure that the transfer is atomic. There is no need
    // to explicitly start the transaction. The first statement on the
    // connection will start a transaction when AutoCommit=false.
    String selectMarketingBudgetSql =
        "SELECT marketing_budget "
            + "from albums "
            + "WHERE singer_id = ? and album_id = ?";
    long album2Budget = 0;
    try (PreparedStatement selectMarketingBudgetStatement =
        connection.prepareStatement(selectMarketingBudgetSql)) {
      // Bind the query parameters to SingerId=2 and AlbumId=2.
      selectMarketingBudgetStatement.setLong(1, 2);
      selectMarketingBudgetStatement.setLong(2, 2);
      try (ResultSet resultSet =
          selectMarketingBudgetStatement.executeQuery()) {
        while (resultSet.next()) {
          album2Budget = resultSet.getLong("marketing_budget");
        }
      }
      // The transaction will only be committed if this condition still holds
      // at the time of commit. Otherwise, the transaction will be aborted.
      final long transfer = 200000;
      if (album2Budget >= transfer) {
        long album1Budget = 0;
        // Re-use the existing PreparedStatement for selecting the
        // marketing_budget to get the budget for Album 1.
        // Bind the query parameters to SingerId=1 and AlbumId=1.
        selectMarketingBudgetStatement.setLong(1, 1);
        selectMarketingBudgetStatement.setLong(2, 1);
        try (ResultSet resultSet =
            selectMarketingBudgetStatement.executeQuery()) {
          while (resultSet.next()) {
            album1Budget = resultSet.getLong("marketing_budget");
          }
        }

        // Transfer part of the marketing budget of Album 2 to Album 1.
        album1Budget += transfer;
        album2Budget -= transfer;
        String updateSql =
            "UPDATE albums "
                + "SET marketing_budget = ? "
                + "WHERE singer_id = ? and album_id = ?";
        try (PreparedStatement updateStatement =
            connection.prepareStatement(updateSql)) {
          // Update Album 1.
          int paramIndex = 0;
          updateStatement.setLong(++paramIndex, album1Budget);
          updateStatement.setLong(++paramIndex, 1);
          updateStatement.setLong(++paramIndex, 1);
          // Create a DML batch by calling addBatch
          // on the current PreparedStatement.
          updateStatement.addBatch();

          // Update Album 2 in the same DML batch.
          paramIndex = 0;
          updateStatement.setLong(++paramIndex, album2Budget);
          updateStatement.setLong(++paramIndex, 2);
          updateStatement.setLong(++paramIndex, 2);
          updateStatement.addBatch();

          // Execute both DML statements in one batch.
          updateStatement.executeBatch();
        }
      }
    }
    // Commit the current transaction.
    connection.commit();
    System.out.println(
        "Transferred marketing budget from Album 2 to Album 1");
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
writewithtransactionusingdml test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
writewithtransactionusingdmlpg test-instance example-db
```

### Transaction tags and request tags

Use [transaction tags and request tags](/spanner/docs/introspection/troubleshooting-with-tags) to troubleshoot transactions and queries in Spanner. You can set transaction tags and request tags in the JDBC with the `  TRANSACTION_TAG  ` and `  STATEMENT_TAG  ` session variables.

### GoogleSQL

``` java
static void tags(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Set AutoCommit=false to enable transactions.
    connection.setAutoCommit(false);
    // Set the TRANSACTION_TAG session variable to set a transaction tag
    // for the current transaction.
    connection
        .createStatement()
        .execute("SET TRANSACTION_TAG='example-tx-tag'");

    // Set the STATEMENT_TAG session variable to set the request tag
    // that should be included with the next SQL statement.
    connection
        .createStatement()
        .execute("SET STATEMENT_TAG='query-marketing-budget'");
    long marketingBudget = 0L;
    long singerId = 1L;
    long albumId = 1L;
    try (PreparedStatement statement = connection.prepareStatement(
        "SELECT MarketingBudget "
        + "FROM Albums "
        + "WHERE SingerId=? AND AlbumId=?")) {
      statement.setLong(1, singerId);
      statement.setLong(2, albumId);
      try (ResultSet albumResultSet = statement.executeQuery()) {
        while (albumResultSet.next()) {
          marketingBudget = albumResultSet.getLong(1);
        }
      }
    }
    // Reduce the marketing budget by 10% if it is more than 1,000.
    final long maxMarketingBudget = 1000L;
    final float reduction = 0.1f;
    if (marketingBudget > maxMarketingBudget) {
      marketingBudget -= (long) (marketingBudget * reduction);
      connection
          .createStatement()
          .execute("SET STATEMENT_TAG='reduce-marketing-budget'");
      try (PreparedStatement statement = connection.prepareStatement(
          "UPDATE Albums SET MarketingBudget=? "
              + "WHERE SingerId=? AND AlbumId=?")) {
        int paramIndex = 0;
        statement.setLong(++paramIndex, marketingBudget);
        statement.setLong(++paramIndex, singerId);
        statement.setLong(++paramIndex, albumId);
        statement.executeUpdate();
      }
    }

    // Commit the current transaction.
    connection.commit();
    System.out.println("Reduced marketing budget");
  }
}
```

### PostgreSQL

``` java
static void tagsPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Set AutoCommit=false to enable transactions.
    connection.setAutoCommit(false);
    // Set the TRANSACTION_TAG session variable to set a transaction tag
    // for the current transaction.
    connection
        .createStatement()
        .execute("set spanner.transaction_tag='example-tx-tag'");

    // Set the STATEMENT_TAG session variable to set the request tag
    // that should be included with the next SQL statement.
    connection
        .createStatement()
        .execute("set spanner.statement_tag='query-marketing-budget'");
    long marketingBudget = 0L;
    long singerId = 1L;
    long albumId = 1L;
    try (PreparedStatement statement = connection.prepareStatement(
        "select marketing_budget "
            + "from albums "
            + "where singer_id=? and album_id=?")) {
      statement.setLong(1, singerId);
      statement.setLong(2, albumId);
      try (ResultSet albumResultSet = statement.executeQuery()) {
        while (albumResultSet.next()) {
          marketingBudget = albumResultSet.getLong(1);
        }
      }
    }
    // Reduce the marketing budget by 10% if it is more than 1,000.
    final long maxMarketingBudget = 1000L;
    final float reduction = 0.1f;
    if (marketingBudget > maxMarketingBudget) {
      marketingBudget -= (long) (marketingBudget * reduction);
      connection
          .createStatement()
          .execute("set spanner.statement_tag='reduce-marketing-budget'");
      try (PreparedStatement statement = connection.prepareStatement(
          "update albums set marketing_budget=? "
              + "where singer_id=? AND album_id=?")) {
        int paramIndex = 0;
        statement.setLong(++paramIndex, marketingBudget);
        statement.setLong(++paramIndex, singerId);
        statement.setLong(++paramIndex, albumId);
        statement.executeUpdate();
      }
    }

    // Commit the current transaction.
    connection.commit();
    System.out.println("Reduced marketing budget");
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
tags test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
tagspg test-instance example-db
```

**Tip:** For a full list of commands that can be used to access Spanner features in JDBC, see [Session management commands](/spanner/docs/jdbc-session-mgmt-commands) .

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data. Set `  ReadOnly=true  ` and `  AutoCommit=false  ` on a `  java.sql.Connection  ` , or use the `  SET TRANSACTION READ ONLY  ` SQL statement, to execute a read-only transaction.

**Tip:** The JDBC driver supports multiple additional SQL statements for executing specific types of transactions and batches, and for accessing specific Spanner features. For a full list of supported statements, see [JDBC session management commands (GoogleSQL)](/spanner/docs/jdbc-session-mgmt-commands) or [JDBC session management commands (PostgreSQL)](/spanner/docs/jdbc-session-mgmt-commands-pgcompat) .

The following shows how to run a query and perform a read in the same read-only transaction:

### GoogleSQL

``` java
static void readOnlyTransaction(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Set AutoCommit=false to enable transactions.
    connection.setAutoCommit(false);
    // This SQL statement instructs the JDBC driver to use
    // a read-only transaction.
    connection.createStatement().execute("SET TRANSACTION READ ONLY");

    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "SELECT SingerId, AlbumId, AlbumTitle "
                    + "FROM Albums "
                    + "ORDER BY SingerId, AlbumId")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            resultSet.getLong("SingerId"),
            resultSet.getLong("AlbumId"),
            resultSet.getString("AlbumTitle"));
      }
    }
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "SELECT SingerId, AlbumId, AlbumTitle "
                    + "FROM Albums "
                    + "ORDER BY AlbumTitle")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            resultSet.getLong("SingerId"),
            resultSet.getLong("AlbumId"),
            resultSet.getString("AlbumTitle"));
      }
    }
    // End the read-only transaction by calling commit().
    connection.commit();
  }
}
```

### PostgreSQL

``` java
static void readOnlyTransactionPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Set AutoCommit=false to enable transactions.
    connection.setAutoCommit(false);
    // This SQL statement instructs the JDBC driver to use
    // a read-only transaction.
    connection.createStatement().execute("set transaction read only");

    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "SELECT singer_id, album_id, album_title "
                    + "FROM albums "
                    + "ORDER BY singer_id, album_id")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            resultSet.getLong("singer_id"),
            resultSet.getLong("album_id"),
            resultSet.getString("album_title"));
      }
    }
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "SELECT singer_id, album_id, album_title "
                    + "FROM albums "
                    + "ORDER BY album_title")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %d %s\n",
            resultSet.getLong("singer_id"),
            resultSet.getLong("album_id"),
            resultSet.getString("album_title"));
      }
    }
    // End the read-only transaction by calling commit().
    connection.commit();
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
readonlytransaction test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
readonlytransactionpg test-instance example-db
```

You should see output similar to:

``` text
    1 1 Total Junk
    1 2 Go, Go, Go
    2 1 Green
    2 2 Forever Hold Your Peace
    2 3 Terrified
    2 2 Forever Hold Your Peace
    1 2 Go, Go, Go
    2 1 Green
    2 3 Terrified
    1 1 Total Junk
```

### Partitioned queries and Data Boost

The [`  partitionQuery  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery) API divides a query into smaller pieces, or partitions, and uses multiple machines to fetch the partitions in parallel. Each partition is identified by a partition token. The PartitionQuery API has higher latency than the standard query API, because it is only intended for bulk operations such as exporting or scanning the whole database.

[Data Boost](/spanner/docs/databoost/databoost-overview) lets you execute analytics queries and data exports with near-zero impact to existing workloads on the provisioned Spanner instance. Data Boost only supports [partitioned queries](/spanner/docs/reads#read_data_in_parallel) .

### GoogleSQL

``` java
static void dataBoost(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // This enables Data Boost for all partitioned queries on this connection.
    connection.createStatement().execute("SET DATA_BOOST_ENABLED=TRUE");

    // Run a partitioned query. This query will use Data Boost.
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "RUN PARTITIONED QUERY "
                    + "SELECT SingerId, FirstName, LastName "
                    + "FROM Singers")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %s %s\n",
            resultSet.getLong("SingerId"),
            resultSet.getString("FirstName"),
            resultSet.getString("LastName"));
      }
    }
  }
}
```

### PostgreSQL

``` java
static void dataBoostPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // This enables Data Boost for all partitioned queries on this connection.
    connection
        .createStatement()
        .execute("set spanner.data_boost_enabled=true");

    // Run a partitioned query. This query will use Data Boost.
    try (ResultSet resultSet =
        connection
            .createStatement()
            .executeQuery(
                "run partitioned query "
                    + "select singer_id, first_name, last_name "
                    + "from singers")) {
      while (resultSet.next()) {
        System.out.printf(
            "%d %s %s\n",
            resultSet.getLong("singer_id"),
            resultSet.getString("first_name"),
            resultSet.getString("last_name"));
      }
    }
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
databoost test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
databoostpg test-instance example-db
```

For more information on running partitioned queries and using Data Boost with the JDBC driver, see:

  - [GoogleSQL: Data Boost and partitioned query statements](/spanner/docs/jdbc-session-mgmt-commands#data_boost_and_partitioned_query_statements)
  - [PostgreSQL: Data Boost and partitioned query statements](/spanner/docs/jdbc-session-mgmt-commands-pgcompat#data_boost_and_partitioned_query_statements)

## Partitioned DML

[Partitioned Data Manipulation Language (DML)](/spanner/docs/dml-partitioned) is designed for the following types of bulk updates and deletes:

  - Periodic cleanup and garbage collection.
  - Backfilling new columns with default values.

\* { GoogleSQL }

``` java
static void partitionedDml(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Enable Partitioned DML on this connection.
    connection
        .createStatement()
        .execute("SET AUTOCOMMIT_DML_MODE='PARTITIONED_NON_ATOMIC'");
    // Back-fill a default value for the MarketingBudget column.
    long lowerBoundUpdateCount =
        connection
            .createStatement()
            .executeUpdate("UPDATE Albums "
                + "SET MarketingBudget=0 "
                + "WHERE MarketingBudget IS NULL");
    System.out.printf("Updated at least %d albums\n", lowerBoundUpdateCount);
  }
}
```

### PostgreSQL

``` java
static void partitionedDmlPostgreSQL(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Enable Partitioned DML on this connection.
    connection
        .createStatement()
        .execute("set spanner.autocommit_dml_mode='partitioned_non_atomic'");
    // Back-fill a default value for the MarketingBudget column.
    long lowerBoundUpdateCount =
        connection
            .createStatement()
            .executeUpdate("update albums "
                + "set marketing_budget=0 "
                + "where marketing_budget is null");
    System.out.printf("Updated at least %d albums\n", lowerBoundUpdateCount);
  }
}
```

Run the sample with this command:

### GoogleSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
pdml test-instance example-db
```

### PostgreSQL

``` text
java -jar target/jdbc-snippets/jdbc-samples.jar \
pdmlpg test-instance example-db
```

For more information on `  AUTOCOMMIT_DML_MODE  ` , see:

  - [GoogleSQL AUTOCOMMIT\_DML\_MODE](/spanner/docs/jdbc-session-mgmt-commands#autocommit_dml_mode)
  - [PostgreSQL SPANNER.AUTOCOMMIT\_DML\_MODE](/spanner/docs/jdbc-session-mgmt-commands-pgcompat#spannerautocommit_dml_mode)

## Cleanup

To avoid incurring additional charges to your Cloud Billing account for the resources used in this tutorial, drop the database and delete the instance that you created.

### Delete the database

If you delete an instance, all databases within it are automatically deleted. This step shows how to delete a database without deleting an instance (you would still incur charges for the instance).

#### On the command line

``` text
gcloud spanner databases delete example-db --instance=test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click the instance.

3.  Click the database that you want to delete.

4.  In the **Database details** page, click **Delete** .

5.  Confirm that you want to delete the database and click **Delete** .

### Delete the instance

Deleting an instance automatically drops all databases created in that instance.

#### On the command line

``` text
gcloud spanner instances delete test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click your instance.

3.  Click **Delete** .

4.  Confirm that you want to delete the instance and click **Delete** .

## What's next

  - Learn how to [Integrate Spanner with Spring Data JPA (GoogleSQL dialect)](/spanner/docs/use-spring-data-jpa) .
  - Learn how to [Integrate Spanner with Spring Data JPA (PostgreSQL dialect)](/spanner/docs/use-spring-data-jpa-postgresql) .
  - Learn how to [Integrate Spanner with Hibernate ORM (GoogleSQL dialect)](/spanner/docs/use-hibernate) .
  - Learn how to [Integrate Spanner with Hibernate ORM (PostgreSQL dialect)](/spanner/docs/use-hibernate-postgresql) .
  - Learn more about [JDBC session management commands (GoogleSQL)](/spanner/docs/jdbc-session-mgmt-commands) .
  - Learn more about [JDBC session management commands (PostgreSQL)](/spanner/docs/jdbc-session-mgmt-commands-pgcompat) .

<!-- end list -->

  - Learn how to [access Spanner with a virtual machine instance](/spanner/docs/configure-virtual-machine-instance) .

  - Learn about authorization and authentication credentials in [Authenticate to Cloud services using client libraries](/docs/authentication/getting-started) .

  - Learn more about Spanner [Schema design best practices](/spanner/docs/schema-design) .
