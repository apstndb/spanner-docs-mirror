Run a query by using JDBC.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Connect JDBC to a GoogleSQL-dialect database](/spanner/docs/use-oss-jdbc)

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

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
