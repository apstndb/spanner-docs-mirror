Insert data by using JDBC.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Connect JDBC to a GoogleSQL-dialect database](/spanner/docs/use-oss-jdbc)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
