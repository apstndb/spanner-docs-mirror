---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-query-data
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-query-data
title: Query data
description: Query data.
data_source: docs.cloud.google.com
---

Query data.

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
  - [Reads outside of transactions](https://docs.cloud.google.com/spanner/docs/reads)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void QueryData(google::cloud::spanner::Client client) {
      namespace spanner = ::google::cloud::spanner;
    
      spanner::SqlStatement select("SELECT SingerId, LastName FROM Singers");
      using RowType = std::tuple<std::int64_t, std::string>;
      auto rows = client.ExecuteQuery(std::move(select));
      for (auto& row : spanner::StreamOf<RowType>(rows)) {
        if (!row) throw std::move(row).status();
        std::cout << "SingerId: " << std::get<0>(*row) << "\t";
        std::cout << "LastName: " << std::get<1>(*row) << "\n";
      }
    
      std::cout << "Query completed for [spanner_query_data]\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    using Google.Cloud.Spanner.Data;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    
    public class QuerySampleDataAsyncSample
    {
        public class Album
        {
            public int SingerId { get; set; }
            public int AlbumId { get; set; }
            public string AlbumTitle { get; set; }
        }
    
        public async Task<List<Album>> QuerySampleDataAsync(string projectId, string instanceId, string databaseId)
        {
            string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
    
            var albums = new List<Album>();
            using var connection = new SpannerConnection(connectionString);
            using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
    
            using var reader = await cmd.ExecuteReaderAsync();
            while (await reader.ReadAsync())
            {
                albums.Add(new Album
                {
                    AlbumId = reader.GetFieldValue<int>("AlbumId"),
                    SingerId = reader.GetFieldValue<int>("SingerId"),
                    AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
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
     "fmt"
     "io"
    
     "cloud.google.com/go/spanner"
     "google.golang.org/api/iterator"
    )
    
    func query(w io.Writer, db string) error {
     ctx := context.Background()
     client, err := spanner.NewClient(ctx, db)
     if err != nil {
         return err
     }
     defer client.Close()
    
     stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
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
         var albumTitle string
         if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
             return err
         }
         fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
     }
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    class QueryData {
      static void queryData(String host, int port, String database) throws SQLException {
        String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
        try (Connection connection = DriverManager.getConnection(connectionUrl)) {
          try (ResultSet resultSet =
              connection
                  .createStatement()
                  .executeQuery("SELECT singer_id, album_id, album_title FROM albums")) {
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
    
    const query = {
      sql: 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums',
    };
    
    // Queries rows from the Albums table
    try {
      const [rows] = await database.run(query);
    
      rows.forEach(row => {
        const json = row.toJSON();
        console.log(
          `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`
        );
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
    
    async function queryData(host: string, port: number, database: string): Promise<void> {
      // Connect to Spanner through PGAdapter.
      const connection = new Client({
        host: host,
        port: port,
        database: database,
      });
      await connection.connect();
    
      const result = await connection.query("SELECT singer_id, album_id, album_title " +
          "FROM albums");
      for (const row of result.rows) {
        console.log(`${row["singer_id"]} ${row["album_id"]} ${row["album_title"]}`);
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
     * Example:
     * ```
     * query_data($instanceId, $databaseId);
     * ```
     *
     * @param string $instanceId The Spanner instance ID.
     * @param string $databaseId The Spanner database ID.
     */
    function query_data(string $instanceId, string $databaseId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
        $database = $instance->database($databaseId);
    
        $results = $database->execute(
            'SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
        );
    
        foreach ($results as $row) {
            printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
                $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
        }
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import string
    import psycopg
    
    
    def query_data(host: string, port: int, database: string):
        with psycopg.connect("host={host} port={port} dbname={database} "
                             "sslmode=disable".format(host=host,
                                                      port=port,
                                                      database=database)) as conn:
            conn.autocommit = True
            with conn.cursor() as cur:
                cur.execute("SELECT singer_id, album_id, album_title "
                            "FROM albums")
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
    
    client.execute("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").rows.each do |row|
      puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:AlbumTitle]}"
    end

### Rust

    use google_cloud_spanner::client::DatabaseClient;
    use google_cloud_spanner::statement::Statement;
    
    pub async fn sample(client: &DatabaseClient) -> anyhow::Result<()> {
        let statement = Statement::builder("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").build();
        let transaction = client.single_use().build();
        let mut result_set = transaction.execute_query(statement).await?;
    
        println!("Listing albums:");
        while let Some(row) = result_set.next().await.transpose()? {
            let singer_id: i64 = row.get("SingerId");
            let album_id: i64 = row.get("AlbumId");
            let album_title: String = row.get("AlbumTitle");
            println!("SingerId: {singer_id}, AlbumId: {album_id}, AlbumTitle: {album_title}");
        }
        println!("Done listing albums.");
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
