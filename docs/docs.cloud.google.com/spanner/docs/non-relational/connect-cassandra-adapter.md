This page describes the Cassandra Adapter and explains how to use and connect Spanner from it.

The Cassandra Adapter is designed to run on the same machine as your application. The adapter exposes an endpoint on localhost that supports the Cassandra Query Language (CQL) wire protocol. It translates the CQL wire protocol into gRPC, the Spanner wire protocol. With this proxy running locally, a Cassandra client can connect to a Spanner database.

You can start the Cassandra Adapter in the following ways:

  - In-process with your Go application
  - In-process with your Java application
  - As a standalone process
  - In a Docker container

## Before you begin

Before starting the Cassandra Adapter, ensure that you have authenticated with either a user account or service account on the machine where Cassandra Adapter will be running. If you are using a service account, you must know the location of the JSON key file (the credentials file). You set the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable to specify the credentials path. Before starting the Cassandra Adapter, ensure that you have authenticated with either a user account or service account on the machine where Cassandra Adapter will be running. If you are using a service account, you must know the location of the JSON key file (the credentials file). You set the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable to specify the credentials path.

For more information, see:

  - [Authenticate to Spanner](/spanner/docs/authentication)
  - [Authorize the gcloud CLI](/sdk/docs/authorizing)

## Connect the Cassandra Adapter to your application

The Cassandra Adapter requires the following information:

  - Project name
  - Spanner instance name
  - Database to connect to

If you use Docker, you need the path for a JSON-formatted credentials file (key file).

### Java in-process

1.  If you are using a service account for authentication, ensure that the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable is set to the path of the credentials file.

2.  For Java applications, you can link the Cassandra Adapter to the application directly by adding `  google-cloud-spanner-cassandra  ` as a dependency to your project.

For Maven, add the following new dependency under the `  <dependencies>  ` section:

``` markdown
<dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-spanner-cassandra</artifactId>
    <version>1.1.0</version>
</dependency>
```

For Gradle, add the following:

``` markdown
dependencies {
    implementation 'com.google.cloud:google-cloud-spanner-cassandra:1.1.0'
}
```

1.  Modify your `  CqlSession  ` creation code. Instead of using `  CqlSessionBuilder  ` , use `  SpannerCqlSessionBuilder  ` and provide the Spanner database URI:
    
    ``` java
    import com.datastax.oss.driver.api.core.CqlSession;
    import com.datastax.oss.driver.api.core.config.DefaultDriverOption;
    import com.datastax.oss.driver.api.core.config.DriverConfigLoader;
    import com.datastax.oss.driver.api.core.cql.ResultSet;
    import com.datastax.oss.driver.api.core.cql.Row;
    import com.google.cloud.spanner.adapter.SpannerCqlSession;
    import java.net.InetSocketAddress;
    import java.time.Duration;
    import java.util.Random;
    
    // This sample assumes your spanner database <my_db> contains a table <users>
    // with the following schema:
    
    // CREATE TABLE users (
    //  id        INT64          OPTIONS (cassandra_type = 'int'),
    //  active    BOOL           OPTIONS (cassandra_type = 'boolean'),
    //  username  STRING(MAX)    OPTIONS (cassandra_type = 'text'),
    // ) PRIMARY KEY (id);
    
    class QuickStartSample {
    
      public static void main(String[] args) {
    
        // TODO(developer): Replace these variables before running the sample.
        final String projectId = "my-gcp-project";
        final String instanceId = "my-spanner-instance";
        final String databaseId = "my_db";
    
        final String databaseUri =
            String.format("projects/%s/instances/%s/databases/%s", projectId, instanceId, databaseId);
    
        try (CqlSession session =
            SpannerCqlSession.builder() // `SpannerCqlSession` instead of `CqlSession`
                .setDatabaseUri(databaseUri) // Set spanner database URI.
                .addContactPoint(new InetSocketAddress("localhost", 9042))
                .withLocalDatacenter("datacenter1")
                .withKeyspace(databaseId) // Keyspace name should be the same as spanner database name
                .withConfigLoader(
                    DriverConfigLoader.programmaticBuilder()
                        .withString(DefaultDriverOption.PROTOCOL_VERSION, "V4")
                        .withDuration(
                            DefaultDriverOption.CONNECTION_INIT_QUERY_TIMEOUT, Duration.ofSeconds(5))
                        .build())
                .build()) {
    
          final int randomUserId = new Random().nextInt(Integer.MAX_VALUE);
    
          System.out.printf("Inserting user with ID: %d%n", randomUserId);
    
          // INSERT data
          session.execute(
              "INSERT INTO users (id, active, username) VALUES (?, ?, ?)",
              randomUserId,
              true,
              "John Doe");
    
          System.out.printf("Successfully inserted user: %d%n", randomUserId);
          System.out.printf("Querying user: %d%n", randomUserId);
    
          // SELECT data
          ResultSet rs =
              session.execute("SELECT id, active, username FROM users WHERE id = ?", randomUserId);
    
          // Get the first row from the result set
          Row row = rs.one();
    
          System.out.printf(
              "%d %b %s%n", row.getInt("id"), row.getBoolean("active"), row.getString("username"));
    
        } catch (Exception e) {
          e.printStackTrace();
        }
      }
    }
    ```

