---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/init
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/init
title: gcloud spanner samples init
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner samples init - initialize a Cloud Spanner sample app

SYNOPSIS

`gcloud spanner samples init` [`APPNAME`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/init#APPNAME) [`--instance-id`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/init#--instance-id) = `INSTANCE_ID` \[ [`--database-id`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/init#--database-id) = `DATABASE_ID` \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples/init#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

This command creates a Cloud Spanner database in the given instance for the sample app and loads any initial data required by the application.

EXAMPLES

To initialize the 'finance' sample app using instance 'my-instance', run:

```
gcloud spanner samples init finance --instance-id=my-instance
```

To initialize the 'finance-graph' sample app using instance 'my-instance', run:

```
gcloud spanner samples init finance-graph --instance-id=my-instance
```

POSITIONAL ARGUMENTS

`APPNAME`  
The sample app name, e.g. "finance", "finance-graph".

REQUIRED FLAGS

`--instance-id` = `INSTANCE_ID`  
The Cloud Spanner instance ID for the sample app.

OPTIONAL FLAGS

`--database-id` = `DATABASE_ID`  
ID of the new Cloud Spanner database to create for the sample app.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner samples init
```

```
gcloud beta spanner samples init
```
