This page explains how to start PGAdapter in Spanner. To learn about PGAdapter, see [About PGAdapter](/spanner/docs/pgadapter) . To get the PGAdapter binary, see [Get PGAdapter](/spanner/docs/pgadapter-get) .

You can start PGAdapter in the following ways:

  - As a standalone process
  - Within a Docker container
  - On Cloud Run
  - Using PGAdapter as a sidecar proxy (for example, in a Kubernetes cluster)
  - In-process with your Java application

## Before you begin

Before starting PGAdapter, ensure that you have authenticated with either a user account or service account on the machine where PGAdapter will be running. If you are using a service account, you must know the location of the JSON key file (the credentials file). You can then either specify the credentials path with the PGAdapter `  -c  ` option, or by setting the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable.

For more information, see:

  - [Authenticate to Spanner](/spanner/docs/authentication)
  - [Authorizing the gcloud CLI](/sdk/docs/authorizing)

## Choose a method for running PGAdapter

You can start PGAdapter as a standalone process, within a Docker container, on Cloud Run, or in-process with your Java application. When you start PGAdapter, you specify the project, Spanner instance ID, and database to connect to. You can also specify the path for a JSON-formatted credentials file (key file).

### Standalone

Download PGAdapter with the following command.

``` text
    wget https://storage.googleapis.com/pgadapter-jar-releases/pgadapter.tar.gz \
    && tar -xzvf pgadapter.tar.gz
    
```

Start PGAdapter with the following command.

``` text
    java -jar pgadapter.jar -p PROJECT_ID -i INSTANCE_ID -d DATABASE_ID \
    -c CREDENTIALS_FILE_PATH \
    ADDITIONAL_OPTIONS
    
```

The following options are required:

  - `  -p project_id  `  
    ID of the project that the Spanner database is running in.
  - `  -i instance_id  `  
    Spanner instance ID.
  - `  -d database_ID  `  
    ID of the Spanner database to connect to.

