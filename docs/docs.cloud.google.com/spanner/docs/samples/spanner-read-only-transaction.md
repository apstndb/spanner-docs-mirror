---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-read-only-transaction
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-read-only-transaction
title: Read-only transaction
description: Use a read-only transaction.
data_source: docs.cloud.google.com
---

Use a read-only transaction.

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
  - [Transactions overview](https://docs.cloud.google.com/spanner/docs/transactions)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void ReadOnlyTransaction(google::cloud::spanner::Client client) {
      namespace spanner = ::google::cloud::spanner;
      auto read_only = spanner::MakeReadOnlyTransaction();
    
      spanner::SqlStatement select(
          "SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
      using RowType = std::tuple<std::int64_t, std::int64_t, std::string>;
    
      // Read#1.
      auto rows1 = client.ExecuteQuery(read_only, select);
      std::cout << "Read 1 results\n";
      for (auto& row : spanner::StreamOf<RowType>(rows1)) {
        if (!row) throw std::move(row).status();
        std::cout << "SingerId: " << std::get<0>(*row)
                  << " AlbumId: " << std::get<1>(*row)
                  << " AlbumTitle: " << std::get<2>(*row) << "\n";
      }
      // Read#2. Even if changes occur in-between the reads the transaction ensures
      // that Read #1 and Read #2 return the same data.
      auto rows2 = client.ExecuteQuery(read_only, select);
      std::cout << "Read 2 results\n";
      for (auto& row : spanner::StreamOf<RowType>(rows2)) {
        if (!row) throw std::move(row).status();
        std::cout << "SingerId: " << std::get<0>(*row)
                  << " AlbumId: " << std::get<1>(*row)
                  << " AlbumTitle: " << std::get<2>(*row) << "\n";
      }
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    using Google.Cloud.Spanner.Data;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using System.Transactions;
    
    public class QueryDataWithTransactionAsyncSample
    {
        public class Album
        {
            public int SingerId { get; set; }
            public int AlbumId { get; set; }
            public string AlbumTitle { get; set; }
        }
    
        public async Task<List<Album>> QueryDataWithTransactionAsync(string projectId, string instanceId, string databaseId)
        {
            string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
    
            var albums = new List<Album>();
            using TransactionScope scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
            using var connection = new SpannerConnection(connectionString);
    
            // Opens the connection so that the Spanner transaction included in the TransactionScope
            // is read-only TimestampBound.Strong.
            await connection.OpenAsync(SpannerTransactionCreationOptions.ReadOnly, options: null, cancellationToken: default);
            using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
    
            // Read #1.
            using (var reader = await cmd.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    Console.WriteLine("SingerId : " + reader.GetFieldValue<string>("SingerId")
                        + " AlbumId : " + reader.GetFieldValue<string>("AlbumId")
                        + " AlbumTitle : " + reader.GetFieldValue<string>("AlbumTitle"));
                }
            }
    
            // Read #2. Even if changes occur in-between the reads,
            // the transaction ensures that Read #1 and Read #2
            // return the same data.
            using (var reader = await cmd.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    albums.Add(new Album
                    {
                        AlbumId = reader.GetFieldValue<int>("AlbumId"),
                        SingerId = reader.GetFieldValue<int>("SingerId"),
                        AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
                    });
                }
            }
            scope.Complete();
            Console.WriteLine("Transaction complete.");
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
    
    func readOnlyTransaction(w io.Writer, db string) error {
     ctx := context.Background()
     client, err := spanner.NewClient(ctx, db)
     if err != nil {
         return err
     }
     defer client.Close()
    
     ro := client.ReadOnlyTransaction()
     defer ro.Close()
     stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
     iter := ro.Query(ctx, stmt)
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             break
         }
         if err != nil {
             return err
         }
         var singerID int64
         var albumID int64
         var albumTitle string
         if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
             return err
         }
         fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
     }
    
     iter = ro.Read(ctx, "Albums", spanner.AllKeys(), []string{"SingerId", "AlbumId", "AlbumTitle"})
     defer iter.Stop()
     for {
         row, err := iter.Next()
         if err == iterator.Done {
             return nil
         }
         if err != nil {
             return err
         }
         var singerID int64
         var albumID int64
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
    
    class ReadOnlyTransaction {
      static void readOnlyTransaction(String host, int port, String database) throws SQLException {
        String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
        try (Connection connection = DriverManager.getConnection(connectionUrl)) {
          // Set AutoCommit=false to enable transactions.
          connection.setAutoCommit(false);
          // This SQL statement instructs the JDBC driver to use
          // a read-only transaction.
          connection.createStatement().execute("set transaction read only");
    
          try (ResultSet resultSet =
              connection
                  .createStatement()
                  .executeQuery(
                      "SELECT singer_id, album_id, album_title "
                          + "FROM albums "
                          + "ORDER BY singer_id, album_id")) {
            while (resultSet.next()) {
              System.out.printf(
                  "%d %d %s\n",
                  resultSet.getLong("singer_id"),
                  resultSet.getLong("album_id"),
                  resultSet.getString("album_title"));
            }
          }
          try (ResultSet resultSet =
              connection
                  .createStatement()
                  .executeQuery(
                      "SELECT singer_id, album_id, album_title "
                          + "FROM albums "
                          + "ORDER BY album_title")) {
            while (resultSet.next()) {
              System.out.printf(
                  "%d %d %s\n",
                  resultSet.getLong("singer_id"),
                  resultSet.getLong("album_id"),
                  resultSet.getString("album_title"));
            }
          }
          // End the read-only transaction by calling commit().
          connection.commit();
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
    let database;
    try {
      // Gets a reference to a Cloud Spanner instance and database
      const instance = spanner.instance(instanceId);
      database = instance.database(databaseId);
    
      // Gets a transaction object that captures the database state
      // at a specific point in time
      const [transaction] = await database.getSnapshot();
    
      try {
        const queryOne = 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums';
    
        // Read #1, using SQL
        const [qOneRows] = await transaction.run(queryOne);
    
        qOneRows.forEach(row => {
          const json = row.toJSON();
          console.log(
            `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`
          );
        });
    
        const queryTwo = {
          columns: ['SingerId', 'AlbumId', 'AlbumTitle'],
        };
    
        // Read #2, using the `read` method. Even if changes occur
        // in-between the reads, the transaction ensures that both
        // return the same data.
        const [qTwoRows] = await transaction.read('Albums', queryTwo);
    
        qTwoRows.forEach(row => {
          const json = row.toJSON();
          console.log(
            `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`
          );
        });
    
        console.log('Successfully executed read-only transaction.');
      } finally {
        // Ensure the transaction is released
        transaction.end();
      }
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
    
    async function readOnlyTransaction(host: string, port: number, database: string): Promise<void> {
      const connection = new Client({
        host: host,
        port: port,
        database: database,
      });
      await connection.connect();
    
      // Start a transaction.
      await connection.query("begin");
      // This SQL statement instructs the PGAdapter to make it a read-only transaction.
      await connection.query("set transaction read only");
    
      const albumsOrderById = await connection.query(
          "SELECT singer_id, album_id, album_title "
          + "FROM albums "
          + "ORDER BY singer_id, album_id");
      for (const row of albumsOrderById.rows) {
        console.log(`${row["singer_id"]} ${row["album_id"]} ${row["album_title"]}`);
      }
      const albumsOrderByTitle = await connection.query(
          "SELECT singer_id, album_id, album_title "
          + "FROM albums "
          + "ORDER BY album_title");
      for (const row of albumsOrderByTitle.rows) {
        console.log(`${row["singer_id"]} ${row["album_id"]} ${row["album_title"]}`);
      }
      // End the read-only transaction by executing commit.
      await connection.query("commit");
    
      // Close the connection.
      await connection.end();
    }

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    use Google\Cloud\Spanner\SpannerClient;
    
    /**
     * Reads data inside of a read-only transaction.
     *
     * Within the read-only transaction, or "snapshot", the application sees
     * consistent view of the database at a particular timestamp.
     * Example:
     * ```
     * read_only_transaction($instanceId, $databaseId);
     * ```
     *
     * @param string $instanceId The Spanner instance ID.
     * @param string $databaseId The Spanner database ID.
     */
    function read_only_transaction(string $instanceId, string $databaseId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
        $database = $instance->database($databaseId);
    
        $snapshot = $database->snapshot();
        $results = $snapshot->execute(
            'SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
        );
        print('Results from the first read:' . PHP_EOL);
        foreach ($results as $row) {
            printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
                $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
        }
    
        // Perform another read using the `read` method. Even if the data
        // is updated in-between the reads, the snapshot ensures that both
        // return the same data.
        $keySet = $spanner->keySet(['all' => true]);
        $results = $database->read(
            'Albums',
            $keySet,
            ['SingerId', 'AlbumId', 'AlbumTitle']
        );
    
        print('Results from the second read:' . PHP_EOL);
        foreach ($results->rows() as $row) {
            printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
                $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
        }
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import string
    import psycopg
    
    
    def read_only_transaction(host: string, port: int, database: string):
        with (psycopg.connect("host={host} port={port} dbname={database} "
                             "sslmode=disable".format(host=host,
                                                      port=port,
                                                      database=database)) as conn):
            # Set autocommit=False to enable transactions.
            conn.autocommit = False
    
            with conn.cursor() as cur:
                # Change the current transaction to a read-only transaction.
                # This statement can only be executed at the start of a transaction.
                cur.execute("set transaction read only")
    
                # The following two queries use the same read-only transaction.
                cur.execute("select singer_id, album_id, album_title "
                            "from albums "
                            "order by singer_id, album_id")
                for album in cur:
                    print(album)
    
                cur.execute("select singer_id, album_id, album_title "
                            "from albums "
                            "order by album_title")
                for album in cur:
                    print(album)
    
            # Read-only transactions must also be committed or rolled back to mark
            # the end of the transaction. There is no semantic difference between
            # rolling back or committing a read-only transaction.
            conn.commit()

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    # project_id  = "Your Google Cloud project ID"
    # instance_id = "Your Spanner instance ID"
    # database_id = "Your Spanner database ID"
    
    require "google/cloud/spanner"
    
    spanner = Google::Cloud::Spanner.new project: project_id
    client  = spanner.client instance_id, database_id
    
    client.snapshot do |snapshot|
      snapshot.execute("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").rows.each do |row|
        puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
      end
    
      # Even if changes occur in-between the reads, the transaction ensures that
      # both return the same data.
      snapshot.read("Albums", [:AlbumId, :AlbumTitle, :SingerId]).rows.each do |row|
        puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
      end
    end

### Rust

    use google_cloud_spanner::client::DatabaseClient;
    use google_cloud_spanner::key::KeySet;
    use google_cloud_spanner::read::ReadRequest;
    use google_cloud_spanner::statement::Statement;
    
    pub async fn sample(client: &DatabaseClient) -> anyhow::Result<()> {
        let transaction = client.read_only_transaction().build().await?;
    
        // 1. Execute a query using the read-only transaction
        let statement = Statement::builder("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").build();
        let mut result_set = transaction.execute_query(statement).await?;
        println!("Results from query:");
        while let Some(row) = result_set.next().await.transpose()? {
            let singer_id: i64 = row.get(0);
            let album_id: i64 = row.get(1);
            let album_title: String = row.get(2);
            println!("{singer_id} {album_id} {album_title}");
        }
    
        // 2. Execute a read using the same read-only transaction
        let read_request = ReadRequest::builder("Albums", ["SingerId", "AlbumId", "AlbumTitle"])
            .with_keys(KeySet::all())
            .build();
        let mut result_set = transaction.execute_read(read_request).await?;
        println!("Results from read:");
        while let Some(row) = result_set.next().await.transpose()? {
            let singer_id: i64 = row.get(0);
            let album_id: i64 = row.get(1);
            let album_title: String = row.get(2);
            println!("{singer_id} {album_id} {album_title}");
        }
    
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
