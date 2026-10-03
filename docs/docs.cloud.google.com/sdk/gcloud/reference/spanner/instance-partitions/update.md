---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update
title: gcloud spanner instance-partitions update
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner instance-partitions update - update a Spanner instance partition. You can't update the default instance partition using this command

SYNOPSIS

`gcloud spanner instance-partitions update` ( [`INSTANCE_PARTITION`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#INSTANCE_PARTITION) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--instance) = `INSTANCE` ) \[ [`--async`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--async) \] \[ [`--description`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--description) = `DESCRIPTION` \] \[ [`--nodes`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--nodes) = `NODES` \| [`--processing-units`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--processing-units) = `PROCESSING_UNITS` \| [`--autoscaling-storage-target`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--autoscaling-storage-target) = `AUTOSCALING_STORAGE_TARGET` [`--autoscaling-high-priority-cpu-target`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--autoscaling-high-priority-cpu-target) = `AUTOSCALING_HIGH_PRIORITY_CPU_TARGET` [`--autoscaling-total-cpu-target`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--autoscaling-total-cpu-target) = `AUTOSCALING_TOTAL_CPU_TARGET` [`--autoscaling-max-nodes`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--autoscaling-max-nodes) = `AUTOSCALING_MAX_NODES` [`--autoscaling-min-nodes`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--autoscaling-min-nodes) = `AUTOSCALING_MIN_NODES` \| [`--autoscaling-max-processing-units`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--autoscaling-max-processing-units) = `AUTOSCALING_MAX_PROCESSING_UNITS` [`--autoscaling-min-processing-units`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#--autoscaling-min-processing-units) = `AUTOSCALING_MIN_PROCESSING_UNITS` \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions/update#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Update a Spanner instance partition. You can't update the default instance partition using this command.

EXAMPLES

To update the display name of a Spanner instance partition, run:

```
gcloud spanner instance-partitions update my-instance-partition-id --instance=my-instance-id --description=my-new-display-name
```

To update the node count of a Spanner instance partition, run:

```
gcloud spanner instance-partitions update my-instance-partition-id --instance=my-instance-id --nodes=1
```

POSITIONAL ARGUMENTS

Instance partition resource - The Spanner instance partition to update. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

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

FLAGS

`--async`

Return immediately, without waiting for the operation in progress to complete.

`--description` = `DESCRIPTION`

Description of the instance partition.

At most one of these can be specified:

`--nodes` = `NODES`

Number of nodes for the instance partition.

`--processing-units` = `PROCESSING_UNITS`

Number of processing units for the instance partition.

Or at least one of these can be specified:

Autoscaling

`--autoscaling-storage-target` = `AUTOSCALING_STORAGE_TARGET`

Specifies the target percentage of storage the autoscaled instance can utilize.

Autoscaling CPU targets.

`--autoscaling-high-priority-cpu-target` = `AUTOSCALING_HIGH_PRIORITY_CPU_TARGET`

Specifies the target percentage of high-priority CPU the autoscaled instance can utilize.

`--autoscaling-total-cpu-target` = `AUTOSCALING_TOTAL_CPU_TARGET`

Specifies the target percentage of total CPU the autoscaled instance can utilize.

Autoscaling limits can be defined in either nodes or processing units.

At most one of these can be specified:

Autoscaling limits in nodes:  
`--autoscaling-max-nodes` = `AUTOSCALING_MAX_NODES`  
Maximum number of nodes for the autoscaled instance.

`--autoscaling-min-nodes` = `AUTOSCALING_MIN_NODES`  
Minimum number of nodes for the autoscaled instance.

Autoscaling limits in processing units:  
`--autoscaling-max-processing-units` = `AUTOSCALING_MAX_PROCESSING_UNITS`  
Maximum number of processing units for the autoscaled instance.

`--autoscaling-min-processing-units` = `AUTOSCALING_MIN_PROCESSING_UNITS`  
Minimum number of processing units for the autoscaled instance.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner instance-partitions update
```

```
gcloud beta spanner instance-partitions update
```
