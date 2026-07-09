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

    public static async Task ReadOnlyTransaction(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Start a read-only transaction on this connection.
        await using var transaction = await connection.BeginReadOnlyTransactionAsync();
    
        await using var command = connection.CreateCommand();
        command.Transaction = transaction;
        command.CommandText = "SELECT SingerId, AlbumId, AlbumTitle " +
                              "FROM Albums " +
                              "ORDER BY SingerId, AlbumId";
        await using (var reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine(
                    $"{reader["SingerId"]} {reader["AlbumId"]} {reader["AlbumTitle"]}");
            }
        }
    
        // Execute another query using the same read-only transaction.
        command.CommandText = "SELECT SingerId, AlbumId, AlbumTitle " +
                              "FROM Albums " +
                              "ORDER BY AlbumTitle";
        await using (var reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine(
                    $"{reader["SingerId"]} {reader["AlbumId"]} {reader["AlbumTitle"]}");
            }
        }
    
        // End the read-only transaction by calling Commit.
        await transaction.CommitAsync();
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    func readOnlyTransaction(ctx context.Context, w io.Writer, client *spanner.Client) error {
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

    static void readOnlyTransaction(
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
        // Set AutoCommit=false to enable transactions.
        connection.setAutoCommit(false);
        // This SQL statement instructs the JDBC driver to use
        // a read-only transaction.
        connection.createStatement().execute("SET TRANSACTION READ ONLY");
    
        try (ResultSet resultSet =
            connection
                .createStatement()
                .executeQuery(
                    "SELECT SingerId, AlbumId, AlbumTitle "
                        + "FROM Albums "
                        + "ORDER BY SingerId, AlbumId")) {
          while (resultSet.next()) {
            System.out.printf(
                "%d %d %s\n",
                resultSet.getLong("SingerId"),
                resultSet.getLong("AlbumId"),
                resultSet.getString("AlbumTitle"));
          }
        }
        try (ResultSet resultSet =
            connection
                .createStatement()
                .executeQuery(
                    "SELECT SingerId, AlbumId, AlbumTitle "
                        + "FROM Albums "
                        + "ORDER BY AlbumTitle")) {
          while (resultSet.next()) {
            System.out.printf(
                "%d %d %s\n",
                resultSet.getLong("SingerId"),
                resultSet.getLong("AlbumId"),
                resultSet.getString("AlbumTitle"));
          }
        }
        // End the read-only transaction by calling commit().
        connection.commit();
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

    def read_only_transaction(instance_id, database_id):
        """Reads data inside of a read-only transaction.
    
        Within the read-only transaction, or "snapshot", the application sees
        consistent view of the database at a particular timestamp.
        """
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
        database = instance.database(database_id)
    
        with database.snapshot(multi_use=True) as snapshot:
            # Read using SQL.
            results = snapshot.execute_sql(
                "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
            )
    
            print("Results from first read:")
            for row in results:
                print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
    
            # Perform another read using the `read` method. Even if the data
            # is updated in-between the reads, the snapshot ensures that both
            # return the same data.
            keyset = spanner.KeySet(all_=True)
            results = snapshot.read(
                table="Albums", columns=("SingerId", "AlbumId", "AlbumTitle"), keyset=keyset
            )
    
            print("Results from second read:")
            for row in results:
                print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))

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
