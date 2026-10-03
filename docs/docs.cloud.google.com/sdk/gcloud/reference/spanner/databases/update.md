---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update
title: gcloud spanner databases update
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner databases update - update a Cloud Spanner database

SYNOPSIS

`gcloud spanner databases update` ( [`DATABASE`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#DATABASE) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#--instance) = `INSTANCE` ) \[ [`--async`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#--async) \] \[ [`--clear-kms-keys`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#--clear-kms-keys) \| [`--[no-]enable-drop-protection`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#--%5Bno-%5Denable-drop-protection) \| [`--kms-keys`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#--kms-keys) = `KMS_KEY` , \[ [`KMS_KEY`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#KMS_KEY) , …\]\] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/update#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Update a Cloud Spanner database.

EXAMPLES

To enable database deletion protection on a Cloud Spanner database 'my-database', run:

```
gcloud spanner databases update my-database --enable-drop-protection
```

To disable database deletion protection on a Cloud Spanner database 'my-database', run:

```
gcloud spanner databases update my-database --no-enable-drop-protection
```

To update KMS key references for a Cloud Spanner database 'my-database', run:

```
gcloud spanner databases update my-database --kms-keys="KEY1,KEY2"
```

To remove all KMS key references and revert a Cloud Spanner database 'my-database' to Google-managed encryption, run:

```
gcloud spanner databases update my-database --clear-kms-keys
```

POSITIONAL ARGUMENTS

Database resource - The Cloud Spanner database to update. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

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

FLAGS

`--async`

Return immediately, without waiting for the operation in progress to complete.

At most one of these can be specified:

`--clear-kms-keys`  
Removes all KMS key references and reverts the database to Google-managed encryption.

`--[no-]enable-drop-protection`  
Enable database deletion protection on this database. Use `--enable-drop-protection` to enable and `--no-enable-drop-protection` to disable.

`--kms-keys` = `KMS_KEY` ,\[ `KMS_KEY` ,…\]  
Update KMS key references for this database. Users should always provide the full set of required KMS key references.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner databases update
```

```
gcloud beta spanner databases update
```
