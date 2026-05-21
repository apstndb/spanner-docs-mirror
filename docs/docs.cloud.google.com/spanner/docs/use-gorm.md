---
name: documents/docs.cloud.google.com/spanner/docs/use-gorm
uri: https://docs.cloud.google.com/spanner/docs/use-gorm
title: Integrate Spanner with GORM (GoogleSQL dialect)
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

GORM is an object-relational mapping tool for the Go programming language. It provides a framework for mapping an object-oriented domain model to a relational database.

You can integrate GoogleSQL-dialect databases with GORM using the open source [Spanner Dialect](https://github.com/googleapis/go-gorm-spanner) ( `SpannerDialect` ).

## Set up GORM with Spanner GoogleSQL-dialect databases

To use the GoogleSQL GORM dialect in your application, add the following code to the file where GORM is initialized:

    import (
      "fmt"
    
      "gorm.io/gorm"
      _ "github.com/googleapis/go-sql-spanner"
      spannergorm "github.com/googleapis/go-gorm-spanner"
    )
    
    dsn := fmt.Sprintf("projects/%s/instances/%s/databases/%s", projectId, instanceId, databaseId)
    db, err := gorm.Open(spannergorm.New(spannergorm.Config{DriverName: "spanner", DSN: dsn}), &gorm.Config{})

For more connection options for Spanner, see the [GORM with GoogleSQL documentation](https://github.com/googleapis/go-sql-spanner/blob/-/driver.go) .

## Use GORM with Spanner GoogleSQL-dialect databases

For more information about the features and recommendations for using GORM with Spanner, consult the [reference documentation](https://github.com/googleapis/go-gorm-spanner) on GitHub.

## What's next

  - Checkout the [sample application](https://github.com/googleapis/go-gorm-spanner/blob/-/samples/helloworld/main.go) using GORM with GoogleSQL and Spanner.
  - Learn more about [GORM](https://gorm.io/) .
  - [File a GitHub issue](https://github.com/googleapis/go-gorm-spanner/issues) to report a bug or ask a question about using GORM with Spanner with GoogleSQL.
  - Learn more about [Integrate Spanner with GORM ( PostgreSQL dialect)](https://docs.cloud.google.com/spanner/docs/use-gorm-postgresql) .
