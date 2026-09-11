---
name: documents/docs.cloud.google.com/spanner-omni/go
uri: https://docs.cloud.google.com/spanner-omni/go
title: Use the Go client library to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Spanner Omni and Spanner use the Go client library similarly. This document shows you how to establish secure connections to Spanner Omni by configuring the Go client library. You establish these connections by configuring [`spanner.ClientConfig`](https://docs.cloud.google.com/go/docs/reference/cloud.google.com/go/spanner/latest#cloud_google_com_go_spanner_ClientConfig) when you create a database administrative client or a database client.

The Go client library supports plain text, TLS, TLS with credentials, and mTLS connections.

For more information, see [Get started with Spanner in Go](https://docs.cloud.google.com/spanner/docs/getting-started/go) in the Spanner documentation.

## Before you begin

To use the Go client library with Spanner Omni, use the Go client library [version v1.94.0](https://docs.cloud.google.com/go/docs/reference/cloud.google.com/go/spanner/1.94.0) or later and [Go release version 1.25](https://go.dev/doc/go1.25) or later.

To add the Spanner Go module to your `go.mod` file, run the following command:

    go get cloud.google.com/go/spanner@v1.94.0

## Configure the `ClientConfig` object

To use the Go client library to create a [`Client`](https://docs.cloud.google.com/go/docs/reference/cloud.google.com/go/spanner/latest#cloud_google_com_go_spanner_Client) or a [`DatabaseAdminClient`](https://docs.cloud.google.com/go/docs/reference/cloud.google.com/go/spanner/latest/admin/database/apiv1#cloud_google_com_go_spanner_admin_database_apiv1_DatabaseAdminClient) , configure the [`ClientConfig`](https://docs.cloud.google.com/go/docs/reference/cloud.google.com/go/spanner/latest#cloud_google_com_go_spanner_ClientConfig) object by specifying `Type: spanner.OMNI` and supply the endpoint using `option.WithEndpoint()` .

The following examples show how to configure the `ClientConfig` object for each supported security configuration:

### Plain text

To establish a plain-text connection, set `UsePlainText` to `true` in `spanner.ClientConfig` :

    clientConfig := spanner.ClientConfig{
      Type:         spanner.OMNI,
      UsePlainText: true,
    }
    
    adminClient, err := database.NewDatabaseAdminClientWithConfig(ctx, clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer adminClient.Close()
    
    databaseClient, err := spanner.NewClientWithConfig(ctx, "DATABASE_NAME", clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer databaseClient.Close()

### TLS

To establish a TLS connection, specify the path to your CA certificate using `CaCertificateFile` :

    clientConfig := spanner.ClientConfig{
      Type:              spanner.OMNI,
      CaCertificateFile: "PATH_TO_CA_CERT",
    }
    
    adminClient, err := database.NewDatabaseAdminClientWithConfig(ctx, clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer adminClient.Close()
    
    databaseClient, err := spanner.NewClientWithConfig(ctx, "DATABASE_NAME", clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer databaseClient.Close()

### TLS with credentials

To establish a TLS connection with username and password authentication, specify `CaCertificateFile` , `Username` , and `Password` :

    clientConfig := spanner.ClientConfig{
      Type:              spanner.OMNI,
      CaCertificateFile: "PATH_TO_CA_CERT",
      Username:          "USERNAME",
      Password:          []byte("PASSWORD"),
    }
    
    adminClient, err := database.NewDatabaseAdminClientWithConfig(ctx, clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer adminClient.Close()
    
    databaseClient, err := spanner.NewClientWithConfig(ctx, "DATABASE_NAME", clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer databaseClient.Close()

### mTLS

To establish a mutual TLS (mTLS) connection, specify `CaCertificateFile` , `ClientCertificateFile` , and `ClientKeyFile` :

    clientConfig := spanner.ClientConfig{
      Type:                  spanner.OMNI,
      CaCertificateFile:     "PATH_TO_CA_CERT",
      ClientCertificateFile: "PATH_TO_CLIENT_CERT",
      ClientKeyFile:         "PATH_TO_CLIENT_KEY",
    }
    
    adminClient, err := database.NewDatabaseAdminClientWithConfig(ctx, clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer adminClient.Close()
    
    databaseClient, err := spanner.NewClientWithConfig(ctx, "DATABASE_NAME", clientConfig,
      option.WithEndpoint("ENDPOINT"),
    )
    if err != nil {
      // Handle error.
    }
    defer databaseClient.Close()

Replace the following:

  - `  PATH_TO_CA_CERT  ` : the path to your CA certificate file.

  - `  PATH_TO_CLIENT_CERT  ` : the path to your client certificate file.

  - `  PATH_TO_CLIENT_KEY  ` : the path to your client key file.
