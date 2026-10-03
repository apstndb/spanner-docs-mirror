---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/describe
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/describe
title: gcloud spanner instance-configs describe
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner instance-configs describe - describe a Cloud Spanner instance configuration

SYNOPSIS

`gcloud spanner instance-configs describe` [`INSTANCE_CONFIG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/describe#INSTANCE_CONFIG) \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/describe#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Describe a Cloud Spanner instance configuration.

EXAMPLES

To describe an instance config named regional-us-central1, run:

```
gcloud spanner instance-configs describe regional-us-central1
```

To describe an instance config named nam-eur-asia1, run:

```
gcloud spanner instance-configs describe nam-eur-asia1
```

POSITIONAL ARGUMENTS

`INSTANCE_CONFIG`  
Cloud Spanner instance config.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner instance-configs describe
```

```
gcloud beta spanner instance-configs describe
```
