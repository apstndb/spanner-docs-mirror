---
name: documents/docs.cloud.google.com/spanner-omni/gorm
uri: https://docs.cloud.google.com/spanner-omni/gorm
title: Use GORM to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
update_time: "2026-05-08T21:32:37Z"
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

[GORM](https://gorm.io/) is an object-relational mapping (ORM) tool for the Go programming language. It provides a framework for mapping an object-oriented domain model to a relational database. You can integrate GoogleSQL-dialect databases with GORM using the open source [Spanner Dialect](https://github.com/googleapis/go-gorm-spanner) ( `SpannerDialect` ).

This document describes how to configure GORM to connect to Spanner Omni. GORM integrates with Spanner Omni in the same way it integrates with Spanner. Note that GORM supports integration with Spanner Omni using the GoogleSQL dialect, but not the PostgreSQL dialect.

For more information, see [Integrate Spanner with GORM](https://docs.cloud.google.com/spanner/docs/use-gorm) in the Spanner documentation.

## Prerequisites

Spanner Omni requires [`go-gorm-spanner`](https://github.com/googleapis/go-gorm-spanner) version 1.10.0 or later. To use the Spanner GORM dialect in your application, add the following modules to your `go.mod` file:

    go get github.com/googleapis/go-sql-spanner
    go get github.com/googleapis/go-gorm-spanner

## Configure GORM for GoogleSQL databases

To use the GoogleSQL GORM dialect in your application, add the following import statements to the file where GORM is initialized:

    import (
        "fmt"
    
        "gorm.io/gorm"
        _ "github.com/googleapis/go-sql-spanner"
        spannergorm "github.com/googleapis/go-gorm-spanner"
    )
    
    dsn := fmt.Sprintf("%s/databases/%s?use_plain_text=true", SPANNER_OMNI_ENDPOINT, DATABASE_ID)
    db, err := gorm.Open(spannergorm.New(spannergorm.Config{DriverName: "spanner", DSN: dsn}), &gorm.Config{})