### Go in-process

For Go applications, you need to make a one-line change to the cluster initialization file to integrate the Spanner Cassandra Go client. You can then link the Cassandra Adapter to the application directly.

1.  Import the adapter's `  spanner  ` package from the Spanner Cassandra Go client in your Go application.

<!-- end list -->

``` text
import spanner "github.com/googleapis/go-spanner-cassandra/cassandra/gocql"
```

1.  Modify your cluster creation code to use `  spanner.NewCluster  ` instead of `  gocql.NewCluster  ` , and provide the Spanner database URI:
    
    ``` go
    import (
     "fmt"
     "io"
     "math"
     "math/rand/v2"
     "time"
    
     spanner "github.com/googleapis/go-spanner-cassandra/cassandra/gocql"
    )
    
    // This sample assumes your spanner database <your_db> contains a table <users>
    // with the following schema:
    //
    // CREATE TABLE users (
    //  id          INT64          OPTIONS (cassandra_type = 'int'),
    //  active      BOOL           OPTIONS (cassandra_type = 'boolean'),
    //  username    STRING(MAX)    OPTIONS (cassandra_type = 'text'),
    // ) PRIMARY KEY (id);
    
    func quickStart(databaseURI string, w io.Writer) error {
     opts := &spanner.Options{
         DatabaseUri: databaseURI,
     }
     cluster := spanner.NewCluster(opts)
     if cluster == nil {
         return fmt.Errorf("failed to create cluster")
     }
     defer spanner.CloseCluster(cluster)
    
     // You can still configure your cluster as usual after connecting to your
     // spanner database
     cluster.Timeout = 5 * time.Second
     cluster.Keyspace = "your_db_name"
    
     session, err := cluster.CreateSession()
    
     if err != nil {
         return err
     }
    
     randomUserId := rand.IntN(math.MaxInt32)
     if err = session.Query("INSERT INTO users (id, active, username) VALUES (?, ?, ?)",
                    randomUserId, true, "John Doe").
         Exec(); err != nil {
         return err
     }
    
     var id int
     var active bool
     var username string
     if err = session.Query("SELECT id, active, username FROM users WHERE id = ?",
                    randomUserId).
         Scan(&id, &active, &username); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %v %s\n", id, active, username)
     return nil
    }
    ```

You can configure your cluster as usual after connecting to your Spanner database.

### Standalone

1.  Clone the repository:

<!-- end list -->

``` text
git clone https://github.com/googleapis/go-spanner-cassandra.git
cd go-spanner-cassandra
```

1.  Run the `  cassandra_launcher.go  ` with the required `  -db  ` flag:

<!-- end list -->

``` text
go run cassandra_launcher.go \
-db "projects/my_project/instances/my_instance/databases/my_database"
```

1.  Replace `  -db  ` with your Spanner database URI.

### Docker

Start the Cassandra Adapter with the following command.

``` text
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
docker run -d -p 9042:9042 \
-e GOOGLE_APPLICATION_CREDENTIALS \
-v ${GOOGLE_APPLICATION_CREDENTIALS}:${GOOGLE_APPLICATION_CREDENTIALS}:ro \
gcr.io/cloud-spanner-adapter/cassandra-adapter \
-db DATABASE_URI
```

