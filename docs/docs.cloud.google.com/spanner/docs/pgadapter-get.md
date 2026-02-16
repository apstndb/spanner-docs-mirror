This page explains how to get the PGAdapter binary. PGAdapter is used with PostgreSQL-dialect databases.

  - To run the PGAdapter standalone in the JVM, download the latest `  jar  ` file and a new `  lib  ` directory that contains the required dependencies.
    
    ``` text
    wget https://storage.googleapis.com/pgadapter-jar-releases/pgadapter.tar.gz \
      && tar -xzvf pgadapter.tar.gz
    ```

  - To run PGAdapter in a Docker container, get the latest version by executing the following command:
    
    ``` text
    docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter
    ```
    
    Get a previous version by appending the version as a tag.
    
    ``` text
    docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter:v<version-number>
    ```

  - To use PGAdapter in a process:
    
    Add `  google-cloud-spanner-pgadapter  ` as a dependency to your project by adding the following code to your `  pom.xml  ` file:
    
    ``` markdown
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>google-cloud-spanner-pgadapter</artifactId>
      <version>0.53.1</version>
    </dependency>
    ```

## What's next

  - Learn how to [Start PGAdapter](/spanner/docs/pgadapter-start) .
