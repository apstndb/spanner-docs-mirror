---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/describe
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/describe
title: gcloud spanner backups describe
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner backups describe - retrieves information about a backup

SYNOPSIS

`gcloud spanner backups describe` ( [`BACKUP`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/describe#BACKUP) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/describe#--instance) = `INSTANCE` ) \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/describe#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Retrieves information about a backup.

EXAMPLES

To describe a backup, run:

```
gcloud spanner backups describe BACKUP_ID --instance=INSTANCE_NAME
```

POSITIONAL ARGUMENTS

Backup resource - Cloud Spanner backup to describe. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

To set the `project` attribute:

- provide the argument `backup` on the command line with a fully specified name;
- provide the argument `--project` on the command line;
- set the property `core/project` .

This must be specified.

`BACKUP`  
ID of the backup or fully qualified identifier for the backup.

To set the `backup` attribute:

- provide the argument `backup` on the command line.

This positional argument must be specified if any of the other arguments in this group are specified.

`--instance` = `INSTANCE`  
The name of the Cloud Spanner instance. To set the `instance` attribute:

- provide the argument `backup` on the command line with a fully specified name;
- provide the argument `--instance` on the command line;
- set the property `spanner/instance` .

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

API REFERENCE

This command uses the `spanner/v1` API. The full documentation for this API can be found at: <https://cloud.google.com/spanner/>

NOTES

These variants are also available:

```
gcloud alpha spanner backups describe
```

```
gcloud beta spanner backups describe
```
