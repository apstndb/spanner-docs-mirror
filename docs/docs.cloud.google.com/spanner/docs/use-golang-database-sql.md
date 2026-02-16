[Go database/sql](https://pkg.go.dev/database/sql) is a generic interface around SQL (or SQL-like) databases for the Go programming language. To use database/sql with your application, use the [Spanner database/sql driver](https://github.com/googleapis/go-sql-spanner) .

The Spannerdatabase/sql driver supports both GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Install the Spanner database/sql driver

To use the Spanner database/sql driver in your application, add the following module to your `  go.mod  ` file:

``` text
  github.com/googleapis/go-sql-spanner
```

## Use the Spanner database/sql driver

To create a database/sql connection to a Spanner database, use `  spanner  ` as the driver name and a fully qualified database name as the connection string:

### GoogleSQL

``` go
import (
 "context"
 "database/sql"
 "fmt"

 _ "github.com/googleapis/go-sql-spanner"
)

func connect(projectId, instanceId, databaseId string) error {
 ctx := context.Background()
 dsn := fmt.Sprintf("projects/%s/instances/%s/databases/%s",
     projectId, instanceId, databaseId)
 db, err := sql.Open("spanner", dsn)
 if err != nil {
     return fmt.Errorf("failed to open database connection: %v", err)
 }
 defer func() { _ = db.Close() }()

 fmt.Printf("Connected to %s\n", dsn)
 row := db.QueryRowContext(ctx, "select cast(@greeting as string)", "Hello from Spanner")
 var greeting string
 if err := row.Scan(&greeting); err != nil {
     return fmt.Errorf("failed to get greeting: %v", err)
 }
 fmt.Printf("Greeting: %s\n", greeting)

 return nil
}
```

### PostgreSQL

``` go
import (
 "context"
 "database/sql"
 "fmt"

 _ "github.com/googleapis/go-sql-spanner"
)

func connect(projectId, instanceId, databaseId string) error {
 ctx := context.Background()
 dsn := fmt.Sprintf("projects/%s/instances/%s/databases/%s",
     projectId, instanceId, databaseId)
 db, err := sql.Open("spanner", dsn)
 if err != nil {
     return fmt.Errorf("failed to open database connection: %v", err)
 }
 defer func() { _ = db.Close() }()

 fmt.Printf("Connected to %s\n", dsn)
 // The Spanner database/sql driver supports both PostgreSQL-style query
 // parameters ($1, $2, ...) and positional query parameters (?, ?, ...).
 // This example uses PostgreSQL-style parameters.
 row := db.QueryRowContext(ctx, "select $1", "Hello from Spanner PostgreSQL")
 var greeting string
 if err := row.Scan(&greeting); err != nil {
     return fmt.Errorf("failed to get greeting: %v", err)
 }
 fmt.Printf("Greeting: %s\n", greeting)

 return nil
}
```

For more information, see the [Spanner database/sql driver GitHub repository](https://github.com/googleapis/go-sql-spanner) .

## Supported features

The [Spanner Go database/sql examples code directory](https://github.com/googleapis/go-sql-spanner/blob/-/examples) contains ready-to-run examples for commonly used Spanner features.

## Performance tips

To get the best possible performance when using the Spanner database/sql driver, follow these best practices:

  - Query parameters: [Use query parameters](https://github.com/googleapis/go-sql-spanner/blob/-/examples/query-parameters/main.go) instead of inline values in SQL statements. This lets Spanner cache and reuse the execution plan for frequently used SQL statements.
  - Database Definition Language (DDL): [Group multiple DDL statements into one batch](https://github.com/googleapis/go-sql-spanner/blob/-/examples/ddl-batches/main.go) instead of executing them one by one.
  - Data Manipulation Language (DML): [Group multiple DML statements into one batch](https://github.com/googleapis/go-sql-spanner/blob/-/examples/dml-batches/main.go) instead of executing them one by one.
  - Read-only transactions: [Use read-only transactions](https://github.com/googleapis/go-sql-spanner/blob/-/examples/read-only-transactions/main.go) for workloads that only read data. Read-only transactions don't take locks.
  - Tags: [Use request and transaction tags](https://github.com/googleapis/go-sql-spanner/blob/-/examples/tags/main.go) to [troubleshoot](/spanner/docs/introspection/troubleshooting-with-tags) .

## What's next

  - Learn more about using Spanner with the database/sql driver [code examples](https://github.com/googleapis/go-sql-spanner/blob/-/examples) .
  - Learn more about [database/sql](https://pkg.go.dev/database/sql) .
  - Use [GORM with Spanner](/spanner/docs/use-gorm) .
  - [File a GitHub issue](https://github.com/googleapis/go-sql-spanner/issues) to report a feature request or bug, or to ask a question about the Spanner database/sql driver.
