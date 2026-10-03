---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/workload
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/workload
title: gcloud spanner samples workload
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner samples workload - generate gRPC traffic for a given sample app's backend service

SYNOPSIS

`gcloud spanner samples workload` [`APPNAME`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/workload#APPNAME) \[ [`--duration`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/workload#--duration) = `DURATION` ; default="1h"\] \[ [`--port`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/workload#--port) = `PORT` \] \[ [`--target-qps`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/workload#--target-qps) = `TARGET_QPS` \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/workload#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Before sending traffic to the backend service, create the database and start the service with:

```
gcloud spanner samples init APPNAME --instance-id=INSTANCE_ID
gcloud spanner samples backend APPNAME --instance-id=INSTANCE_ID
```

To run all three steps together, use:

```
gcloud spanner samples run APPNAME --instance-id=INSTANCE_ID
```

EXAMPLES

To generate traffic for the 'finance' sample app, run:

```
gcloud spanner samples workload finance
```

POSITIONAL ARGUMENTS

`APPNAME`  
The sample app name, e.g. "finance".

FLAGS

`--duration` = `DURATION` ; default="1h"  
Duration of time allowed to run before stopping the workload.

`--port` = `PORT`  
Port of the running backend service.

`--target-qps` = `TARGET_QPS`  
Target requests per second.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner samples workload
```

```
gcloud beta spanner samples workload
```
