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

    using Google.Cloud.Spanner.Data;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    
    public class QueryNewColumnAsyncSample
    {
        public class Album
        {
            public int SingerId { get; set; }
            public int AlbumId { get; set; }
            public long MarketingBudget { get; set; }
        }
    
        public async Task<List<Album>> QueryNewColumnAsync(string projectId, string instanceId, string databaseId)
        {
            string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
    
            var albums = new List<Album>();
            using var connection = new SpannerConnection(connectionString);
            using var cmd = connection.CreateSelectCommand("SELECT * FROM Albums");
            using var reader = await cmd.ExecuteReaderAsync();
            while (await reader.ReadAsync())
            {
                albums.Add(new Album
                {
                    SingerId = reader.GetFieldValue<int>("SingerId"),
                    AlbumId = reader.GetFieldValue<int>("AlbumId"),
                    MarketingBudget = reader.IsDBNull(reader.GetOrdinal("MarketingBudget")) ? 0 : reader.GetFieldValue<long>("MarketingBudget")
                });
            }
            return albums;
        }
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import (
     "context"
     "database/sql"
     "fmt"
     "io"
    
     _ "github.com/googleapis/go-sql-spanner"
    )
    
    func QueryNewColumn(ctx context.Context, w io.Writer, databaseName string) error {
     db, err := sql.Open("spanner", databaseName)
     if err != nil {
         return err
     }
     defer db.Close()
    
     rows, err := db.QueryContext(ctx,
         `SELECT SingerId, AlbumId, MarketingBudget
         FROM Albums
         ORDER BY SingerId, AlbumId`)
     defer rows.Close()
     if err != nil {
         return err
     }
     for rows.Next() {
         var singerId, albumId int64
         var marketingBudget sql.NullInt64
         err = rows.Scan(&singerId, &albumId, &marketingBudget)
         if err != nil {
             return err
         }
         budget := "NULL"
         if marketingBudget.Valid {
             budget = fmt.Sprintf("%v", marketingBudget.Int64)
         }
         fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, budget)
     }
     if rows.Err() != nil {
         return rows.Err()
     }
     return rows.Close()
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    class QueryDataWithNewColumn {
      static void queryDataWithNewColumn(String host, int port, String database) throws SQLException {
        String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
        try (Connection connection = DriverManager.getConnection(connectionUrl)) {
          try (ResultSet resultSet =
              connection
                  .createStatement()
                  .executeQuery(
                      "SELECT singer_id, album_id, marketing_budget "
                          + "FROM albums "
                          + "ORDER BY singer_id, album_id")) {
            while (resultSet.next()) {
              System.out.printf(
                  "%d %d %s\n",
                  resultSet.getLong("singer_id"),
                  resultSet.getLong("album_id"),
                  resultSet.getString("marketing_budget"));
            }
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

    function query_data_with_new_column(string $host, string $port, string $database): void
    {
        $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
        $connection = new PDO($dsn);
    
        $statement = $connection->query(
            "SELECT singer_id, album_id, marketing_budget "
            ."FROM albums "
            ."ORDER BY singer_id, album_id"
        );
        $rows = $statement->fetchAll();
        foreach ($rows as $album)
        {
            printf("%s\t%s\t%s\n", $album["singer_id"], $album["album_id"], $album["marketing_budget"]);
        }
    
        $rows = null;
        $statement = null;
        $connection = null;
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import string
    import psycopg
    
    
    def query_data_with_new_column(host: string, port: int, database: string):
        with psycopg.connect("host={host} port={port} dbname={database} "
                             "sslmode=disable".format(host=host,
                                                      port=port,
                                                      database=database)) as conn:
            conn.autocommit = True
            with conn.cursor() as cur:
                cur.execute("SELECT singer_id, album_id, marketing_budget "
                            "FROM albums "
                            "ORDER BY singer_id, album_id")
                for album in cur:
                    print(album)

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
