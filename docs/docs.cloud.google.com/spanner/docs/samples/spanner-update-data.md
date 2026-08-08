---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-update-data
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-update-data
title: Mutations update data
description: Update data by using mutations.
data_source: docs.cloud.google.com
---

Update data by using mutations.

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

    void UpdateData(google::cloud::spanner::Client client) {
      //! [commit-with-mutations]
      namespace spanner = ::google::cloud::spanner;
      auto commit_result = client.Commit(spanner::Mutations{
          spanner::UpdateMutationBuilder("Albums",
                                         {"SingerId", "AlbumId", "MarketingBudget"})
              .EmplaceRow(1, 1, 100000)
              .EmplaceRow(2, 2, 500000)
              .Build()});
      if (!commit_result) throw std::move(commit_result).status();
      //! [commit-with-mutations]
      std::cout << "Update was successful [spanner_update_data]\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    using Google.Cloud.Spanner.Data;
    using System;
    using System.Threading.Tasks;
    
    public class UpdateDataAsyncSample
    {
        public async Task<int> UpdateDataAsync(string projectId, string instanceId, string databaseId)
        {
            string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
    
            using var connection = new SpannerConnection(connectionString);
    
            var rowCount = 0;
            SpannerCommand cmd = connection.CreateDmlCommand(
                "UPDATE Albums SET MarketingBudget = @MarketingBudget "
                + "WHERE SingerId = 1 and AlbumId = 1");
            cmd.Parameters.Add("MarketingBudget", SpannerDbType.Int64, 100000);
            rowCount += await cmd.ExecuteNonQueryAsync();
    
            cmd = connection.CreateDmlCommand(
                "UPDATE Albums SET MarketingBudget = @MarketingBudget "
                + "WHERE SingerId = 2 and AlbumId = 2");
            cmd.Parameters.Add("MarketingBudget", SpannerDbType.Int64, 500000);
            rowCount += await cmd.ExecuteNonQueryAsync();
    
            Console.WriteLine("Data Updated.");
            return rowCount;
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
    
     "cloud.google.com/go/spanner"
     spannerdriver "github.com/googleapis/go-sql-spanner"
    )
    
    func UpdateDataWithMutations(ctx context.Context, w io.Writer, databaseName string) error {
     db, err := sql.Open("spanner", databaseName)
     if err != nil {
         return err
     }
     defer db.Close()
    
     // Get a connection so that we can get access to the Spanner specific
     // connection interface SpannerConn.
     conn, err := db.Conn(ctx)
     if err != nil {
         return err
     }
     defer conn.Close()
    
     cols := []string{"SingerId", "AlbumId", "MarketingBudget"}
     mutations := []*spanner.Mutation{
         spanner.Update("Albums", cols, []interface{}{1, 1, 100000}),
         spanner.Update("Albums", cols, []interface{}{2, 2, 500000}),
     }
     if err := conn.Raw(func(driverConn interface{}) error {
         spannerConn, ok := driverConn.(spannerdriver.SpannerConn)
         if !ok {
             return fmt.Errorf("unexpected driver connection %v, "+
                 "expected SpannerConn", driverConn)
         }
         _, err = spannerConn.Apply(ctx, mutations)
         return err
     }); err != nil {
         return err
     }
     fmt.Fprintf(w, "Updated %v albums\n", len(mutations))
    
     return nil
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import java.io.IOException;
    import java.io.StringReader;
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.SQLException;
    import org.postgresql.PGConnection;
    import org.postgresql.copy.CopyManager;
    
    class UpdateDataWithCopy {
    
      static void updateDataWithCopy(String host, int port, String database)
          throws SQLException, IOException {
        String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
        try (Connection connection = DriverManager.getConnection(connectionUrl)) {
          // Unwrap the PostgreSQL JDBC connection interface to get access to
          // a CopyManager.
          PGConnection pgConnection = connection.unwrap(PGConnection.class);
          CopyManager copyManager = pgConnection.getCopyAPI();
    
          // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
          // will succeed even if it exceeds Spanner's mutation limit per transaction.
          connection
              .createStatement()
              .execute("set spanner.autocommit_dml_mode='partitioned_non_atomic'");
    
          // Instruct PGAdapter to use insert-or-update for COPY statements.
          // This enables us to use COPY to update existing data.
          connection.createStatement().execute("set spanner.copy_upsert=true");
    
          // COPY uses mutations to insert or update existing data in Spanner.
          long numAlbums =
              copyManager.copyIn(
                  "COPY albums (singer_id, album_id, marketing_budget) FROM STDIN",
                  new StringReader("1\t1\t100000\n" + "2\t2\t500000\n"));
          System.out.printf("Updated %d albums\n", numAlbums);
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
    
    // Update a row in the Albums table
    // Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
    // must be converted to strings before being inserted as INT64s
    const albumsTable = database.table('Albums');
    
    try {
      await albumsTable.update([
        {SingerId: '1', AlbumId: '1', MarketingBudget: '100000'},
        {SingerId: '2', AlbumId: '2', MarketingBudget: '500000'},
      ]);
      console.log('Updated data.');
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
    import { pipeline } from 'node:stream/promises'
    import { from as copyFrom } from 'pg-copy-streams'
    import {Readable} from "stream";
    
    async function updateDataWithCopy(host: string, port: number, database: string): Promise<void> {
      const connection = new Client({
        host: host,
        port: port,
        database: database,
      });
      await connection.connect();
    
      // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
      // will succeed even if it exceeds Spanner's mutation limit per transaction.
      await connection.query("set spanner.autocommit_dml_mode='partitioned_non_atomic'");
    
      // Instruct PGAdapter to use insert-or-update for COPY statements.
      // This enables us to use COPY to update existing data.
      await connection.query("set spanner.copy_upsert=true");
    
      // Copy data to Spanner using the COPY command.
      const copyStream = copyFrom('COPY albums (singer_id, album_id, marketing_budget) FROM STDIN');
      const ingestStream = connection.query(copyStream);
    
      // Create a source stream and attach the source to the destination.
      const sourceStream = new Readable();
      const operation = pipeline(sourceStream, ingestStream);
      // Manually push data to the source stream to write data to Spanner.
      sourceStream.push("1\t1\t100000\n");
      sourceStream.push("2\t2\t500000\n");
      // Push a 'null' to indicate the end of the stream.
      sourceStream.push(null);
      // Wait for the copy operation to finish.
      await operation;
      console.log(`Updated ${copyStream.rowCount} albums`);
    
      // Close the connection.
      await connection.end();
    }

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    function update_data_with_copy(string $host, string $port, string $database): void
    {
        $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
        $connection = new PDO($dsn);
    
        // Instruct PGAdapter to use insert-or-update for COPY statements.
        // This enables us to use COPY to update data.
        $connection->exec("set spanner.copy_upsert=true");
    
        // COPY uses mutations to insert or update existing data in Spanner.
        $connection->pgsqlCopyFromArray(
            "albums",
            ["1\t1\t100000", "2\t2\t500000"],
            "\t",
            "\\\\N",
            "singer_id, album_id, marketing_budget",
        );
        print("Updated 2 albums\n");
    
        $connection = null;
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import string
    import psycopg
    
    
    def update_data_with_copy(host: string, port: int, database: string):
        with psycopg.connect("host={host} port={port} dbname={database} "
                             "sslmode=disable".format(host=host,
                                                      port=port,
                                                      database=database)) as conn:
            conn.autocommit = True
            with conn.cursor() as cur:
                # Instruct PGAdapter to use insert-or-update for COPY statements.
                # This enables us to use COPY to update data.
                cur.execute("set spanner.copy_upsert=true")
    
                # COPY uses mutations to insert or update existing data in Spanner.
                with cur.copy("COPY albums (singer_id, album_id, marketing_budget) "
                              "FROM STDIN") as copy:
                    copy.write_row((1, 1, 100000))
                    copy.write_row((2, 2, 500000))
                print("Updated %d albums" % cur.rowcount)

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    # project_id  = "Your Google Cloud project ID"
    # instance_id = "Your Spanner instance ID"
    # database_id = "Your Spanner database ID"
    
    require "google/cloud/spanner"
    
    spanner = Google::Cloud::Spanner.new project: project_id
    client  = spanner.client instance_id, database_id
    
    client.commit do |c|
      c.update "Albums", [
        { SingerId: 1, AlbumId: 1, MarketingBudget: 100_000 },
        { SingerId: 2, AlbumId: 2, MarketingBudget: 500_000 }
      ]
    end
    
    puts "Updated data"

### Rust

    use google_cloud_spanner::client::DatabaseClient;
    use google_cloud_spanner::mutation::Mutation;
    
    pub async fn sample(client: &DatabaseClient) -> anyhow::Result<()> {
        let mutations = vec![
            Mutation::new_update_builder("Albums")
                .set("SingerId")
                .to(1)
                .set("AlbumId")
                .to(1)
                .set("MarketingBudget")
                .to(100000)
                .build(),
            Mutation::new_update_builder("Albums")
                .set("SingerId")
                .to(2)
                .set("AlbumId")
                .to(2)
                .set("MarketingBudget")
                .to(500000)
                .build(),
        ];
    
        println!("Updating MarketingBudget on Albums...");
        let write_transaction = client.write_only_transaction().build();
        write_transaction.write(mutations).await?;
        println!("Updated budget successfully.");
    
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
