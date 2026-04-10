Create a table by using JDBC.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Connect JDBC to a GoogleSQL-dialect database](https://docs.cloud.google.com/spanner/docs/use-oss-jdbc)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=spanner) .