The following list contains the most frequently used startup options for the Spanner Cassandra Adapter:

  - `  -db <DatabaseUri>  `

The Spanner database URI (required). This specifies the Spanner database that the client connects to. For example, `  projects/YOUR_PROJECT/instances/YOUR_INSTANCE/databases/YOUR_DATABASE  ` .

  - `  -tcp <TCPEndpoint>  `

The client proxy listener address. This defines the TCP endpoint where the client listens for incoming Cassandra client connections. Default: `  localhost:9042  `

  - `  -grpc-channels <NumGrpcChannels>  `

The number of gRPC channels to use when connecting to Spanner. Default: 4

For example, the following command starts the Cassandra Adapter on port `  9042  ` using the application credentials, and connects the adapter to the `  projects/my_project/instances/my_instance/databases/my_database  ` database:

``` text
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
docker run -d -p 9042:9042 \
-e GOOGLE_APPLICATION_CREDENTIALS \
-v ${GOOGLE_APPLICATION_CREDENTIALS}:${GOOGLE_APPLICATION_CREDENTIALS}:ro \
gcr.io/cloud-spanner-adapter/cassandra-adapter \
-db projects/my_project/instances/my_instance/databases/my_database
```

## Recommendations

The following recommendations help you to improve your experience with Cassandra Adapter. These recommendations are focused on Java, specifically the [Cassandra client driver for Java version 4](https://github.com/apache/cassandra-java-driver/tree/4.x) .

### Increase request timeout

A request timeout of five seconds or longer provides a better experience with the Cassandra Adapter than the default value of two seconds.

``` text
# Sample application.conf: increases request timeout to five seconds
datastax-java-driver {
  basic {
    request {
      timeout = 5 seconds
    }
  }
}
```

### Tune connection pooling

The default configurations for maximum connection count and maximum concurrent requests per connection or host are appropriate for development, testing, and low-volume production or staging environments. However, we recommend that you increase these values, as the Cassandra Adapter masquerades as a single node, unlike a pool of nodes within Cassandra cluster.

Increasing these values enables more concurrent connections between the client and Cassandra Interface. This can prevent connection pool exhaustion under heavy load.

``` text
# Sample application.conf: increases maximum number of requests that can be
# executed concurrently on a connection
advanced.connection {
  max-requests-per-connection = 32000
  pool {
    local.size = 10
  }
}
```

### Tune gRPC channels

gRPC channels are used by the Spanner client for communication. One gRPC channel is roughly equivalent to a TCP connection. One gRPC channel can handle up to 100 concurrent requests. This means that an application will need at least as many gRPC channels as the number of concurrent requests the application will execute, divided by 100.

### Disable token-aware routing

Drivers that use token-aware load balancing might print a warning or might not work when using the Cassandra Adapter. Because the Cassandra Adapter masquerades as a single node, the Cassandra Adapter doesn't always work well with token-aware drivers that expect there to be at least a replication factor number of nodes in the cluster. Some drivers might print a warning (which can be ignored) and fallback to something like round-robin balancing policy, while other drivers might fail with an error. For the drivers that fail with an error, you must disable token-aware or configure the round-robin load balancing policy.

``` text
# Sample application.conf: disables token-aware routing
metadata {
  token-map {
    enabled = false
  }
}
```

### Pin protocol version to V4

The Cassandra Adapter is compatible with any [CQL Binary v4 wire protocol](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v4.spec) compliant, open-source Apache Cassandra client driver. Make sure to pin `  PROTOCOL_VERSION  ` to `  V4  ` , otherwise you might see connection errors.

``` text
# Sample application.conf: overrides protocol version to V4
datastax-java-driver {
  advanced.protocol.version = V4
}
```

## What's next

  - Learn more about the Spanner Cassandra Adapter for Java in the [java-spanner-cassandra GitHub repository](https://github.com/googleapis/java-spanner-cassandra) .
  - Learn more about the Spanner Cassandra Adapter for Go in the [go-spanner-cassandra GitHub repository](https://github.com/googleapis/go-spanner-cassandra) .
  - See a comparison between [Cassandra and Spanner concepts and architecture](/spanner/docs/non-relational/cassandra-overview) .
  - Learn how to [Migrate from Cassandra to Spanner](/spanner/docs/non-relational/migrate-from-cassandra-to-spanner) .
