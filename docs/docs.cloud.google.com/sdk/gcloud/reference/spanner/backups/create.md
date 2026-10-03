---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create
title: gcloud spanner backups create
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner backups create - creates a backup of a Cloud Spanner database

SYNOPSIS

`gcloud spanner backups create` ( [`BACKUP`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#BACKUP) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--instance) = `INSTANCE` ) [`--database`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--database) = `DATABASE` ( [`--expiration-date`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--expiration-date) = `EXPIRATION_DATE` \| [`--retention-period`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--retention-period) = `RETENTION_PERIOD` ) \[ [`--async`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--async) \] \[ [`--version-time`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--version-time) = `TIMESTAMP` \] \[ [`--encryption-type`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--encryption-type) = `ENCRYPTION_TYPE` [`--kms-keys`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--kms-keys) =\[ `KMS_KEYS` , …\] \| \[ [`--kms-key`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--kms-key) = `KMS_KEY` : [`--kms-keyring`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--kms-keyring) = `KMS_KEYRING` [`--kms-location`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--kms-location) = `KMS_LOCATION` [`--kms-project`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#--kms-project) = `KMS_PROJECT` \]\] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/create#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Creates a backup of a Cloud Spanner database.

EXAMPLES

To create a backup asynchronously, run:

```
gcloud spanner backups create BACKUP_ID --instance=INSTANCE_NAME --database=DATABASE --expiration-date=2020-03-29T10:49:41Z --async
```

To create a backup synchronously, run:

```
gcloud spanner backups create BACKUP_ID --instance=INSTANCE_NAME --database=DATABASE --retention-period=2w
```

POSITIONAL ARGUMENTS

Backup resource - The Cloud Spanner backup to create. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

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
The Cloud Spanner instance for the backup.

To set the `instance` attribute:

- provide the argument `backup` on the command line with a fully specified name;
- provide the argument `--instance` on the command line;
- set the property `spanner/instance` .

REQUIRED FLAGS

`--database` = `DATABASE`

ID of the database from which the backup will be created.

Exactly one of these must be specified:

`--expiration-date` = `EXPIRATION_DATE`  
Expiration time of the backup, must be at least 6 hours and at most 30 days from the time the request is received. See `$ `[`gcloud topic datetimes`](https://docs.cloud.google.com/sdk/gcloud/reference/topic/datetimes) for information on date/time formats.

`--retention-period` = `RETENTION_PERIOD`  
Retention period of the backup relative from now, must be at least 6 hours and at most 30 days. See `$ `[`gcloud topic datetimes`](https://docs.cloud.google.com/sdk/gcloud/reference/topic/datetimes) for information on duration formats.

OPTIONAL FLAGS

`--async`

Return immediately, without waiting for the operation in progress to complete.

`--version-time` = `TIMESTAMP`

The backup will contain an externally consistent copy of the database at the timestamp specified by `--version-time` . If `--version-time` is not specified, the system will use the creation time of the backup.

`--encryption-type` = `ENCRYPTION_TYPE`

The encryption type of the backup. `ENCRYPTION_TYPE` must be one of:

`customer-managed-encryption`  
Use the provided Cloud KMS key for encryption.If this option is selected, kms-key must be set.

`google-default-encryption`  
Use Google default encryption.

`use-database-encryption`  
Use the same encryption configuration as the database.

KMS key name group

At most one of these can be specified:

Key resource - Cloud KMS key(s) to be used to create the Cloud Spanner backup. This represents a Cloud resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

To set the `kms-project` attribute:

- provide the argument `--kms-keys` on the command line with a fully specified name.

To set the `kms-location` attribute:

- provide the argument `--kms-keys` on the command line with a fully specified name.

To set the `kms-keyring` attribute:

- provide the argument `--kms-keys` on the command line with a fully specified name.

`--kms-keys` =\[ `KMS_KEYS` ,…\]

IDs of the keys or fully qualified identifiers for the keys.

To set the `kms-key` attribute:

- provide the argument `--kms-keys` on the command line.

Key resource - Cloud KMS key to be used to create the Cloud Spanner backup. The arguments in this group can be used to specify the attributes of this resource.

`--kms-key` = `KMS_KEY`

ID of the key or fully qualified identifier for the key.

To set the `kms-key` attribute:

- provide the argument `--kms-key` on the command line.

This flag argument must be specified if any of the other arguments in this group are specified.

`--kms-keyring` = `KMS_KEYRING`

KMS keyring id of the key.

To set the `kms-keyring` attribute:

- provide the argument `--kms-key` on the command line with a fully specified name;
- provide the argument `--kms-keyring` on the command line.

`--kms-location` = `KMS_LOCATION`

Cloud location for the key.

To set the `kms-location` attribute:

- provide the argument `--kms-key` on the command line with a fully specified name;
- provide the argument `--kms-location` on the command line.

`--kms-project` = `KMS_PROJECT`

Cloud project id for the key.

To set the `kms-project` attribute:

- provide the argument `--kms-key` on the command line with a fully specified name;
- provide the argument `--kms-project` on the command line.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner backups create
```

```
gcloud beta spanner backups create
```
