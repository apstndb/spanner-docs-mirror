**Note:** This page describes the [Spanner JDBC driver](https://github.com/googleapis/java-spanner-jdbc) , which is recommended and authored by Google.

This page shows you how to perform basic operations in Spanner using the [Spanner JDBC driver](https://github.com/googleapis/java-spanner-jdbc) .

## Install the JDBC driver

Follow the steps in [Spanner client libraries](/spanner/docs/reference/libraries) to set up authentication, and then add the Spanner JDBC driver dependencies, shown in the following snippet, to your `  pom.xml  ` file.

``` xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>libraries-bom</artifactId>
      <version>26.74.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-spanner-jdbc</artifactId>
    <exclusions>
      <exclusion>
        <groupId>com.google.api.grpc</groupId>
        <artifactId>proto-google-cloud-spanner-executor-v1</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
```

In case you are using a framework that requires the Java class name in order to load the JDBC driver, it is `  com.google.cloud.spanner.jdbc.JdbcDriver  ` . See the [API documentation for JdbcDriver](/java/docs/reference/google-cloud-spanner-jdbc/latest/com.google.cloud.spanner.jdbc.JdbcDriver) for how to set up a connection.

## Connect to a Spanner database

The [JdbcDriver](/java/docs/reference/google-cloud-spanner-jdbc/latest/com.google.cloud.spanner.jdbc.JdbcDriver) class description shows the connection string syntax and includes sample code to create a connection and run a query.

The driver automatically detects the SQL dialect (GoogleSQL or PostgreSQL) of the specified database. A dialect parameter is not required or allowed.

## Connect to the emulator

To connect to the [emulator](/spanner/docs/emulator) , set the `  SPANNER_EMULATOR_HOST  ` environment variable, for example:

### Linux / macOS

``` text
export SPANNER_EMULATOR_HOST=localhost:9010
```

### Windows

``` text
set SPANNER_EMULATOR_HOST=localhost:9010
```

This instructs the Spanner JDBC driver driver to connect to the emulator running on the `  localhost  ` instead of the default production service.

**Note:** Although less common, you can specify the host in the connection string, for example `  jdbc:cloudspanner://localhost:9010/projects/test-project/instances/test-instance/databases/test-db;usePlainText=true  ` . The host in the connection string always takes precedence regardless of whether `  SPANNER_EMULATOR_HOST  ` is set. Also, you must specify `  usePlainText=true  ` in the connection string because the emulator does not support SSL/TLS.

## Examples

The following code examples cover some common use cases.

### Run a schema update

The following code example adds the `  Singers  ` table to the database by first creating a JDBC connection and then creating the table:

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

class CreateTableExample {

  static void createTable() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    createTable(projectId, instanceId, databaseId);
  }

  static void createTable(String projectId, String instanceId, String databaseId)
      throws SQLException {
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
            projectId, instanceId, databaseId);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (Statement statement = connection.createStatement()) {
        statement.execute(
            "CREATE TABLE Singers (\n"
                + "  SingerId   INT64 NOT NULL,\n"
                + "  FirstName  STRING(1024),\n"
                + "  LastName   STRING(1024),\n"
                + "  SingerInfo BYTES(MAX),\n"
                + "  Revenues   NUMERIC,\n"
                + ") PRIMARY KEY (SingerId)\n");
      }
    }
    System.out.println("Created table [Singers]");
  }
}
```

### Use a transaction in autocommit mode to add rows

If you don't need to commit multiple operations as a group, you can use a transaction in autocommit mode, which is the default behavior. The following code example uses a transaction in autocommit mode to add rows to the `  Singers  ` table:

``` java
import java.math.BigDecimal;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Types;
import java.util.Arrays;
import java.util.List;

class InsertDataExample {
  // Class to contain singer sample data.
  static class Singer {
    final long singerId;
    final String firstName;
    final String lastName;
    final BigDecimal revenues;

    Singer(long singerId, String firstName, String lastName, BigDecimal revenues) {
      this.singerId = singerId;
      this.firstName = firstName;
      this.lastName = lastName;
      this.revenues = revenues;
    }
  }

  static final List<Singer> SINGERS =
      Arrays.asList(
          new Singer(10, "Marc", "Richards", new BigDecimal("104100.00")),
          new Singer(20, "Catalina", "Smith", new BigDecimal("9880.99")),
          new Singer(30, "Alice", "Trentor", new BigDecimal("300183")),
          new Singer(40, "Lea", "Martin", new BigDecimal("20118.12")),
          new Singer(50, "David", "Lomond", new BigDecimal("311399.26")));

