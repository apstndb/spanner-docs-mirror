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

This document shows you how to establish secure connections to Spanner Omni by configuring the Go client library. You establish these connections by setting client options when you create a database administrative client or a database client.

The Go client library supports plain text, TLS, and mTLS connections. For all connection types, include `option.WithoutAuthentication()` to prevent transmitting Google Cloud credentials to a Spanner Omni endpoint.

For more information, see [Getting started with Spanner in Go](https://docs.cloud.google.com/spanner/docs/getting-started/go) in the Spanner documentation.

## Plain-text communication

To establish plain-text communication, run the following code:

    import ("google.golang.org/grpc/credentials/insecure")
    
    adminClient, err := database.NewDatabaseAdminClient(ctx,
        option.WithEndpoint(OMNI_ENDPOINT),
    option.WithoutAuthentication(),
    option.WithGRPCDialOption(grpc.WithTransportCredentials(insecure.NewCredentials())),
    )
    
    clientConfig := ClientConfig{
            IsExperimentalHost: true,
        }
    databaseClient, err := spanner.NewClientWithConfig(ctx, db,
    clientConfig,
        option.WithEndpoint(OMNI_ENDPOINT),
    option.WithoutAuthentication(),
    option.WithGRPCDialOption(grpc.WithTransportCredentials(insecure.NewCredentials())),
    )

## TLS connection

To establish a TLS connection, run the following code:

    func createClients(ca_certificate, database, omniEndpoint string){
        // TLS CA cert configuration
    caCert, err := os.ReadFile(ca_certificate)
        capool := x509.NewCertPool()
    capool.AppendCertsFromPEM(caCert)
        creds := credentials.NewTLS(&tls.Config{RootCAs: capool})
    
        adminClient, err := database.NewDatabaseAdminClient(ctx,
            option.WithEndpoint(omniEndpoint),
    option.WithGRPCDialOption(grpc.WithTransportCredentials(creds)),
    option.WithoutAuthentication(),
    )
    clientConfig := ClientConfig{
            IsExperimentalHost: true,
        }
    databaseClient, err := spanner.NewClientWithConfig(ctx, db,
    clientConfig,
            option.WithEndpoint(omniEndpoint),
    option.WithoutAuthentication(),
    option.WithGRPCDialOption(grpc.WithTransportCredentials(creds)),
    )
    }

## mTLS connection

To establish an mTLS connection, run the following code:

    func createClients(ca_certificate, client_certificate, client_key, database, omniEndpoint string){
        // mTLS cred configuration
        caCert, err := os.ReadFile(ca_certificate)
            capool := x509.NewCertPool()
            capool.AppendCertsFromPEM(caCert)
            cert := tls.LoadX509KeyPair(client_certificate, client_key)
            creds := credentials.NewTLS(&tls.Config{Certificates: []tls.Certificate{cert}, RootCAs: capool})
    
            adminClient, err := database.NewDatabaseAdminClient(ctx,
                option.WithEndpoint(omniEndpoint),
                option.WithoutAuthentication(),
        option.WithGRPCDialOption(grpc.WithTransportCredentials(creds))
        )
        clientConfig := ClientConfig{
                IsExperimentalHost: true,
            }
        databaseClient, err := spanner.NewClientWithConfig(ctx, db,
        clientConfig,
                option.WithEndpoint(omniEndpoint),
        option.WithoutAuthentication(),
        option.WithGRPCDialOption(grpc.WithTransportCredentials(creds)),
        )
    }
