---
name: documents/docs.cloud.google.com/spanner-omni/java
uri: https://docs.cloud.google.com/spanner-omni/java
title: Use the Java client library to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

The Java client library for Spanner works with Spanner Omni in the same way it works with Spanner. This document shows you how to establish secure connections to Spanner Omni by configuring the Java client library. You establish these connections by setting client options when you create a database administrative client or a database client.

The Java client library supports plain text, TLS, TLS with credentials, and mTLS connections.

For more information, see [Get started with Spanner in Java](https://docs.cloud.google.com/spanner/docs/getting-started/java) in the Spanner documentation.

## Before you begin

To get started with Spanner Omni in Java, use the Java client library version 6.119.0 or later.

If you use Maven without the Bill of Materials (BOM), add the following to the `pom.xml` file dependencies:

    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner</artifactId>
      <version>6.119.0</version>
    </dependency>

## Security configurations

The Spanner Java client library supports four security configurations, which define how communication is encrypted and authenticated between the client and Spanner Omni. The following table describes each configuration:

| Security configuration | Description                                                                                                                                                                                                                                                                               |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plain text             | Communication is not encrypted.                                                                                                                                                                                                                                                           |
| TLS                    | Communication is encrypted using Transport Layer Security (TLS). This configuration requires that you add the Spanner Omni CA certificate to the Java truststore, as described in [Configure the Java truststore](https://docs.cloud.google.com/spanner-omni/java#configure-truststore) . |
| TLS with credentials   | Communication is encrypted using TLS, and authentication is performed using a username and password.                                                                                                                                                                                      |
| mTLS                   | Communication is encrypted using mutual TLS (mTLS). This configuration requires you to provide both a client certificate and a client private key.                                                                                                                                        |

## Configure the Java truststore

For all encrypted connection types (TLS, TLS with credentials, and mTLS), you must add the Spanner Omni CA certificate to the Java truststore so that the client can verify the server's certificate.

To add the CA certificate to the default Java truststore, run the following command:

    sudo keytool -import -trustcacerts -file ~/.spanner/certs/ca.crt -alias spanner-ca -keystore $JAVA_HOME/lib/security/cacerts

Alternatively, you can specify a custom truststore when you run the application:

1.  To maintain compatibility with other services that use standard certificate authorities (CAs), copy the default Java truststore:
    
        cp $JAVA_HOME/lib/security/cacerts /PATH_TO_CUSTOM_CACERTS

2.  Import the CA certificate into your custom truststore:
    
        keytool -import -trustcacerts -file ~/.spanner/certs/ca.crt -alias spanner-ca -keystore /PATH_TO_CUSTOM_CACERTS

3.  Specify the custom truststore using JVM system properties when you run the application:
    
        java -Djavax.net.ssl.trustStore=/PATH_TO_CUSTOM_CACERTS -Djavax.net.ssl.trustStorePassword=changeit app

## Configure the `SpannerOptions` object

When configuring the [`SpannerOptions`](https://docs.cloud.google.com/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.SpannerOptions) object to create a [`DatabaseClient`](https://docs.cloud.google.com/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.DatabaseClient) or [`DatabaseAdminClient`](https://docs.cloud.google.com/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.InstanceAdminClient) , specify the Spanner Omni endpoint using `setHost()` followed by `setType(SpannerOptions.InstanceType.OMNI)` .

The following examples show how to configure the `SpannerOptions` object for each supported security configuration:

### Plain text

To establish a plain-text connection, specify the Spanner Omni endpoint with `http://` and use the `usePlainText()` method:

    SpannerOptions options =
        SpannerOptions.newBuilder()
            .setHost("http://ENDPOINT") // Replace with your Spanner Omni endpoint
            .setType(SpannerOptions.InstanceType.OMNI)
            .usePlainText()
            .build();
    Spanner spanner = options.getService();

### TLS

When you configure the `SpannerOptions` object for a TLS connection, you don't need to specify username and password credentials. Specify the Spanner Omni endpoint using `https://` :

    SpannerOptions options =
        SpannerOptions.newBuilder()
            .setHost("https://ENDPOINT") // Replace with your Spanner Omni endpoint
            .setType(SpannerOptions.InstanceType.OMNI)
            .build();
    Spanner spanner = options.getService();

### TLS with credentials

To establish a TLS connection with username and password authentication, specify the Spanner Omni endpoint using `https://` and the username and password using the `login()` method:

    SpannerOptions options =
        SpannerOptions.newBuilder()
            .setHost("https://ENDPOINT") // Replace with your Spanner Omni endpoint
            .setType(SpannerOptions.InstanceType.OMNI)
            .login("USERNAME", "PASSWORD".toCharArray())
            .build();
    Spanner spanner = options.getService();

### mTLS

<span id="mtls"></span> To use an mTLS connection, convert the key generated by Spanner Omni to a format compliant with Java using the following command:

    openssl pkcs8 -topk8 -in ~/.spanner/certs/client.key -out ~/.spanner/certs/java-client.key -nocrypt

The following example shows how to configure the `SpannerOptions` object to use a client certificate:

    SpannerOptions options =
        SpannerOptions.newBuilder()
            .setHost("https://ENDPOINT") // Replace with your Spanner Omni endpoint
            .setType(SpannerOptions.InstanceType.OMNI)
            .useClientCert(
                "PATH_TO_CLIENT_CERT",
                "PATH_TO_CLIENT_CERT_KEY")
            .build();
    Spanner spanner = options.getService();

## Get a database client

After you configure the `SpannerOptions` object, you can get a database client. Because Spanner Omni does not use Google Cloud project or instance IDs, specify `default` for the project ID and instance ID when you create a `DatabaseId` :

    DatabaseId dbId = DatabaseId.of("default", "default", "DATABASE_ID");
    DatabaseClient client = spanner.getDatabaseClient(dbId);
