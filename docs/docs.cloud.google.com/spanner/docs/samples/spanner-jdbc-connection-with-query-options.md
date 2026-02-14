Set connection query options by using JDBC.

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

class ConnectionWithQueryOptionsExample {

  static void connectionWithQueryOptions() throws SQLException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    connectionWithQueryOptions(projectId, instanceId, databaseId);
  }

  static void connectionWithQueryOptions(String projectId, String instanceId, String databaseId)
      throws SQLException {
    String optimizerVersion = "1";
    String connectionUrl =
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s?optimizerVersion=%s",
            projectId, instanceId, databaseId, optimizerVersion);
    try (Connection connection = DriverManager.getConnection(connectionUrl);
        Statement statement = connection.createStatement()) {
      // Execute a query using the optimizer version '1'.
      try (ResultSet rs =
          statement.executeQuery(
              "SELECT SingerId, FirstName, LastName FROM Singers ORDER BY LastName")) {
        while (rs.next()) {
          System.out.printf("%d %s %s%n", rs.getLong(1), rs.getString(2), rs.getString(3));
        }
      }
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
