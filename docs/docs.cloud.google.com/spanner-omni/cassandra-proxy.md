---
name: documents/docs.cloud.google.com/spanner-omni/cassandra-proxy
uri: https://docs.cloud.google.com/spanner-omni/cassandra-proxy
title: Use the Cassandra proxy to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Connect the Apache Cassandra proxy to Spanner Omni to let your existing Cassandra applications interact with Spanner Omni using the Cassandra Query Language (CQL). This integration lets you use Spanner Omni capabilities while maintaining compatibility with your Cassandra client applications.

To connect the proxy, follow these high-level steps:

1.  Clone the [Cassandra-to-Spanner proxy repository](https://github.com/cloudspannerecosystem/cassandra-to-spanner-proxy) .

2.  Convert your Cassandra schema definitions to the Spanner Omni schema using the provided schema converter script, which supports plain-text, TLS, and mTLS security modes.

3.  Configure the proxy adapter by updating its configuration file with the Spanner Omni endpoint and security settings.

4.  Build and run the proxy. Then, connect to it using `cqlsh` to begin operations.

For more information, see [Cassandra interface](https://docs.cloud.google.com/spanner/docs/non-relational/cassandra-overview) in the Spanner documentation.

## Before you begin

Before you start, complete the following requirements:

  - Configure a Spanner Omni deployment and create a database.

  - Enable multiplexed sessions within your environment by setting the required [environment variables](https://docs.cloud.google.com/spanner-omni/cassandra-proxy#set-environment-variables) .

  - Ensure that Go is installed on your local machine.

  - Read the [limitations of the Cassandra-to-Spanner proxy](https://docs.cloud.google.com/spanner/docs/non-relational/spanner-for-cassandra-users#limitations) for usage considerations.

  - Establish the security mode (plain-text, TLS, or mTLS) that you want to use for communication between the proxy and Spanner Omni.

## Set environment variables

To use the Cassandra proxy, you must enable multiplexed sessions by setting the required environment variables. Multiplexed sessions are required for Spanner Omni connections, but are disabled by default in the Spanner client libraries and drivers.

Set the following environment variables:

    GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS_FOR_RW=true
    GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS_PARTITIONED_OPS=true
    GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS=true

## Clone the repository

Clone the repository supporting the proxy and ensure that Go is installed on your machine:

    git clone https://github.com/cloudspannerecosystem/cassandra-to-spanner-proxy.git
    
    // Ensure all Go modules are installed
    go mod tidy

## Convert the Cassandra schema

Create all Cassandra tables in advance. The `schema_converter/cql_to_spanner_schema_converter.go` script converts Cassandra `CREATE TABLE` queries from a CQL file into Spanner `CREATE TABLE` queries.

  - ` --database DATABASE_ID  ` : Specify the target database name in Spanner Omni. Replace `DATABASE_ID` with your database name.

  - ` --cql PATH_TO_CQL_FILE  ` : Specify the path to the CQL file that contains the Cassandra schema definition.

  - ` --endpoint ENDPOINT  ` : Specify the Spanner Omni endpoint address. Replace `ENDPOINT` with your Spanner Omni endpoint.
    
        go run schema_converter/cql_to_spanner_schema_converter.go --database DATABASE_ID --cql PATH_TO_CQL_FILE --endpoint ENDPOINT
    
    The schema converter supports all three Spanner Omni security modes: plain-text, TLS, and mTLS. Each security mode requires additional parameters:

  - For the plain-text mode, use the `--usePlainText` flag:
    
        go run schema_converter/cql_to_spanner_schema_converter.go --database DATABASE_ID --cql PATH_TO_CQL_FILE --endpoint ENDPOINT --usePlainText

  - For the TLS mode, use the `--caCertificate` flag with the path to the CA certificate file:
    
        go run schema_converter/cql_to_spanner_schema_converter.go --database DATABASE_ID --cql PATH_TO_CQL_FILE --endpoint ENDPOINT --caCertificate PATH_TO_CA_CRT

  - For the mTLS mode, use the `--caCertificate` , `--clientCertificate` , and `--clientKey` flags with the corresponding paths:
    
        go run schema_converter/cql_to_spanner_schema_converter.go --database DATABASE_ID --cql PATH_TO_CQL_FILE --endpoint ENDPOINT --caCertificate PATH_TO_CA_CRT --clientCertificate PATH_TO_CLIENT_CERT --clientKey PATH_TO_CLIENT_KEY
    
    The script also creates a `TableConfigurations` table if it isn't already present. This table tracks the schema metadata of your Cassandra tables and columns:
    
        CREATE TABLE IF NOT EXISTS TableConfigurations (
            `KeySpaceName` STRING(MAX),
            `TableName` STRING(MAX),
            `ColumnName` STRING(MAX),
            `ColumnType` STRING(MAX),
            `IsPrimaryKey` BOOL,
            `PK_Precedence` INT64,
        ) PRIMARY KEY (TableName, ColumnName, KeySpaceName);

## Configure the proxy adapter

After you create Cassandra tables using the schema converter script, configure the adapter to perform operations on your tables. To set up the adapter, do the following:

Update the `config.yaml` file in the root directory of the repository with the available configuration options. The adapter configuration file requires that you define the Spanner Omni endpoint, along with the relevant security mode options.

    # [Optional] endpoint configuration for spanner
    endpoint: ENDPOINT
    
    # [Optional] If set to True, will connect to endpoint over plain text
    usePlainText: False
    
    # [Optional] CA certificate path for TLS and mTLS configuration
    caCertificate: PATH_TO_CA_CRT
    
    # [Optional] client certificate path for mTLS configuration
    clientCertificate: PATH_TO_CLIENT_CERT
    
    # [Optional] client key path for mTLS configuration
    clientKey: PATH_TO_CLIENT_KEY

The following configuration setup provides the minimum configuration to run the adapter:

    cassandra_to_spanner_configs:
      # [Optional] endpoint configuration for spanner
      endpoint: ENDPOINT
    
      # Uncomment the options as required by the security mode of the Spanner Omni deployment
      # [Optional] If set to True, will connect to endpoint over plain text
      # usePlainText: False
      # [Optional] CA certificate path for TLS and mTLS configuration
      # caCertificate: /tmp/ca.crt
      # [Optional] client certificate path for mTLS configuration
      # clientCertificate: /tmp/client.crt
      # [Optional] client key path for mTLS configuration
      # clientKey: /tmp/client.key
    
    listeners:
      - name: CLUSTER_NAME
        port: 9042
        spanner:
          databaseId: DATABASE_ID

## Build and run the proxy

After you configure the proxy adapter, build and run the proxy using the following commands:

    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o cassandra-to-spanner-proxy .
    ./cassandra-to-spanner-proxy

Connect to the proxy using the shell with the following command:

    ./cqlsh localhost 9042
