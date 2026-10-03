---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy
title: gcloud spanner backup-schedules get-iam-policy
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner backup-schedules get-iam-policy - get the IAM policy for a Cloud Spanner backup schedule

SYNOPSIS

`gcloud spanner backup-schedules get-iam-policy` ( [`BACKUP_SCHEDULE`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#BACKUP_SCHEDULE) : [`--database`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#--database) = `DATABASE` [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#--instance) = `INSTANCE` ) \[ [`--filter`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#--filter) = `EXPRESSION` \] \[ [`--limit`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#--limit) = `LIMIT` \] \[ [`--page-size`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#--page-size) = `PAGE_SIZE` \] \[ [`--sort-by`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#--sort-by) =\[ `FIELD` , …\]\] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backup-schedules/get-iam-policy#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

`gcloud spanner backup-schedules get-iam-policy` displays the IAM policy associated with a Cloud Spanner backup schedule. If formatted as JSON, the output can be edited and used as a policy file for `set-iam-policy` . The output includes an "etag" field identifying the version emitted and allowing detection of concurrent policy updates; see \$ {parent} set-iam-policy for additional details.

EXAMPLES

To print the IAM policy for a given Cloud Spanner backup schedule, run:

```
gcloud spanner backup-schedules get-iam-policy backup-schedule-id --instance=instance-id --database=database-id
```

POSITIONAL ARGUMENTS

BackupSchedule resource - The Cloud Spanner backup schedule for which to display the IAM policy. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

To set the `project` attribute:

- provide the argument `backup_schedule` on the command line with a fully specified name;
- provide the argument `--project` on the command line;
- set the property `core/project` .

This must be specified.

`BACKUP_SCHEDULE`  
ID of the backupSchedule or fully qualified identifier for the backupSchedule.

To set the `backup_schedule` attribute:

- provide the argument `backup_schedule` on the command line.

This positional argument must be specified if any of the other arguments in this group are specified.

`--database` = `DATABASE`  
The name of the Cloud Spanner database. To set the `database` attribute:

- provide the argument `backup_schedule` on the command line with a fully specified name;
- provide the argument `--database` on the command line.

`--instance` = `INSTANCE`  
The name of the Cloud Spanner instance. To set the `instance` attribute:

- provide the argument `backup_schedule` on the command line with a fully specified name;
- provide the argument `--instance` on the command line;
- set the property `spanner/instance` .

LIST COMMAND FLAGS

`--filter` = `EXPRESSION`  
Apply a Boolean filter `EXPRESSION` to each resource item to be listed. If the expression evaluates `True` , then that item is listed. For more details and examples of filter expressions, run \$ [gcloud topic filters](https://docs.cloud.google.com/sdk/gcloud/reference/topic/filters) . This flag interacts with other flags that are applied in this order: `--flatten` , `--sort-by` , `--filter` , `--limit` .

`--limit` = `LIMIT`  
Maximum number of resources to list. The default is `unlimited` . This flag interacts with other flags that are applied in this order: `--flatten` , `--sort-by` , `--filter` , `--limit` .

`--page-size` = `PAGE_SIZE`  
Some services group resource list output into pages. This flag specifies the maximum number of resources per page. The default is determined by the service if it supports paging, otherwise it is `unlimited` (no paging). Paging may be applied before or after `--filter` and `--limit` depending on the service.

`--sort-by` =\[ `FIELD` ,…\]  
Comma-separated list of resource field key names to sort by. The default order is ascending. Prefix a field with \`\`\~´´ for descending order on that field. This flag interacts with other flags that are applied in this order: `--flatten` , `--sort-by` , `--filter` , `--limit` .

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

API REFERENCE

This command uses the `spanner/v1` API. The full documentation for this API can be found at: <https://cloud.google.com/spanner/>

NOTES

These variants are also available:

```
gcloud alpha spanner backup-schedules get-iam-policy
```

```
gcloud beta spanner backup-schedules get-iam-policy
```
