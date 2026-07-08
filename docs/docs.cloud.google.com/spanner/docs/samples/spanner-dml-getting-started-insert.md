---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-dml-getting-started-insert
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-dml-getting-started-insert
title: DML write data
description: Write data by using DML.
data_source: docs.cloud.google.com
---

Write data by using DML.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Getting started with Spanner and PGAdapter](https://docs.cloud.google.com/spanner/docs/getting-started/pgadapter)
  - [Getting started with Spanner in ADO.NET](https://docs.cloud.google.com/spanner/docs/getting-started/ado_net)
  - [Getting started with Spanner in C\#](https://docs.cloud.google.com/spanner/docs/getting-started/csharp)
  - [Getting started with Spanner in C++](https://docs.cloud.google.com/spanner/docs/getting-started/cpp)
  - [Getting started with Spanner in Go](https://docs.cloud.google.com/spanner/docs/getting-started/go)
  - [Getting started with Spanner in Go database/sql](https://docs.cloud.google.com/spanner/docs/getting-started/database_sql)
  - [Getting started with Spanner in Java](https://docs.cloud.google.com/spanner/docs/getting-started/java)
  - [Getting started with Spanner in JDBC](https://docs.cloud.google.com/spanner/docs/getting-started/jdbc)
  - [Getting started with Spanner in Node.js](https://docs.cloud.google.com/spanner/docs/getting-started/nodejs)
  - [Getting started with Spanner in PHP](https://docs.cloud.google.com/spanner/docs/getting-started/php)
  - [Getting started with Spanner in Python](https://docs.cloud.google.com/spanner/docs/getting-started/python)
  - [Getting started with Spanner in Ruby](https://docs.cloud.google.com/spanner/docs/getting-started/ruby)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void DmlGettingStartedInsert(google::cloud::spanner::Client client) {
      using ::google::cloud::StatusOr;
      namespace spanner = ::google::cloud::spanner;
    
      auto commit_result = client.Commit(
          [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
            auto insert = client.ExecuteDml(
                std::move(txn),
                spanner::SqlStatement(
                    "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES"
                    " (12, 'Melissa', 'Garcia'),"
                    " (13, 'Russell', 'Morales'),"
                    " (14, 'Jacqueline', 'Long'),"
                    " (15, 'Dylan', 'Shaw')"));
            if (!insert) return std::move(insert).status();
            return spanner::Mutations{};
          });
      if (!commit_result) throw std::move(commit_result).status();
      std::cout << "Insert was successful [spanner_dml_getting_started_insert]\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    public static async Task WriteDataWithDml(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Add 4 rows in one statement.
        // The ADO.NET driver supports positional query parameters.
        await using var command = connection.CreateCommand();
        command.CommandText = "INSERT INTO Singers (SingerId, FirstName, LastName) " +
                              "VALUES (?, ?, ?), (?, ?, ?), " +
                              "       (?, ?, ?), (?, ?, ?)";
        command.Parameters.Add(12);
        command.Parameters.Add("Melissa");
        command.Parameters.Add("Garcia");
    
        command.Parameters.Add(13);
        command.Parameters.Add("Russel");
        command.Parameters.Add("Morales");
    
        command.Parameters.Add(14);
        command.Parameters.Add("Jacqueline");
        command.Parameters.Add("Long");
    
        command.Parameters.Add(15);
        command.Parameters.Add("Dylan");
        command.Parameters.Add("Shaw");
    
        var affected = await command.ExecuteNonQueryAsync();
        Console.WriteLine($"{affected} record(s) inserted.");
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import (
     "context"
     "fmt"
    
     "github.com/jackc/pgx/v5"
    )
    
    func WriteDataWithDml(host string, port int, database string) error {
     ctx := context.Background()
     connString := fmt.Sprintf(
         "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
         host, port, database)
     conn, err := pgx.Connect(ctx, connString)
     if err != nil {
         return err
     }
     defer conn.Close(ctx)
    
     tag, err := conn.Exec(ctx,
         "INSERT INTO singers (singer_id, first_name, last_name) "+
             "VALUES ($1, $2, $3), ($4, $5, $6), "+
             "       ($7, $8, $9), ($10, $11, $12)",
         12, "Melissa", "Garcia",
         13, "Russel", "Morales",
         14, "Jacqueline", "Long",
         15, "Dylan", "Shaw")
     if err != nil {
         return err
     }
     fmt.Printf("%v records inserted\n", tag.RowsAffected())
    
     return nil
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    import java.util.Arrays;
    import java.util.List;
    
    class WriteDataWithDml {
      static class Singer {
        private final long singerId;
        private final String firstName;
        private final String lastName;
    
        Singer(final long id, final String first, final String last) {
          this.singerId = id;
          this.firstName = first;
          this.lastName = last;
        }
      }
    
      static void writeDataWithDml(String host, int port, String database) throws SQLException {
        String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
        try (Connection connection = DriverManager.getConnection(connectionUrl)) {
          // Add 4 rows in one statement.
          // JDBC always uses '?' as a parameter placeholder.
          try (PreparedStatement preparedStatement =
              connection.prepareStatement(
                  "INSERT INTO singers (singer_id, first_name, last_name) VALUES "
                      + "(?, ?, ?), "
                      + "(?, ?, ?), "
                      + "(?, ?, ?), "
                      + "(?, ?, ?)")) {
    
            final List<Singer> singers =
                Arrays.asList(
                    new Singer(/* SingerId= */ 12L, "Melissa", "Garcia"),
                    new Singer(/* SingerId= */ 13L, "Russel", "Morales"),
                    new Singer(/* SingerId= */ 14L, "Jacqueline", "Long"),
                    new Singer(/* SingerId= */ 15L, "Dylan", "Shaw"));
    
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
    }

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    // Imports the Google Cloud client library
    const {Spanner} = require('@google-cloud/spanner');
    
    /**
     * TODO(developer): Uncomment the following lines before running the sample.
     */
    // const projectId = 'my-project-id';
    // const instanceId = 'my-instance';
    // const databaseId = 'my-database';
    
    // Creates a client
    const spanner = new Spanner({
      projectId: projectId,
    });
    
    // Gets a reference to a Cloud Spanner instance and database
    const instance = spanner.instance(instanceId);
    const database = instance.database(databaseId);
    
    try {
      await database.runTransactionAsync(async transaction => {
        const [rowCount] = await transaction.runUpdate({
          sql: `INSERT Singers (SingerId, FirstName, LastName) VALUES
          (12, 'Melissa', 'Garcia'),
          (13, 'Russell', 'Morales'),
          (14, 'Jacqueline', 'Long'),
          (15, 'Dylan', 'Shaw')`,
        });
        console.log(`${rowCount} records inserted.`);
        await transaction.commit();
      });
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the database when finished.
      await database.close();
    }

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import { Client } from 'pg';
    
    async function writeDataWithDml(host: string, port: number, database: string): Promise<void> {
      const connection = new Client({
        host: host,
        port: port,
        database: database,
      });
      await connection.connect();
    
      const result = await connection.query("INSERT INTO singers (singer_id, first_name, last_name) " +
          "VALUES ($1, $2, $3), ($4, $5, $6), " +
          "       ($7, $8, $9), ($10, $11, $12)",
           [12, "Melissa", "Garcia",
            13, "Russel", "Morales",
            14, "Jacqueline", "Long",
            15, "Dylan", "Shaw"])
      console.log(`${result.rowCount} records inserted`);
    
      // Close the connection.
      await connection.end();
    }

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    function write_data_with_dml(string $host, string $port, string $database): void
    {
        $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
        $connection = new PDO($dsn);
    
        $sql = "INSERT INTO singers (singer_id, first_name, last_name)"
                            ." VALUES (?, ?, ?), (?, ?, ?), "
                            ."        (?, ?, ?), (?, ?, ?)";
        $statement = $connection->prepare($sql);
        $statement->execute([
            12, "Melissa", "Garcia",
            13, "Russel", "Morales",
            14, "Jacqueline", "Long",
            15, "Dylan", "Shaw"
        ]);
        printf("%d records inserted\n", $statement->rowCount());
    
        $statement = null;
        $connection = null;
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    # instance_id = "your-spanner-instance"
    # database_id = "your-spanner-db-id"
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)
    
    def insert_singers(transaction):
        row_ct = transaction.execute_update(
            "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES "
            "(12, 'Melissa', 'Garcia'), "
            "(13, 'Russell', 'Morales'), "
            "(14, 'Jacqueline', 'Long'), "
            "(15, 'Dylan', 'Shaw')"
        )
        print("{} record(s) inserted.".format(row_ct))
    
    database.run_in_transaction(insert_singers)

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    # project_id  = "Your Google Cloud project ID"
    # instance_id = "Your Spanner instance ID"
    # database_id = "Your Spanner database ID"
    
    require "google/cloud/spanner"
    
    spanner = Google::Cloud::Spanner.new project: project_id
    client  = spanner.client instance_id, database_id
    row_count = 0
    
    client.transaction do |transaction|
      row_count = transaction.execute_update(
        "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES
         (12, 'Melissa', 'Garcia'),
         (13, 'Russell', 'Morales'),
         (14, 'Jacqueline', 'Long'),
         (15, 'Dylan', 'Shaw'),
         (16, 'Billie', 'Eillish'),
         (17, 'Judy', 'Garland'),
         (18, 'Taylor', 'Swift'),
         (19, 'Miley', 'Cyrus'),
         (20, 'Michael', 'Jackson'),
         (21, 'Ariana', 'Grande'),
         (22, 'Elvis', 'Presley'),
         (23, 'Kanye', 'West'),
         (24, 'Lady', 'Gaga'),
         (25, 'Nick', 'Jonas')"
      )
    end
    
    puts "#{row_count} records inserted."

### Rust

    use google_cloud_spanner::client::DatabaseClient;
    use google_cloud_spanner::statement::Statement;
    
    pub async fn sample(client: &DatabaseClient) -> anyhow::Result<()> {
        let runner = client.read_write_transaction().build().await?;
    
        runner
            .run(async |transaction| {
                let sql = r#"INSERT INTO Singers (SingerId, FirstName, LastName) VALUES
                           (12, 'Melissa', 'Garcia'),
                           (13, 'Russell', 'Morales'),
                           (14, 'Jacqueline', 'Long'),
                           (15, 'Dylan', 'Shaw')"#;
                let statement = Statement::builder(sql).build();
                let row_count = transaction.execute_update(statement).await?;
                println!("{row_count} records inserted.");
                Ok(())
            })
            .await?;
    
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
