---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create
title: gcloud spanner instance-configs create
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner instance-configs create - create a Cloud Spanner instance configuration

SYNOPSIS

`gcloud spanner instance-configs create` [`INSTANCE_CONFIG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#INSTANCE_CONFIG) ( [`--base-config`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--base-config) = `BASE_CONFIG` [`--replicas`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--replicas) = `location` = `LOCATION` , `type` = `TYPE` :\[…\] \| \[ [`--clone-config`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--clone-config) = `INSTANCE_CONFIG` : [`--add-replicas`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--add-replicas) = `location` = `LOCATION` , `type` = `TYPE` :\[…\] [`--skip-replicas`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--skip-replicas) = `location` = `LOCATION` , `type` = `TYPE` :\[…\]\]) \[ [`--async`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--async) \] \[ [`--display-name`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--display-name) = `DISPLAY_NAME` \] \[ [`--etag`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--etag) = `ETAG` \] \[ [`--labels`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--labels) =\[ `KEY` = `VALUE` , …\]\] \[ [`--validate-only`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#--validate-only) \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/create#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Create a Cloud Spanner instance configuration.

EXAMPLES

To create a custom Cloud Spanner instance configuration based on an existing Google-managed configuration ( `nam3` ) by adding a `READ_ONLY` type replica in location `us-east4` , run:

```
gcloud spanner instance-configs create custom-instance-config --clone-config=nam3 --add-replicas=location=us-east4,type=READ_ONLY
```

To create a custom Cloud Spanner instance configuration based on another custom configuration ( `custom-instance-config` ) by adding a `READ_ONLY` type replica in location `us-east1` and removing a `READ_ONLY` type replica in location `us-east4` , run:

```
gcloud spanner instance-configs create custom-instance-config1 --clone-config=custom-instance-config --add-replicas=location=us-east1,type=READ_ONLY --skip-replicas=location=us-east4,type=READ_ONLY
```

POSITIONAL ARGUMENTS

`INSTANCE_CONFIG`  
Cloud Spanner instance configuration. The 'custom-' prefix is required to avoid name conflicts with Google-managed configurations.

REQUIRED FLAGS

Exactly one of these must be specified:

Command-line flags to setup a custom instance configuration replicas:  
`--base-config` = `BASE_CONFIG`  
The name of the Google-managed instance configuration, based on which your custom configuration is created.

This flag argument must be specified if any of the other arguments in this group are specified.

`--replicas` = `location` = `LOCATION` , `type` = `TYPE` :\[…\]  
The geographic placement of nodes in this instance configuration and their replication types.

`location`  
The location of the serving resources, e.g. "us-central1".

`type`  
The type of replica.

Items in the list are separated by ":". The allowed values and formats are as follows.

`READ_ONLY`  
Read-only replicas only support reads (not writes). Read-only replicas:

- Maintain a full copy of your data.

- Serve reads.

- Do not participate in voting to commit writes.

- Are not eligible to become a leader.

`READ_WRITE`  
Read-write replicas support both reads and writes. These replicas:

- Maintain a full copy of your data.

- Serve reads.

- Can vote whether to commit a write.

- Participate in leadership election.

- Are eligible to become a leader.

`WITNESS`  
Witness replicas don't support reads but do participate in voting to commit writes. Witness replicas:

- Do not maintain a full copy of data.

- Do not serve reads.

- Vote whether to commit writes.

- Participate in leader election but are not eligible to become leader.

This flag argument must be specified if any of the other arguments in this group are specified.

Command-line flags to setup a custom instance configuration using clone options:  
`--clone-config` = `INSTANCE_CONFIG`  
The ID of the instance config, based on which this configuration is created. The clone is an independent copy of this config. Available configurations can be found by running "gcloud spanner instance-configs list"

This flag argument must be specified if any of the other arguments in this group are specified.

`--add-replicas` = `location` = `LOCATION` , `type` = `TYPE` :\[…\]  
Add new replicas while cloning from the source config.

`--skip-replicas` = `location` = `LOCATION` , `type` = `TYPE` :\[…\]  
Skip replicas from the source config while cloning. Each replica in the list must exist in the source config replicas list.

OPTIONAL FLAGS

`--async`  
Return immediately, without waiting for the operation in progress to complete.

`--display-name` = `DISPLAY_NAME`  
The name of this instance configuration as it appears in UIs. Must specify this option if creating an instance-config with --replicas.

`--etag` = `ETAG`  
Used for optimistic concurrency control.

`--labels` =\[ `KEY` = `VALUE` ,…\]  
List of label KEY=VALUE pairs to add.

Keys must start with a lowercase character and contain only hyphens ( `-` ), underscores ( `_` ), lowercase characters, and numbers. Values must contain only hyphens ( `-` ), underscores ( `_` ), lowercase characters, and numbers.

`--validate-only`  
If specified, validate that the creation will succeed without creating the instance configuration.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner instance-configs create
```

```
gcloud beta spanner instance-configs create
```
