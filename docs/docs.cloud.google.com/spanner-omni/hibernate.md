---
name: documents/docs.cloud.google.com/spanner-omni/hibernate
uri: https://docs.cloud.google.com/spanner-omni/hibernate
title: Use Hibernate to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

[Hibernate ORM](https://hibernate.org/orm/) is an object-relational mapping (ORM) tool for the Java programming language. It provides a framework for mapping an object-oriented domain model to a relational database. You can integrate GoogleSQL-dialect databases with Hibernate using the open-source [Spanner dialect](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate) ( `SpannerDialect` ).

This document describes how to configure Hibernate to connect to Spanner Omni. Hibernate integrates with Spanner Omni in the same way it integrates with Spanner. Spanner is compatible with Hibernate ORM 6.x. The Spanner dialect produces SQL, DML, and DDL statements for most common entity types and relationships using standard Hibernate and Java Persistence annotations.

Using Hibernate with Spanner Omni lets you use your existing Hibernate experience to interact with your databases. For more information, see [Integrate Spanner with Hibernate ORM (Spanner dialect)](https://docs.cloud.google.com/spanner/docs/use-hibernate) in the Spanner documentation.

## Prerequisites

To use Hibernate with Spanner Omni, include the following Maven dependencies in your project:

  - **Spanner Hibernate dialect** : Add the following to your `pom.xml` file. Use [version 4.2.1](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/releases/tag/v4.2.1) or later.
    
        <dependency>
          <groupId>com.google.cloud</groupId>
          <artifactId>google-cloud-spanner-hibernate-dialect</artifactId>
          <version>4.2.1</version>
        </dependency>

  - **Spanner JDBC driver** : Add the following to your `pom.xml` file. Use [version 2.35.0](https://github.com/googleapis/java-spanner-jdbc/releases/tag/v2.35.0) or later.
    
        <dependency>
          <groupId>com.google.cloud</groupId>
          <artifactId>google-cloud-spanner-jdbc</artifactId>
          <version>2.35.0</version>
        </dependency>

> **Note:** Formal support for Hibernate ORM with Spanner Omni is provided through the open-source Spanner JDBC driver.

## Configure Hibernate

Configure the `SpannerDialect` and Spanner driver class in the `hibernate.properties` file using standard Hibernate dialect conventions:

    hibernate.dialect=com.google.cloud.spanner.hibernate.SpannerDialect
    hibernate.connection.driver_class=com.google.cloud.spanner.jdbc.JdbcDriver
    hibernate.connection.url=jdbc:spanner://ENDPOINT/databases/DATABASE_ID?isExperimentalHost=true

Update the `hibernate.connection.url` for the specific connection protocol (plain-text, TLS, or mTLS) that your Spanner Omni instance uses. For more information, see [Establish a Spanner Omni connection](https://docs.cloud.google.com/spanner-omni/jdbc-driver#establish-jdbc-connection) . For secure modes, add the CA certificate to the Java truststore or pass it directly when you run the application, as described in the [Java SDK TLS instructions](https://docs.cloud.google.com/spanner-omni/java#configure-options) .
