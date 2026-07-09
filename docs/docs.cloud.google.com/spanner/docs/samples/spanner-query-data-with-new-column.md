---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-query-data-with-new-column
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-query-data-with-new-column
title: Query data from a new column
description: Query data from a new column.
data_source: docs.cloud.google.com
---

Query data from a new column.

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

    void QueryNewColumn(google::cloud::spanner::Client client) {
      namespace spanner = ::google::cloud::spanner;
    
      spanner::SqlStatement select(
          "SELECT SingerId, AlbumId, MarketingBudget FROM Albums");
      using RowType =
          std::tuple<std::int64_t, std::int64_t, std::optional<std::int64_t>>;
    
      auto rows = client.ExecuteQuery(std::move(select));
      for (auto& row : spanner::StreamOf<RowType>(rows)) {
        if (!row) throw std::move(row).status();
        std::cout << "SingerId: " << std::get<0>(*row) << "\t";
        std::cout << "AlbumId: " << std::get<1>(*row) << "\t";
        auto marketing_budget = std::get<2>(*row);
        if (marketing_budget) {
          std::cout << "MarketingBudget: " << *marketing_budget << "\n";
        } else {
          std::cout << "MarketingBudget: NULL\n";
        }
      }
      std::cout << "Read completed for [spanner_read_data_with_new_column]\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    public static async Task QueryNewColumn(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        await using var command = connection.CreateCommand();
        command.CommandText = "SELECT SingerId, AlbumId, MarketingBudget " +
                              "FROM Albums " +
                              "ORDER BY SingerId, AlbumId";
        await using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            Console.WriteLine($"{reader["SingerId"]} {reader["AlbumId"]} {reader["MarketingBudget"]}");
        }
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    func queryNewColumn(ctx context.Context, w io.Writer, client *spanner.Client) error {
     stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, MarketingBudget FROM Albums`}
     iter := client.Single().Query(ctx, stmt)
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             return nil
         }
         if err != nil {
             return err
         }
         var singerID, albumID int64
         var marketingBudget spanner.NullInt64
         if err := row.ColumnByName("SingerId", &singerID); err != nil {
             return err
         }
         if err := row.ColumnByName("AlbumId", &albumID); err != nil {
             return err
         }
         if err := row.ColumnByName("MarketingBudget", &marketingBudget); err != nil {
             return err
         }
         budget := "NULL"
         if marketingBudget.Valid {
             budget = strconv.FormatInt(marketingBudget.Int64, 10)
         }
         fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, budget)
     }
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    // This sample uses the `MarketingBudget` column. You can add the column
    // by running the `add_column` sample or by running this DDL statement against
    // your database:
    //    ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
    
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
    
    const query = {
      sql: 'SELECT SingerId, AlbumId, MarketingBudget FROM Albums',
    };
    
    // Queries rows from the Albums table
    try {
      const [rows] = await database.run(query);
    
      rows.forEach(row => {
        const json = row.toJSON();
    
        console.log(
          `SingerId: ${json.SingerId}, AlbumId: ${
            json.AlbumId
          }, MarketingBudget: ${
            json.MarketingBudget ? json.MarketingBudget : null
          }`
        );
      });
    } catch (err) {
      console.error('Failed to query data with new column:', err.message || err);
    } finally {
      // Close the database when finished.
      await database.close();
    }

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import { Client } from 'pg';
    
    async function queryDataWithNewColumn(host: string, port: number, database: string): Promise<void> {
      const connection = new Client({
        host: host,
        port: port,
        database: database,
      });
      await connection.connect();
    
      const result = await connection.query(
          "SELECT singer_id, album_id, marketing_budget "
          + "FROM albums "
          + "ORDER BY singer_id, album_id"
      );
      for (const row of result.rows) {
        console.log(`${row["singer_id"]} ${row["album_id"]} ${row["marketing_budget"]}`);
      }
    
      // Close the connection.
      await connection.end();
    }

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    use Google\Cloud\Spanner\SpannerClient;
    
    /**
     * Queries sample data from the database using SQL.
     * This sample uses the `MarketingBudget` column. You can add the column
     * by running the `add_column` sample or by running this DDL statement against
     * your database:
     *
     *      ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
     *
     * Example:
     * ```
     * query_data_with_new_column($instanceId, $databaseId);
     * ```
     *
     * @param string $instanceId The Spanner instance ID.
     * @param string $databaseId The Spanner database ID.
     */
    function query_data_with_new_column(string $instanceId, string $databaseId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
        $database = $instance->database($databaseId);
    
        $results = $database->execute(
            'SELECT SingerId, AlbumId, MarketingBudget FROM Albums'
        );
    
        foreach ($results as $row) {
            printf('SingerId: %s, AlbumId: %s, MarketingBudget: %d' . PHP_EOL,
                $row['SingerId'], $row['AlbumId'], $row['MarketingBudget']);
        }
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    def query_data_with_new_column(instance_id, database_id):
        """Queries sample data from the database using SQL.
    
        This sample uses the `MarketingBudget` column. You can add the column
        by running the `add_column` sample or by running this DDL statement against
        your database:
    
            ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
        """
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
        database = instance.database(database_id)
    
        with database.snapshot() as snapshot:
            results = snapshot.execute_sql(
                "SELECT SingerId, AlbumId, MarketingBudget FROM Albums"
            )
    
            for row in results:
                print("SingerId: {}, AlbumId: {}, MarketingBudget: {}".format(*row))

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    # project_id  = "Your Google Cloud project ID"
    # instance_id = "Your Spanner instance ID"
    # database_id = "Your Spanner database ID"
    
    require "google/cloud/spanner"
    
    spanner = Google::Cloud::Spanner.new project: project_id
    client  = spanner.client instance_id, database_id
    
    client.execute("SELECT SingerId, AlbumId, MarketingBudget FROM Albums").rows.each do |row|
      puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:MarketingBudget]}"
    end

### Rust

    use google_cloud_spanner::client::DatabaseClient;
    use google_cloud_spanner::statement::Statement;
    
    pub async fn sample(client: &DatabaseClient) -> anyhow::Result<()> {
        let statement =
            Statement::builder("SELECT SingerId, AlbumId, MarketingBudget FROM Albums").build();
        let transaction = client.single_use().build();
        let mut result_set = transaction.execute_query(statement).await?;
    
        while let Some(row) = result_set.next().await.transpose()? {
            let singer_id: i64 = row.get("SingerId");
            let album_id: i64 = row.get("AlbumId");
            let marketing_budget: Option<i64> = row.get("MarketingBudget");
    
            match marketing_budget {
                Some(budget) => println!("{singer_id} {album_id} {budget}"),
                None => println!("{singer_id} {album_id} NULL"),
            }
        }
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
