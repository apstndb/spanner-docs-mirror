---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum
title: gcloud spanner databases change-quorum
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner databases change-quorum - change quorum of a Cloud Spanner database

SYNOPSIS

`gcloud spanner databases change-quorum` ( [`DATABASE`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum#DATABASE) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum#--instance) = `INSTANCE` ) ( [`--dual-region`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum#--dual-region) \| [`--serving-location`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum#--serving-location) = `SERVING_LOCATION` [`--single-region`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum#--single-region) ) \[ [`--etag`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum#--etag) = `ETAG` \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/change-quorum#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Change quorum of a Cloud Spanner database.

EXAMPLES

To trigger change quorum from single-region mode to dual-region mode, run:

```
gcloud spanner databases change-quorum my-database-id --instance=my-instance-id --dual-region
```

To trigger change quorum from dual-region mode to single-region mode with serving location as `asia-south1` , run:

```
gcloud spanner databases change-quorum my-database-id --instance=my-instance-id --single-region --serving-location=asia-south1
```

To trigger change quorum using etag specified, run:

```
gcloud spanner databases change-quorum my-database-id --instance=my-instance-id --dual-region --etag=ETAG
```

POSITIONAL ARGUMENTS

Database resource - The Cloud Spanner database to change quorum. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

To set the `project` attribute:

- provide the argument `database` on the command line with a fully specified name;
- provide the argument `--project` on the command line;
- set the property `core/project` .

This must be specified.

`DATABASE`  
ID of the database or fully qualified identifier for the database.

To set the `database` attribute:

- provide the argument `database` on the command line.

This positional argument must be specified if any of the other arguments in this group are specified.

`--instance` = `INSTANCE`  
The Cloud Spanner instance for the database.

To set the `instance` attribute:

- provide the argument `database` on the command line with a fully specified name;
- provide the argument `--instance` on the command line;
- set the property `spanner/instance` .

REQUIRED FLAGS

Exactly one of these must be specified:

Command-line flag for dual-region quorum change:  
`--dual-region`  
Switch to dual-region quorum type.

Command-line flags for single-region quorum change:  
`--serving-location` = `SERVING_LOCATION`  
The cloud Spanner location.

This flag argument must be specified if any of the other arguments in this group are specified.

`--single-region`  
Switch to single-region quorum type.

This flag argument must be specified if any of the other arguments in this group are specified.

OPTIONAL FLAGS

`--etag` = `ETAG`  
Used for optimistic concurrency control.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner databases change-quorum
```

```
gcloud beta spanner databases change-quorum
```
