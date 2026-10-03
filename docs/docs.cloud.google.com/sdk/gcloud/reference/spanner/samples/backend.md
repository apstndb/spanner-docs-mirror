---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend
title: gcloud spanner samples backend
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner samples backend - run the backend gRPC service for the given Cloud Spanner sample app

SYNOPSIS

`gcloud spanner samples backend` [`APPNAME`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend#APPNAME) [`--instance-id`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend#--instance-id) = `INSTANCE_ID` \[ [`--database-id`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend#--database-id) = `DATABASE_ID` \] \[ [`--duration`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend#--duration) = `DURATION` ; default="1h"\] \[ [`--port`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend#--port) = `PORT` \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/backend#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

This command starts the backend gRPC service for the given sample application. Before starting the service, create the database and load any initial data with:

```
gcloud spanner samples init APPNAME --instance-id=INSTANCE_ID
```

After starting the service, generate traffic with:

```
gcloud spanner samples workload APPNAME
```

To run all three steps together, use:

```
gcloud spanner samples run APPNAME --instance-id=INSTANCE_ID
```

EXAMPLES

To run the backend gRPC service for the 'finance' sample app using instance 'my-instance', run:

```
gcloud spanner samples backend finance --instance-id=my-instance
```

POSITIONAL ARGUMENTS

`APPNAME`  
The sample app name, e.g. "finance".

REQUIRED FLAGS

`--instance-id` = `INSTANCE_ID`  
The Cloud Spanner instance ID for the sample app.

OPTIONAL FLAGS

`--database-id` = `DATABASE_ID`  
The Cloud Spanner database ID for the sample app.

`--duration` = `DURATION` ; default="1h"  
Duration of time allowed to run before stopping the service.

`--port` = `PORT`  
Port on which to receive gRPC requests.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner samples backend
```

```
gcloud beta spanner samples backend
```
