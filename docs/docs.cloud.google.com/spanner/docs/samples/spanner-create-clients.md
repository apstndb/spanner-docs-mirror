Create a client that can read, write, and run transactions.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Getting started with Spanner in Go](/spanner/docs/getting-started/go)

## Code sample

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
 database "cloud.google.com/go/spanner/admin/database/apiv1"
)

func createClients(w io.Writer, db string) error {
 ctx := context.Background()

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 dataClient, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer dataClient.Close()

 _ = adminClient
 _ = dataClient

 return nil
}
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
