---
name: documents/docs.cloud.google.com/spanner-omni/go-cassandra
uri: https://docs.cloud.google.com/spanner-omni/go-cassandra
title: Use the Cassandra Go client to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

The Cassandra Go client for Spanner connects applications written for the Apache Cassandra database with Spanner. The client works with Spanner Omni in the same way it works with Spanner. Because Spanner natively supports the Cassandra v4 wire protocol, this client lets Go applications that use the `gocql` driver, or non-Go applications and tools such as `cqlsh` , connect to a Spanner database.

This client acts as a local TCP proxy. It intercepts the raw Cassandra protocol bytes that a driver or client tool sends. It then wraps these bytes and necessary metadata into gRPC messages to communicate with Spanner Omni. The client translates responses from Spanner Omni back into the Cassandra wire format and sends them back to the originating driver or tool.

This document shows how to integrate the client with Spanner Omni using one of the following methods:

  - **[In-process dependency](https://docs.cloud.google.com/spanner-omni/go-cassandra#in-process)** : Use this method for Go applications already using the `gocql` driver. This approach embeds the client within your application process for minimal code modifications.

  - **[Sidecar proxy](https://docs.cloud.google.com/spanner-omni/go-cassandra#sidecar-proxy)** : Use this method for non-Go applications or when using external Cassandra tools, such as `cqlsh` . This approach runs the client as a standalone process.

For more information about how Apache Cassandra works with Spanner, see [Cassandra interface](https://docs.cloud.google.com/spanner/docs/non-relational/cassandra-overview) .

## When to use the Spanner Cassandra Go Client

This client is useful in the following scenarios:

  - **Use Spanner with minimal refactoring.** You want to use Spanner as the backend for your Go application but prefer to keep using the familiar `gocql` API for data access.

  - **Use non-Go Cassandra tools.** You want to connect to Spanner using standard Cassandra tools like `cqlsh` or applications written in other languages that use Cassandra drivers.

## Use the client as an in-process dependency

Go applications connect to Spanner Omni by integrating the Spanner Cassandra Go client as an in-process dependency. This approach embeds the proxy logic directly within your application, which simplifies your deployment architecture by removing the need for a separate process. This configuration also provides optimal performance by avoiding an extra network hop and extra serialization and deserialization of the data.

To use the client as an in-process dependency, do the following:

  - Import the Spanner package in your Go application:
    
        import spanner "github.com/googleapis/go-spanner-cassandra/cassandra/gocql"

  - Modify your cluster creation code. Instead of using `gocql.NewCluster` , use `spanner.NewCluster` and provide the following Spanner Omni specific options:
    
    ### Plain-text communication
    
    The following example shows how to establish a plain-text connection to Spanner Omni:
    
        func main() {
          opts := &spanner.Options{
              // Required: Specify the Spanner database URI
              DatabaseUri: "DATABASE_ID",
          }
          // Optional: Configure Spanner Omni cluster settings as needed
          opts.ExperimentalHost = true
          opts.UsePlainText = true
        
          cluster := spanner.NewCluster(opts)
          // ...
        }
    
    ### TLS connection
    
    The following example shows how to establish a TLS connection to Spanner Omni:
    
        func main() {
          opts := &spanner.Options{
              // Required: Specify the Spanner database URI
              DatabaseUri: "DATABASE_ID",
          }
          // Optional: Configure Spanner Omni cluster settings as needed
          opts.ExperimentalHost = true
          opts.CaCertificate = "PATH_TO_CA_CRT"
        
          cluster := spanner.NewCluster(opts)
          // ...
        }
    
    ### mTLS connection
    
    The following example shows how to establish an mTLS connection to Spanner Omni:
    
        func main() {
          opts := &spanner.Options{
              // Required: Specify the Spanner database URI
              DatabaseUri: "DATABASE_ID",
          }
          // Optional: Configure Spanner Omni cluster settings as needed
          opts.ExperimentalHost = true
          opts.CaCertificate = "PATH_TO_CA_CRT"
          opts.ClientCertificate = "PATH_TO_CLIENT_CERT"
          opts.ClientKey = "PATH_TO_CLIENT_KEY"
        
          cluster := spanner.NewCluster(opts)
          // ...
        }

## Deploy the client as a sidecar proxy

Deploying the Spanner Cassandra Go client as a sidecar proxy is an effective option for non-Go applications and tools, such as `cqlsh` , to connect to Spanner Omni using standard Cassandra drivers. This method runs the client as a standalone TCP proxy that intercepts Cassandra wire protocol traffic and translates it into gRPC for communication with Spanner Omni.

This configuration is useful when you need to use external Cassandra tools or when you need to avoid making direct modifications to your application's code.

You can run the sidecar proxy in the following ways:

  - [Run locally with the Go `run` command](https://docs.cloud.google.com/spanner-omni/go-cassandra#go-run-locally)

  - [Run with a pre-built Docker image](https://docs.cloud.google.com/spanner-omni/go-cassandra#docker-run)

### Run locally with the Go `run` command

Running the sidecar proxy as a local process from source code is useful for development and testing environments where you want to quickly iterate on your application and proxy configuration.

1.  Clone the repository:
    
    `git clone https://github.com/googleapis/go-spanner-cassandra.git`

2.  Change to the repository directory:
    
    `cd go-spanner-cassandra`

3.  Run `cassandra_launcher.go` with the required `-db` flag and the following Spanner Omni specific flags. Replace the value of `-db` with your Spanner Omni database name:

<!-- end list -->

  - For plain-text communication, run the following:

<!-- end list -->

    go run cassandra_launcher.go -db DATABASE_ID -endpoint ENDPOINT -experimentalHost -usePlainText

  - For a TLS connection, run the following:

<!-- end list -->

    go run cassandra_launcher.go -db DATABASE_ID -endpoint ENDPOINT -experimentalHost -caCertificate PATH_TO_CA_CRT

  - For an mTLS connection, run the following:

<!-- end list -->

    go run cassandra_launcher.go -db DATABASE_ID -endpoint ENDPOINT -experimentalHost -caCertificate PATH_TO_CA_CRT -clientCertificate PATH_TO_CLIENT_CERT -clientKey PATH_TO_CLIENT_KEY

### Run with a prebuilt Docker image

We recommend running the sidecar proxy as a containerized application using a prebuilt Docker image for production environments because it provides a consistent and isolated runtime environment.

1.  Pull the image from the official registry repository:
    
    `docker pull gcr.io/cloud-spanner-adapter/cassandra-adapter`

2.  Run the image with the required flags:
    
    ### Plain-text communication
    
    For plain-text communication, run the following command:
    
        docker run -d -p 9042:9042 gcr.io/cloud-spanner-adapter/cassandra-adapter -db DATABASE_ID -endpoint ENDPOINT -experimentalHost -usePlainText
    
    ### TLS connection
    
    For a TLS connection, run the following command:
    
        docker run -d -p 9042:9042 gcr.io/cloud-spanner-adapter/cassandra-adapter -db DATABASE_ID -endpoint ENDPOINT -experimentalHost -caCertificate PATH_TO_CA_CRT
    
    ### mTLS connection
    
    For an mTLS connection, run the following command:
    
        docker run -d -p 9042:9042 gcr.io/cloud-spanner-adapter/cassandra-adapter -db DATABASE_ID -endpoint ENDPOINT -experimentalHost -caCertificate PATH_TO_CA_CRT -clientCertificate PATH_TO_CLIENT_CERT -clientKey PATH_TO_CLIENT_KEY
