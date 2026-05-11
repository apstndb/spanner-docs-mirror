---
name: documents/docs.cloud.google.com/spanner-omni/go-sql
uri: https://docs.cloud.google.com/spanner-omni/go-sql
title: Use the Go SQL package to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

The [Go SQL package](https://pkg.go.dev/database/sql) is a generic interface around SQL (or SQL-like) databases for the Go programming language. The Spanner driver for the Go SQL package lets you use this standard interface to interact with your Spanner databases.

This document describes how to configure the Go SQL package driver to connect to Spanner Omni. The driver works with Spanner Omni in the same way it works with Spanner, supporting both GoogleSQL and PostgreSQL database dialects.

## Prerequisites

To use the Go SQL package to connect to Spanner Omni, ensure you meet the following requirements:

  - Use [version 1.23.0](https://pkg.go.dev/database/sql@go1.23.0) or later of the Go SQL package.

  - Add the following module to your `go.mod` file to use the driver in your application:
    
        go get github.com/googleapis/go-sql-spanner

## Establish a connection

Run one of the following commands to establish a connection with Spanner Omni:

### Plain-text communication

To establish a plain-text communication channel, run the following command:

    db, err := sql.Open("spanner", "ENDPOINT/databases/DATABASE_ID?use_plain_text=true;is_experimental_host=true")

### TLS connection

To establish a secure TLS connection, run the following command:

    db, err := sql.Open("spanner", "ENDPOINT/databases/DATABASE_ID?ca_cert_file=PATH_TO_CA_CRT;is_experimental_host=true")

### mTLS connection

To establish a mutual TLS (mTLS) connection, run the following command:

    db, err := sql.Open("spanner", "ENDPOINT/databases/DATABASE_ID?ca_cert_file=PATH_TO_CA_CRT;client_cert_file=PATH_TO_CLIENT_CRT;client_cert_key=PATH_TO_CLIENT_KEY;is_experimental_host=true")
