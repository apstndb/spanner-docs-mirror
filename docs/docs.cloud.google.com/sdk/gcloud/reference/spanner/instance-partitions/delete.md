---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/delete
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/delete
title: gcloud spanner instance-partitions delete
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner instance-partitions delete - delete a Spanner instance partition. You can't delete the default instance partition using this command

SYNOPSIS

`gcloud spanner instance-partitions delete` ( [`INSTANCE_PARTITION`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/delete#INSTANCE_PARTITION) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/delete#--instance) = `INSTANCE` ) \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/delete#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Delete a Spanner instance partition. You can't delete the default instance partition using this command.

EXAMPLES

To delete a Spanner instance partition, run:

```
gcloud spanner instance-partitions delete my-instance-partition-id --instance=my-instance-id
```

POSITIONAL ARGUMENTS

Instance partition resource - The Spanner instance partition to delete. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

To set the `project` attribute:

- provide the argument `instance_partition` on the command line with a fully specified name;
- provide the argument `--project` on the command line;
- set the property `core/project` .

This must be specified.

`INSTANCE_PARTITION`  
ID of the instance partition or fully qualified identifier for the instance partition.

To set the `instance partition` attribute:

- provide the argument `instance_partition` on the command line.

This positional argument must be specified if any of the other arguments in this group are specified.

`--instance` = `INSTANCE`  
The Cloud Spanner instance for the instance partition.

To set the `instance` attribute:

- provide the argument `instance_partition` on the command line with a fully specified name;
- provide the argument `--instance` on the command line;
- set the property `spanner/instance` .

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner instance-partitions delete
```

```
gcloud beta spanner instance-partitions delete
```
