Connect by using functions.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Create a Cloud Run function that returns Spanner results](/run/docs/tutorials/function-sends-query-to-spanner-database)
  - [Using Spanner with Cloud Run functions (1st gen)](/functions/1stgendocs/tutorials/use-cloud-spanner-1st-gen)

## Code sample

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
// Package spanner contains an example of using Spanner from a Cloud Function.
package spanner

import (
 "context"
 "fmt"
 "log"
 "net/http"
 "sync"

 "cloud.google.com/go/spanner"
 "github.com/GoogleCloudPlatform/functions-framework-go/functions"
 "google.golang.org/api/iterator"
)

// client is a global Spanner client, to avoid initializing a new client for
// every request.
var client *spanner.Client
var clientOnce sync.Once

// db is the name of the database to query.
var db = "projects/my-project/instances/my-instance/databases/example-db"

func init() {
 functions.HTTP("HelloSpanner", HelloSpanner)
}

// HelloSpanner is an example of querying Spanner from a Cloud Function.
func HelloSpanner(w http.ResponseWriter, r *http.Request) {
 clientOnce.Do(func() {
     // Declare a separate err variable to avoid shadowing client.
     var err error
     client, err = spanner.NewClient(context.Background(), db)
     if err != nil {
         http.Error(w, "Error initializing database", http.StatusInternalServerError)
         log.Printf("spanner.NewClient: %v", err)
         return
     }
 })

 fmt.Fprintln(w, "Albums:")
 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := client.Single().Query(r.Context(), stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return
     }
     if err != nil {
         http.Error(w, "Error querying database", http.StatusInternalServerError)
         log.Printf("iter.Next: %v", err)
         return
     }
     var singerID, albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         http.Error(w, "Error parsing database response", http.StatusInternalServerError)
         log.Printf("row.Columns: %v", err)
         return
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import com.google.api.client.http.HttpStatusCodes;
import com.google.cloud.functions.HttpFunction;
import com.google.cloud.functions.HttpRequest;
import com.google.cloud.functions.HttpResponse;
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.LazySpannerInitializer;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.SpannerException;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import com.google.common.annotations.VisibleForTesting;
import com.google.common.base.MoreObjects;
import java.io.PrintWriter;
import java.util.logging.Level;
import java.util.logging.Logger;

// HelloSpanner is an example of querying Spanner from a Cloud Function.
public class HelloSpanner implements HttpFunction {
  private static final Logger logger = Logger.getLogger(HelloSpanner.class.getName());

  // TODO<developer>: Set these environment variables.
  private static final String SPANNER_INSTANCE_ID =
      MoreObjects.firstNonNull(System.getenv("SPANNER_INSTANCE"), "my-instance");
  private static final String SPANNER_DATABASE_ID =
      MoreObjects.firstNonNull(System.getenv("SPANNER_DATABASE"), "example-db");

  private static final DatabaseId databaseId =
      DatabaseId.of(
          SpannerOptions.getDefaultProjectId(),
          SPANNER_INSTANCE_ID,
          SPANNER_DATABASE_ID);

  // The LazySpannerInitializer instance is shared across all instances of the HelloSpanner class.
  // It will create a Spanner instance the first time one is requested, and continue to return that
  // instance for all subsequent requests.
  private static final LazySpannerInitializer SPANNER_INITIALIZER = new LazySpannerInitializer();

  @VisibleForTesting
  DatabaseClient getClient() throws Throwable {
    return SPANNER_INITIALIZER.get().getDatabaseClient(databaseId);
  }

  @Override
  public void service(HttpRequest request, HttpResponse response) throws Exception {
    var writer = new PrintWriter(response.getWriter());
    try {
      DatabaseClient client = getClient();
      try (ResultSet rs =
          client
              .singleUse()
              .executeQuery(Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"))) {
        writer.printf("Albums:%n");
        while (rs.next()) {
          writer.printf(
              "%d %d %s%n",
              rs.getLong("SingerId"), rs.getLong("AlbumId"), rs.getString("AlbumTitle"));
        }
      } catch (SpannerException e) {
        writer.printf("Error querying database: %s%n", e.getMessage());
        response.setStatusCode(HttpStatusCodes.STATUS_CODE_SERVER_ERROR, e.getMessage());
      }
    } catch (Throwable t) {
      logger.log(Level.SEVERE, "Spanner example failed", t);
      writer.printf("Error setting up Spanner: %s%n", t.getMessage());
      response.setStatusCode(HttpStatusCodes.STATUS_CODE_SERVER_ERROR, t.getMessage());
    }
  }
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Imports the functions framework to register your HTTP function
const functions = require('@google-cloud/functions-framework');

// Instantiates a client
const spanner = new Spanner();

// Your Cloud Spanner instance ID
const instanceId = 'test-instance';

// Your Cloud Spanner database ID
const databaseId = 'example-db';

/**
 * HTTP Cloud Function.
 *
 * @param {Object} req Cloud Function request context.
 * @param {Object} res Cloud Function response context.
 */
functions.http('spannerQuickstart', async (req, res) => {
  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  // The query to execute
  const query = {
    sql: 'SELECT * FROM Albums',
  };

  // Execute the query
  try {
    const results = await database.run(query);
    const rows = results[0].map(row => row.toJSON());
    rows.forEach(row => {
      res.write(
        `SingerId: ${row.SingerId}, ` +
          `AlbumId: ${row.AlbumId}, ` +
          `AlbumTitle: ${row.AlbumTitle}\n`
      );
    });
    res.status(200).end();
  } catch (err) {
    res.status(500).send(`Error querying Spanner: ${err}`);
  }
});
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
import functions_framework
from google.cloud import spanner

instance_id = "test-instance"
database_id = "example-db"

client = spanner.Client()
instance = client.instance(instance_id)
database = instance.database(database_id)


@functions_framework.http
def spanner_read_data(request):
    query = "SELECT * FROM Albums"

    outputs = []
    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(query)

        for row in results:
            output = "SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row)
            outputs.append(output)

    return "\n".join(outputs)
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
