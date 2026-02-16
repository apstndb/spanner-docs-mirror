Use the SET statement to specify query options by using JDBC.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage the query optimizer](/spanner/docs/query-optimizer/manage-query-optimizer)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

class SetQueryOptionsExample {

  static void setQueryOptions() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    setQueryOptions(projectId, instanceId, databaseId);
  }

  static void setQueryOptions(String projectId, String instanceId, String databaseId)
      throws SQLException {
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
            projectId, instanceId, databaseId);
    try (Connection connection = DriverManager.getConnection(connectionUrl);
        Statement statement = connection.createStatement()) {
      // Instruct the JDBC connection to use version '1' of the query optimizer.
      // NOTE: Use `SET SPANNER.OPTIMIZER_VERSION='1`` when connected to a PostgreSQL database.
      statement.execute("SET OPTIMIZER_VERSION='1'");
      // Execute a query using the latest optimizer version.
      try (ResultSet rs =
          statement.executeQuery(
              "SELECT SingerId, FirstName, LastName FROM Singers ORDER BY LastName")) {
        while (rs.next()) {
          System.out.printf("%d %s %s%n", rs.getLong(1), rs.getString(2), rs.getString(3));
        }
      }
      // NOTE: Use `SHOW SPANNER.OPTIMIZER_VERSION` when connected to a PostgreSQL database.
      try (ResultSet rs = statement.executeQuery("SHOW VARIABLE OPTIMIZER_VERSION")) {
        while (rs.next()) {
          System.out.printf("Optimizer version: %s%n", rs.getString(1));
        }
      }
    }
  }
}
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
