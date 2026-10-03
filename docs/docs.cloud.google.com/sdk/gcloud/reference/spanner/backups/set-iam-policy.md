---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/set-iam-policy
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/set-iam-policy
title: gcloud spanner backups set-iam-policy
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner backups set-iam-policy - set the IAM policy for a Cloud Spanner backup

SYNOPSIS

`gcloud spanner backups set-iam-policy` ( [`BACKUP`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/set-iam-policy#BACKUP) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/set-iam-policy#--instance) = `INSTANCE` ) [`POLICY_FILE`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/set-iam-policy#POLICY_FILE) \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/backups/set-iam-policy#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Set the IAM policy for a Cloud Spanner backup given a backup ID and a file encoded in JSON or YAML that contains the IAM policy.

EXAMPLES

The following command reads an IAM policy defined in a JSON file `policy.json` and sets it for a spanner instance with the ID `example-instance` :

```
gcloud spanner backups set-iam-policy example-backup --instance=example-instance policy.json
```

See <https://cloud.google.com/iam/docs/managing-policies> for details of the policy file format and contents.

POSITIONAL ARGUMENTS

Backup resource - The Cloud Spanner backup to set the IAM policy for. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

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

`POLICY_FILE`

Path to a local JSON or YAML formatted file containing a valid policy.

The output of the `get-iam-policy` command is a valid file, as is any JSON or YAML file conforming to the structure of a [Policy](https://cloud.google.com/iam/reference/rest/v1/Policy) .

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

API REFERENCE

This command uses the `spanner/v1` API. The full documentation for this API can be found at: <https://cloud.google.com/spanner/>

NOTES

These variants are also available:

```
gcloud alpha spanner backups set-iam-policy
```

```
gcloud beta spanner backups set-iam-policy
```