  static void insertData() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    insertData(projectId, instanceId, databaseId);
  }

  static void insertData(String projectId, String instanceId, String databaseId)
      throws SQLException {
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
            projectId, instanceId, databaseId);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (PreparedStatement ps =
          connection.prepareStatement(
              "INSERT INTO Singers\n"
                  + "(SingerId, FirstName, LastName, SingerInfo, Revenues)\n"
                  + "VALUES\n"
                  + "(?, ?, ?, ?, ?)")) {
        for (Singer singer : SINGERS) {
          ps.setLong(1, singer.singerId);
          ps.setString(2, singer.firstName);
          ps.setString(3, singer.lastName);
          ps.setNull(4, Types.BINARY);
          ps.setBigDecimal(5, singer.revenues);
          ps.addBatch();
        }
        int[] updateCounts = ps.executeBatch();
        System.out.printf("Insert counts: %s%n", Arrays.toString(updateCounts));
      }
    }
  }
}
```

### Control how multiple operations are committed as a group

If you want to control whether Spanner commits multiple operations together as a group, you can disable autocommit mode. The following code example uses `  connection.setAutoCommit(false)  ` and `  connection.commit()  ` to add rows to the `  Singers  ` table.

``` java
import com.google.common.collect.ImmutableList;
import java.math.BigDecimal;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Arrays;

class BatchDmlExample {
  static class Singer {
    final long singerId;
    final String firstName;
    final String lastName;
    final BigDecimal revenues;

    Singer(long singerId, String firstName, String lastName, BigDecimal revenues) {
      this.singerId = singerId;
      this.firstName = firstName;
      this.lastName = lastName;
      this.revenues = revenues;
    }
  }

  static void batchDml() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    batchDml(projectId, instanceId, databaseId);
  }

  // This example shows how to execute a batch of DML statements with the JDBC driver.
  static void batchDml(String projectId, String instanceId, String databaseId) throws SQLException {
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
            projectId, instanceId, databaseId);

    ImmutableList<Singer> singers = ImmutableList.of(
        new Singer(10, "Marc", "Richards", BigDecimal.valueOf(10000)),
        new Singer(11, "Amirah", "Finney", BigDecimal.valueOf(195944.10d)),
        new Singer(12, "Reece", "Dunn", BigDecimal.valueOf(10449.90))
    );

    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      connection.setAutoCommit(false);
      // Use prepared statements for the lowest possible latency when executing the same SQL string
      // multiple times.
      try (PreparedStatement statement = connection.prepareStatement(
          "INSERT INTO Singers (SingerId, FirstName, LastName, Revenues)\n" 
              + "VALUES (?, ?, ?, ?)")) {
        for (Singer singer : singers) {
          statement.setLong(1, singer.singerId);
          statement.setString(2, singer.firstName);
          statement.setString(3, singer.lastName);
          statement.setBigDecimal(4, singer.revenues);
          // Add the current parameter values to the batch.
          statement.addBatch();
        }
        // Execute the batched statements.
        int[] updateCounts = statement.executeBatch();
        connection.commit();
        System.out.printf("Batch insert counts: %s%n", Arrays.toString(updateCounts));
      }
    }
  }
}
```

### Run a SQL query

The following code example returns all rows in the `  Singers  ` table ordered by the singer's last name:

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class SingleUseReadOnlyExample {

  static void singleUseReadOnly() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    singleUseReadOnly(projectId, instanceId, databaseId);
  }

  static void singleUseReadOnly(String projectId, String instanceId, String databaseId)
      throws SQLException {
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
            projectId, instanceId, databaseId);
    try (Connection connection = DriverManager.getConnection(connectionUrl);
        Statement statement = connection.createStatement()) {
      // When the connection is in autocommit mode, any query that is executed will automatically
      // be executed using a single-use read-only transaction, even if the connection itself is in
      // read/write mode.
      try (ResultSet rs =
          statement.executeQuery(
              "SELECT SingerId, FirstName, LastName, Revenues FROM Singers ORDER BY LastName")) {
        while (rs.next()) {
          System.out.printf(
              "%d %s %s %s%n",
              rs.getLong(1), rs.getString(2), rs.getString(3), rs.getBigDecimal(4));
        }
      }
    }
  }
}
```

## What's next

  - Get answers to [frequently asked questions](/spanner/docs/open-source-jdbc) about the Spanner JDBC driver.
  - Learn about the Spanner JDBC driver [session management commands](/spanner/docs/jdbc-session-mgmt-commands) .
