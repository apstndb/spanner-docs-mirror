---
name: documents/docs.cloud.google.com/spanner/docs/pgadapter-get
uri: https://docs.cloud.google.com/spanner/docs/pgadapter-get
title: Get PGAdapter
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

This document explains how to get the PGAdapter binary. Use PGAdapter with PostgreSQL-dialect databases.

Run PGAdapter in a Docker container, as a standalone process, or in-process with a Java application.

## Run in a Docker container

Run PGAdapter in a Docker container using one of the following options:

### Standard image

To get the latest version of the standard image, run the following command:

    docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter

To get a previous version, append the version as a tag:

    docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter:vVERSION_NUMBER

### Distroless image

We also publish a [distroless Docker image](https://github.com/GoogleContainerTools/distroless) under the tag `gcr.io/cloud-spanner-pg-adapter/pgadapter-distroless` . This image runs PGAdapter as a non-root user.

To pull the distroless image, run the following command:

    docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter-distroless

## Run as a standalone process

Run PGAdapter as a standalone process in the JVM by getting the JAR file using one of the following options:

### Prebuilt JAR (latest version)

To download the latest JAR file and dependencies, run the following command:

    wget https://storage.googleapis.com/pgadapter-jar-releases/pgadapter.tar.gz \
      && tar -xzvf pgadapter.tar.gz

### Prebuilt JAR (specific version)

To download a specific version of the JAR file, set the version when you download it.

    VERSION=VERSION_NUMBER
    wget https://storage.googleapis.com/pgadapter-jar-releases/pgadapter-${VERSION}.tar.gz \
      && tar -xzvf pgadapter-${VERSION}.tar.gz

### Locally built JAR (from source)

To build the JAR file and assemble dependencies from source, run the following commands:

1.  Clone the repository:
    
        git clone https://github.com/GoogleCloudPlatform/pgadapter.git

2.  Build the package:
    
        mvn package -P assembly

The build process creates the binaries in the `target/pgadapter` folder.

## Use in a Java process

Add `google-cloud-spanner-pgadapter` as a dependency to your project.

### Maven

If you use Maven, add the following dependency to your `pom.xml` file:

    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-pgadapter</artifactId>
      <version>0.55.2</version>
    </dependency>

## What's next

  - Learn how to [Start PGAdapter](https://docs.cloud.google.com/spanner/docs/pgadapter-start) .
