---
name: documents/docs.cloud.google.com/spanner-omni/jdbc-driver
uri: https://docs.cloud.google.com/spanner-omni/jdbc-driver
title: Use the JDBC driver to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document shows you how to use the Spanner JDBC driver to connect your Java applications to Spanner Omni and execute SQL statements.

Java Database Connectivity (JDBC) is a standard Java API that provides a consistent way for applications to interact with relational databases. The Spanner JDBC driver works with Spanner Omni in the same way it works with Spanner.

By using the JDBC driver, you can leverage standard JDBC-compatible tools and libraries with Spanner Omni.

Spanner Omni JDBC connections support plain text, TLS, TLS with credentials, and mTLS.

For more information, see [Get started with Spanner in JDBC](https://docs.cloud.google.com/spanner/docs/getting-started/jdbc) in the Spanner documentation.

## Before you begin

To use Spanner Omni with the JDBC driver, use the Spanner JDBC driver version 2.41.0 or later.

If you use Maven without the Bill of Materials (BOM), add the following to the `pom.xml` file dependencies:

    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-jdbc</artifactId>
      <version>2.41.0</version>
    </dependency>

## Connection URL considerations

Because Spanner Omni is not directly connected to a Google Cloud project, the ` projects/ name  ` component is not required in the JDBC connection URL. Similarly, because each Spanner Omni deployment has a single, already-created instance ( `instances/default` ), the ` instances/ name  ` component is optional.

To connect the JDBC driver to Spanner Omni instead of Spanner, append the `;type=omni` property to the connection URL.

## Establish a Spanner Omni connection

The following examples show how to establish a connection with Spanner Omni using the Spanner JDBC driver for each supported security configuration:

### Plain text

To establish a plain-text connection, use a connection URL similar to the following:

    String url = "jdbc:spanner://HOST_ADDRESS:PORT/databases/DATABASE_ID;usePlainText=true;type=omni";
    try (java.sql.Connection connection = DriverManager.getConnection(url)) {
      try (ResultSet rs = connection.createStatement().executeQuery("SELECT * FROM Singers")) {
        while (rs.next()) {
          System.out.print(rs.getLong(1) + "\t");
          System.out.println(rs.getString(2));
        }
      }
    } catch (Exception e) {
      System.out.println(e.getMessage());
    }

### TLS

To establish a TLS connection, add the CA certificate to the Java truststore or specify a custom truststore when you run the application, as described in [Configure the Java truststore](https://docs.cloud.google.com/spanner-omni/java#configure-truststore) . The JDBC URL does not require any additional authentication parameters:

    String url = "jdbc:spanner://HOST_ADDRESS:PORT/databases/DATABASE_ID;type=omni";

### TLS with credentials

To establish a TLS connection with username and password authentication, add the CA certificate to the Java truststore as described in [Configure the Java truststore](https://docs.cloud.google.com/spanner-omni/java#configure-truststore) , and specify the `username` and `password` properties in the JDBC URL:

    String url = "jdbc:spanner://HOST_ADDRESS:PORT/databases/DATABASE_ID;type=omni;username=USERNAME;password=PASSWORD";

### mTLS

To establish an mTLS connection, add the CA certificate to the Java truststore as described in [Configure the Java truststore](https://docs.cloud.google.com/spanner-omni/java#configure-truststore) , and specify the `clientCertificate` and `clientKey` parameters in the JDBC URL. The client private key must be in a Java-compliant PKCS\#8 format, as described in the [Java SDK mTLS instructions](https://docs.cloud.google.com/spanner-omni/java#mtls) :

    String url = "jdbc:spanner://HOST_ADDRESS:PORT/databases/DATABASE_ID;type=omni;clientCertificate=PATH_TO_CLIENT_CERT;clientKey=PATH_TO_CLIENT_KEY";
