---
name: documents/docs.cloud.google.com/spanner-omni/java-cassandra
uri: https://docs.cloud.google.com/spanner-omni/java-cassandra
title: Use the Cassandra Java client to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

The [Cassandra Java client for Spanner](https://github.com/googleapis/java-spanner-cassandra?tab=readme-ov-file#spanner-cassandra-Java-client) connects applications written for the Apache Cassandra database with Spanner. The client works with Spanner Omni in the same way it works with Spanner.

Because Spanner natively supports the Cassandra v4 wire protocol, the Spanner Cassandra Java client translates the Cassandra wire protocol to the Spanner gRPC API. This allows you to migrate your Cassandra applications to Spanner with minimal code changes.

This document shows how to integrate the client with Spanner Omni using one of the following methods:

  - **[In-process dependency](https://docs.cloud.google.com/spanner-omni/java-cassandra#in-process)** : Use this method for Java applications already using the `cassandra-java-driver` . This approach embeds the client within your application process for minimal code modifications.

  - **[Sidecar proxy](https://docs.cloud.google.com/spanner-omni/java-cassandra#sidecar-proxy)** : Use this method for non-Java applications or when using external Cassandra tools, such as `cqlsh` . This approach runs the client as a standalone process.

For more information, see [Cassandra interface](https://docs.cloud.google.com/spanner/docs/non-relational/cassandra-overview) in the Spanner documentation.

## When to use the Spanner Cassandra Java client

This client is useful in the following scenarios:

  - **Use Spanner with minimal refactoring.** You want to use Spanner as the backend for your Java application but prefer to keep using the familiar `cassandra-java-driver` API for data access.

  - **Use non-Java Cassandra tools.** You want to connect to Spanner using standard Cassandra tools like `cqlsh` or applications written in other languages that use Cassandra drivers.

## Use the client as an in-process dependency

Integrating the Spanner Cassandra Java client as an in-process dependency is the **recommended** method for Java applications to connect to Spanner Omni. Compared to the sidecar proxy method, the in-process configuration provides better performance by avoiding an extra network hop and the associated serialization and deserialization of data. It also simplifies your deployment architecture by removing the need to manage a separate standalone process.

To use the client as an in-process dependency, do the following:

1.  Add the Spanner Cassandra Java client as a dependency to your project:
    
        <dependency>
          <groupId>com.google.cloud</groupId>
          <artifactId>google-cloud-spanner-cassandra</artifactId>
          <version>1.2.0</version>
        </dependency>

2.  Modify your `CqlSession` creation code and add the Spanner Omni communication-specific options:
    
    ### Plain text
    
    The following example shows how to establish a plain-text connection to Spanner Omni:
    
        CqlSession session =
            SpannerCqlSession.builder() // `SpannerCqlSession` instead of `CqlSession`
                .setDatabaseUri("DATABASE_ID") // Required: Specify the Spanner database name
                .withConfigLoader(
                    DriverConfigLoader.programmaticBuilder()
                        .withString(DefaultDriverOption.PROTOCOL_VERSION, "V4")
                        .withDuration(
                            DefaultDriverOption.CONNECTION_INIT_QUERY_TIMEOUT,
                            Duration.ofSeconds(5))
                        .build())
                .setHost("ENDPOINT")
                .setInstanceType(SpannerCqlSessionBuilder.InstanceType.OMNI)
                .setUsePlainText(true)
                .build();
        
        // Rest of your business logic such as session.Query(SELECT * FROM ...)
        
        session.close();
    
    ### TLS
    
    To use a TLS connection, ensure the CA certificate is added to the truststore used by the application, as described in [Configure the Java truststore](https://docs.cloud.google.com/spanner-omni/java#configure-truststore) :
    
        CqlSession session =
            SpannerCqlSession.builder() // `SpannerCqlSession` instead of `CqlSession`
                .setDatabaseUri("DATABASE_ID") // Required: Specify the Spanner database name
                .withConfigLoader(
                    DriverConfigLoader.programmaticBuilder()
                        .withString(DefaultDriverOption.PROTOCOL_VERSION, "V4")
                        .withDuration(
                            DefaultDriverOption.CONNECTION_INIT_QUERY_TIMEOUT,
                            Duration.ofSeconds(5))
                        .build())
                .setHost("ENDPOINT")
                .setInstanceType(SpannerCqlSessionBuilder.InstanceType.OMNI)
                .setUsePlainText(false)
                .build();
        
        // Rest of your business logic such as session.Query(SELECT * FROM ...)
        
        session.close();
    
    ### mTLS
    
    To use an mTLS connection, ensure the CA certificate is added to the truststore used by the application and the client key is in PKCS\#8 format. For more information, see the [Java SDK mTLS instructions](https://docs.cloud.google.com/spanner-omni/java#mtls) .
    
        CqlSession session =
            SpannerCqlSession.builder() // `SpannerCqlSession` instead of `CqlSession`
                .setDatabaseUri("DATABASE_ID") // Required: Specify the Spanner database name
                .withConfigLoader(
                    DriverConfigLoader.programmaticBuilder()
                        .withString(DefaultDriverOption.PROTOCOL_VERSION, "V4")
                        .withDuration(
                            DefaultDriverOption.CONNECTION_INIT_QUERY_TIMEOUT,
                            Duration.ofSeconds(5))
                        .build())
                .setHost("ENDPOINT")
                .setInstanceType(SpannerCqlSessionBuilder.InstanceType.OMNI)
                .setUsePlainText(false)
                .setClientCertPathAndKey(
                    "PATH_TO_CLIENT_CERT",
                    "PATH_TO_CLIENT_KEY_PKCS8")
                .build();
        
        // Rest of your business logic such as session.Query(SELECT * FROM ...)
        
          session.close();
    
    Replace the following:
    
      - `  DATABASE_ID  ` : the ID of your Spanner Omni database, for example, `test-db` .
    
      - `  ENDPOINT  ` : the endpoint of your Spanner Omni instance, for example, `localhost:15000` .
    
      - `  PATH_TO_CLIENT_CERT  ` : the path to your client certificate file.
    
      - `  PATH_TO_CLIENT_KEY_PKCS8  ` : the path to your client private key file in PKCS\#8 format.

## Sidecar proxy or standalone process

Deploying the Spanner Cassandra Java client as a sidecar proxy is an effective option for non-Java applications and tools, such as `cqlsh` , to connect to Spanner Omni using standard Cassandra drivers. This method runs the client as a standalone TCP proxy that intercepts Cassandra wire protocol traffic and translates it into gRPC for communication with Spanner Omni.

You can run the sidecar proxy using a YAML configuration file or by specifying system properties.

### Use a YAML configuration file

For production setups, we recommend that you use a YAML file to configure the adapter. This method supports multiple listeners and global settings:

    java -DconfigFilePath=PATH_TO_CONFIG_YAML -jar PATH_TO_ADAPTER_JAR

Example `config.yaml` :

    globalClientConfigs:
      enableBuiltInMetrics: false
      healthCheckEndpoint: "127.0.0.1:8080"
      spannerEndpoint: "ENDPOINT"
      instanceType: "omni"
      clientCertPath: "PATH_TO_CLIENT_CERT"
      clientKeyPath: "PATH_TO_CLIENT_KEY_PKCS8"
      usePlainText: "false"
    
    listeners:
      - name: "listener_1"
        host: "127.0.0.1"
        port: 9042
        spanner:
          databaseUri: "DATABASE_ID"
          numGrpcChannels: 4
          maxCommitDelayMillis: 5
      - name: "listener_2"
        host: "127.0.0.2"
        port: 9043
        spanner:
          databaseUri: "DATABASE_ID_2"
          numGrpcChannels: 8

### Use system properties

For a single listener or simpler deployments, you can run the sidecar proxy as a standalone process and configure its settings using Java system properties.

The following examples show how to run the sidecar proxy as a standalone process for each supported security configuration:

### Plain text

To run with plain-text communication, run the following command:

    java -DdatabaseUri=DATABASE_ID \
    -Dhost=127.0.0.1 \
    -Dport=9042 \
    -DnumGrpcChannels=4 \
    -DhealthCheckPort=8080 \
    -DspannerEndpoint=ENDPOINT \
    -DinstanceType=omni \
    -DusePlainText=true \
    -jar PATH_TO_ADAPTER_JAR

### TLS

To run with a TLS connection, run the following command:

    java -DdatabaseUri=DATABASE_ID \
    -Dhost=127.0.0.1 \
    -Dport=9042 \
    -DnumGrpcChannels=4 \
    -DhealthCheckPort=8080 \
    -DspannerEndpoint=ENDPOINT \
    -DinstanceType=omni \
    -jar PATH_TO_ADAPTER_JAR

### mTLS

To run with an mTLS connection, run the following command:

    java -DdatabaseUri=DATABASE_ID \
    -Dhost=127.0.0.1 \
    -Dport=9042 \
    -DnumGrpcChannels=4 \
    -DhealthCheckPort=8080 \
    -DspannerEndpoint=ENDPOINT \
    -DinstanceType=omni \
    -DclientCertPath=PATH_TO_CLIENT_CERT \
    -DclientKeyPath=PATH_TO_CLIENT_KEY_PKCS8 \
    -jar PATH_TO_ADAPTER_JAR

Replace the following:

  - `  DATABASE_ID  ` : the ID of your Spanner Omni database, for example, `test-db` .

  - `  ENDPOINT  ` : the endpoint of your Spanner Omni instance, for example, `localhost:15000` .

  - `  PATH_TO_ADAPTER_JAR  ` : the path to your Cassandra adapter JAR file.

  - `  PATH_TO_CLIENT_CERT  ` : the path to your client certificate file.

  - `  PATH_TO_CLIENT_KEY_PKCS8  ` : the path to your client private key file in PKCS\#8 format.