The following options are optional:

  - `  -r databaseRole= database_role  `  
    Database role to use for the session. For more information, see [Authorization with PGAdapter](/spanner/docs/pgadapter#pgadapter-authorization) .

  - `  -c credentials_file_path  `  
    Full path for the keys file containing the service account credentials in JSON format. If this option is not set, credentials are read from the path specified by the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable.
    
    To learn how to create a service account and download a JSON-formatted key file, see [Creating a service account](https://cloud.google.com/docs/authentication/production#create_service_account) .
    
    Ensure that you grant the service account sufficient credentials to access the database.
    
    You can omit this option if you first authenticate with the Google Cloud CLI with the following command:
    
    `  gcloud auth application-default login  `
    
    For more information, see [Set up authentication and authorization.](/spanner/docs/getting-started/set-up#set_up_authentication_and_authorization)

  - `  -s port  `  
    Port that PGAdapter listens on. Defaults to 5432 (the default PostgreSQL port).

  - `  -v version_number  `  
    Version number of PostgreSQL to expose to the client during connection. Default value is 14.1
    
    Some PostgreSQL applications and drivers enable additional features depending on this version number. Spanner might not support these features. See [Drivers and Clients](https://github.com/GoogleCloudPlatform/pgadapter#drivers-and-clients) for a full list of supported clients.

  - `  -x  `  
    Enable connections from hosts other than localhost. *Don't use when starting PGAdapter in standalone mode. Use only when starting within a Docker container* .
    
    By default, as a security measure, PGAdapter accepts connections only from localhost.

The following example starts PGAdapter in standalone mode on port 5432 using the default application credentials.

``` bash
java -jar pgadapter.jar \
-p my-project \
-i my-instance \
-d my-database \
-s 5432
```

### Docker

Start PGAdapter with the following command.

``` text
docker run -d -p HOST-PORT:DOCKER-PORT \
-v CREDENTIALS_FILE_PATH:/acct_credentials.json \
gcr.io/cloud-spanner-pg-adapter/pgadapter:latest \
-p PROJECT_ID -i INSTANCE_ID -d DATABASE_ID  \
-c /acct_credentials.json -x OTHER_PGAdapter_OPTIONS
```

In addition to the PGAdapter options to specify project, instance, database, and credentials, the following options are required:

  - `  -p 127.0.0.1: HOST-PORT : DOCKER-PORT  `  
    This Docker option maps the port `  DOCKER-PORT  ` inside the Docker container to the port `  HOST-PORT  ` outside the container. `  DOCKER-PORT  ` must match how PGAdapter is configured inside the container. It defaults to 5432. `  HOST-PORT  ` is the port that Docker should listen on outside the container for connection requests. It must always be an available port on localhost.
    
    For more information, see [Publish or expose port (-p, --expose)](https://docs.docker.com/engine/reference/commandline/run/#publish-or-expose-port--p---expose) in the Docker documentation.

  - `  -v CREDENTIALS_FILE_PATH : in_container_mount_point  `  
    This Docker option bind mounts a shared volume. It maps the host path outside the container to a volume (mount point) inside the container. The host and container paths are separated by a colon (:).
    
    This option lets PGAdapter access the JSON credentials file that is outside the container. In the preceding example, the `  -c  ` option references the in-container mount point. This example names the in-container mount point `  /acct_credentials.json  ` . You can name it whatever you want.
    
    For more information, see [VOLUME (shared filesystems)](https://docs.docker.com/engine/reference/run/#volume-shared-filesystems) in the Docker documentation.

  - `  -x  `  
    Enable connections from hosts other than localhost. This is needed because the port insider the container that is mapped to the host port does not appear to PGAdapter as localhost.

The following options are optional:

  - `  -r databaseRole= database_role  `  
    Database role to use for the session. For more information, see [Authorization with PGAdapter](/spanner/docs/pgadapter#pgadapter-authorization) .

In the following example, the Docker port and host port are both set to the PostgreSQL database service default port 5432.

``` text
docker run -d -p 127.0.0.1:5432:5432 \
-v /tmp/credentials.json:/acct_credentials.json \
gcr.io/cloud-spanner-pg-adapter/pgadapter:latest \
-p my_project -i my_instance -d my_database \
-c /acct_credentials.json -x
```

### Cloud Run

You can't deploy PGAdapter as a standalone service on Cloud Run, but you can deploy it as a sidecar proxy.

Running PGAdapter in a sidecar pattern is recommended over running it as a separate service for the following reasons:

  - Prevents a single point of failure. Each application's access to your database is independent of the others, making them more resilient.
  - The number of PGAdapter instances automatically scales linearly with the number of application instances.

The PGAdapter GitHub repository contains several [working sample applications using Cloud Run and PGAdapter as a sidecar proxy](https://github.com/GoogleCloudPlatform/pgadapter/tree/postgresql-dialect/samples/cloud-run) for various programming languages.

The following configuration file shows how to add PGAdapter as a sidecar proxy to Cloud Run:

``` text
  apiVersion: serving.knative.dev/v1
  kind: Service
  metadata:
  annotations:
    # This example uses an in-memory volume for Unix domain sockets.
    # This is a Cloud Run beta feature.
    run.googleapis.com/launch-stage: BETA
  name: pgadapter-sidecar-example
  spec:
  template:
    metadata:
      annotations:
        run.googleapis.com/execution-environment: gen1
        # This registers 'pgadapter' as a dependency of 'app' and ensures that pgadapter starts
        # before the app container.
        run.googleapis.com/container-dependencies: '{"app":["pgadapter"]}'
    spec:
      # Create an in-memory volume that can be used for Unix domain sockets.
      volumes:
        - name: sockets-dir
          emptyDir:
            # This directory contains the virtual socket files that are used to
            # communicate between your application and PGAdapter.
            sizeLimit: 50Mi
            medium: Memory
      containers:
        # This is the main application container.
        - name: app
          # Example: europe-north1-docker.pkg.dev/my-test-project/cloud-run-source-deploy/pgadapter-sidecar-example
          image: MY-REGION.pkg.dev/MY-PROJECT/cloud-run-source-deploy/pgadapter-sidecar-example
          # The PGADAPTER_HOST variable is set to point to /sockets, which is the shared in-memory
          # volume that is used for Unix domain sockets.
          env:
            - name: SPANNER_PROJECT
              value: my-project
            - name: SPANNER_INSTANCE
              value: my-instance
            - name: SPANNER_DATABASE
              value: my-database
            - name: PGADAPTER_HOST
              value: /sockets
            - name: PGADAPTER_PORT
              value: "5432"
          ports:
            - containerPort: 8080
          volumeMounts:
            - mountPath: /sockets
              name: sockets-dir
        # This is the PGAdapter sidecar container.
        - name: pgadapter
          image: gcr.io/cloud-spanner-pg-adapter/pgadapter
          volumeMounts:
            - mountPath: /sockets
              name: sockets-dir
          args:
            - -dir /sockets
            - -x
          # Add a startup probe that checks that PGAdapter is listening on port 5432.
          startupProbe:
            initialDelaySeconds: 10
            timeoutSeconds: 10
            periodSeconds: 10
            failureThreshold: 3
            tcpSocket:
              port: 5432
  
```

### Sidecar Proxy

You can use PGAdapter as a sidecar proxy in, for example, a Kubernetes cluster. Kubernetes sidecar containers run in parallel with the main container in the pod.

Running PGAdapter in a sidecar pattern is recommended over running it as a separate service for the following reasons:

  - Prevents a single point of failure. Each application's access to your database is independent of the others, making them more resilient.
  - Because PGAdapter consumes resources in a linear relation to usage, this pattern lets you more accurately scope and request resources to match your applications as they scale.

The following configuration file shows how to add PGAdapter as a sidecar proxy to your Kubernetes cluster:

``` text
containers:
- name: pgadapter
image: gcr.io/cloud-spanner-pg-adapter/pgadapter
ports:
  - containerPort: 5432
args:
  - "-p my-project"
  - "-i my-instance"
  - "-d my-database"
  - "-x"
resources:
  requests:
    # PGAdapter's memory use scales linearly with the number of active
    # connections. Fewer open connections will use less memory. Adjust
    # this value based on your application's requirements.
    memory: "512Mi"
    # PGAdapter's CPU use scales linearly with the amount of IO between
    # the database and the application. Adjust this value based on your
    # application's requirements.
    cpu: "1"
```

The PGAdapter GitHub repository contains a [step-by-step guide and a sample application](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/sidecar-proxy) . This sample includes instructions for using [Workload Identity Federation for GKE](../../kubernetes-engine/docs/how-to/workload-identity) with PGAdapter.

### Java In-process

Create and start a PGAdapter instance with your Java code. This is the recommended setup for Java applications.

If you are using a service account for authentication, ensure that the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable is set to the path of the credentials file.

1.  Add `  google-cloud-spanner-pgadapter  ` as a dependency to your project. For details, see [Get PGAdapter](/spanner/docs/pgadapter-get) .
2.  Build a server using the `  com.google.cloud.spanner.pgadapter.ProxyServer  ` class.

The PGAdapter GitHub repository contains a [full sample application](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/jdbc) .

## Resource allocation guidance for the PGAdapter sidecar proxy

The following guidelines can help you configure the CPU and memory resources for the PGAdapter sidecar proxy. The optimal values vary depending on your specific workload.

### Memory

Because PGAdapter doesn't cache much data, it requires memory to convert query results from the Spanner gRPC format to the PostgreSQL wire protocol format. This is especially relevant for workloads with large binary columns because Spanner and PostgreSQL handle these data types differently.

To determine how much memory to allocate, use the following formula for memory allocation, where `  <var>number of concurrent connections</var>  ` is the number of simultaneous connections your application handles:

384 MB + (2 MB \* number of concurrent connections )

For example, if your application handles 200 concurrent connections, allocate approximately 784 MB of memory:

384 MB + (2 MB \* 200) = 784 MB

Start with this baseline memory allocation. Monitor its usage under a realistic load to fine-tune the value for your specific needs.

### CPU

PGAdapter isn't CPU-intensive. This is because its primary role is to act as a pass-through proxy. However, CPU usage increases with the amount of data being sent and received.

Consider the following factors when allocating CPU:

  - **Workload:** Applications that execute queries returning large amounts of data require more CPU power per connection than those that return only a few rows and columns.
  - **Application access pattern:** If your application accesses Spanner synchronously, it is idle while waiting for data from the proxy. In this case, the application and the proxy are less likely to compete for CPU resources.

Start with a baseline CPU allocation. Monitor its usage under a realistic load to fine-tune the value for your specific needs.

## What's next

  - [Connect `  psql  ` to a PostgreSQL database](/spanner/docs/psql-connect)
  - [Connect `  JDBC  ` to a PostgreSQL database](/spanner/docs/pg-jdbc-connect)
  - [Connect `  pgx  ` to a PostgreSQL database](/spanner/docs/pg-pgx-connect)
  - [Connect `  psycopg2  ` to a PostgreSQL database](/spanner/docs/pg-psycopg2-connect)
  - [Connect `  psycopg3  ` to a PostgreSQL database](/spanner/docs/pg-psycopg3-connect)
  - [Connect `  node-postgres  ` to a PostgreSQL database](/spanner/docs/pg-node-postgres-connect)
