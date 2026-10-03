---
name: documents/docs.cloud.google.com/spanner-omni/gorm
uri: https://docs.cloud.google.com/spanner-omni/gorm
title: Use GORM to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

[GORM](https://gorm.io/) is an object-relational mapping (ORM) tool for the Go programming language. It provides a framework for mapping an object-oriented domain model to a relational database. You can integrate GoogleSQL-dialect databases with GORM using the open source [Spanner Dialect](https://github.com/googleapis/go-gorm-spanner) ( `SpannerDialect` ).

This document describes how to configure GORM to connect to Spanner Omni. GORM integrates with Spanner Omni in the same way it integrates with Spanner. Note that GORM supports integration with Spanner Omni using the GoogleSQL dialect, but not the PostgreSQL dialect.

For more information, see [Integrate Spanner with GORM](https://docs.cloud.google.com/spanner/docs/use-gorm) in the Spanner documentation.

## Prerequisites

Spanner Omni requires [`go-gorm-spanner`](https://github.com/googleapis/go-gorm-spanner) version 1.10.0 or later. To use the Spanner GORM dialect in your application, add the following modules to your `go.mod` file:

```
go get github.com/googleapis/go-sql-spanner
go get github.com/googleapis/go-gorm-spanner
```

## Configure GORM for GoogleSQL databases

To use the GoogleSQL GORM dialect in your application, add the following import statements to the file where GORM is initialized:

```
import (
    "fmt"

    "gorm.io/gorm"
    _ "github.com/googleapis/go-sql-spanner"
    spannergorm "github.com/googleapis/go-gorm-spanner"
)

dsn := fmt.Sprintf("%s/databases/%s?use_plain_text=true", SPANNER_OMNI_ENDPOINT, DATABASE_ID)
db, err := gorm.Open(spannergorm.New(spannergorm.Config{DriverName: "spanner", DSN: dsn}), &gorm.Config{})
```
