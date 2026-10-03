---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update
title: gcloud spanner instance-configs update
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner instance-configs update - update a Cloud Spanner instance configuration

SYNOPSIS

`gcloud spanner instance-configs update` [`INSTANCE_CONFIG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#INSTANCE_CONFIG) \[ [`--async`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#--async) \] \[ [`--display-name`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#--display-name) = `DISPLAY_NAME` \] \[ [`--etag`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#--etag) = `ETAG` \] \[ [`--update-labels`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#--update-labels) =\[ `KEY` = `VALUE` , …\]\] \[ [`--validate-only`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#--validate-only) \] \[ [`--clear-labels`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#--clear-labels) \| [`--remove-labels`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#--remove-labels) =\[ `KEY` , …\]\] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/update#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Update a Cloud Spanner instance configuration.

EXAMPLES

To update display name of a custom Cloud Spanner instance configuration 'custom-instance-config', run:

```
gcloud spanner instance-configs update custom-instance-config --display-name=nam3-RO-us-central1
```

To modify the instance config 'custom-instance-config' by adding label 'k0', with value 'value1' and label 'k1' with value 'value2' and removing labels with key 'k3', run:

```
gcloud spanner instance-configs update custom-instance-config --update-labels=k0=value1,k1=value2 --remove-labels=k3
```

To clear all labels of a custom Cloud Spanner instance configuration 'custom-instance-config', run:

```
gcloud spanner instance-configs update custom-instance-config --clear-labels
```

To remove an existing label of a custom Cloud Spanner instance configuration 'custom-instance-config', run:

```
gcloud spanner instance-configs update custom-instance-config --remove-labels=KEY1,KEY2
```

POSITIONAL ARGUMENTS

`INSTANCE_CONFIG`  
Cloud Spanner instance config. The 'custom-' prefix is required to avoid name conflicts with Google-managed configurations.

FLAGS

`--async`

Return immediately, without waiting for the operation in progress to complete.

`--display-name` = `DISPLAY_NAME`

The name of this instance configuration as it appears in UIs.

`--etag` = `ETAG`

Used for optimistic concurrency control.

`--update-labels` =\[ `KEY` = `VALUE` ,…\]

List of label KEY=VALUE pairs to update. If a label exists, its value is modified. Otherwise, a new label is created.

Keys must start with a lowercase character and contain only hyphens ( `-` ), underscores ( `_` ), lowercase characters, and numbers. Values must contain only hyphens ( `-` ), underscores ( `_` ), lowercase characters, and numbers.

`--validate-only`

Use this flag to validate that the request will succeed before executing it.

At most one of these can be specified:

`--clear-labels`  
Remove all labels. If `--update-labels` is also specified then `--clear-labels` is applied first.

For example, to remove all labels:

```
gcloud spanner instance-configs update --clear-labels
```

To remove all existing labels and create two new labels, `foo` and `baz` :

```
gcloud spanner instance-configs update --clear-labels --update-labels foo=bar,baz=qux
```

`--remove-labels` =\[ `KEY` ,…\]  
List of label keys to remove. If a label does not exist it is silently ignored. If `--update-labels` is also specified then `--update-labels` is applied first.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner instance-configs update
```

```
gcloud beta spanner instance-configs update
```
