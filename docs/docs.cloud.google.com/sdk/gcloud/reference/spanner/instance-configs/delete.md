---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete
title: gcloud spanner instance-configs delete
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner instance-configs delete - delete a Cloud Spanner instance configuration

SYNOPSIS

`gcloud spanner instance-configs delete` [`INSTANCE_CONFIG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete#INSTANCE_CONFIG) \[ [`--etag`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete#--etag) = `ETAG` \] \[ [`--validate-only`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete#--validate-only) \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Delete a Cloud Spanner instance configuration.

EXAMPLES

To delete a custom Cloud Spanner instance configuration, run:

```
gcloud spanner instance-configs delete custom-instance-config
```

POSITIONAL ARGUMENTS

`INSTANCE_CONFIG`  
Cloud Spanner instance config.

FLAGS

`--etag` = `ETAG`  
Used for optimistic concurrency control as a way to help prevent simultaneous deletes of an instance config from overwriting each other.

`--validate-only`  
If specified, validate that the deletion will succeed without deleting the instance config.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner instance-configs delete
```

```
gcloud beta spanner instance-configs delete
```
