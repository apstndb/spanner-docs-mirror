---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete
title: gcloud spanner databases sessions delete
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner databases sessions delete - delete a Cloud Spanner session

SYNOPSIS

`gcloud spanner databases sessions delete` ( [`SESSION`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete#SESSION) : [`--database`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete#--database) = `DATABASE` [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete#--instance) = `INSTANCE` ) \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Delete a Cloud Spanner session.

EXAMPLES

To delete a Cloud Spanner session, run:

```
gcloud spanner databases sessions delete my-session-id --instance=my-instance-id --database=my-database-id
```

POSITIONAL ARGUMENTS

Session resource - The Cloud Spanner session to delete. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

To set the `project` attribute:

- provide the argument `session` on the command line with a fully specified name;
- provide the argument `--project` on the command line;
- set the property `core/project` .

This must be specified.

`SESSION`  
ID of the session or fully qualified identifier for the session.

To set the `session` attribute:

- provide the argument `session` on the command line.

This positional argument must be specified if any of the other arguments in this group are specified.

`--database` = `DATABASE`  
The Cloud Spanner database for the session.

To set the `database` attribute:

- provide the argument `session` on the command line with a fully specified name;
- provide the argument `--database` on the command line.

`--instance` = `INSTANCE`  
The Cloud Spanner instance for the session.

To set the `instance` attribute:

- provide the argument `session` on the command line with a fully specified name;
- provide the argument `--instance` on the command line;
- set the property `spanner/instance` .

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner databases sessions delete
```

```
gcloud beta spanner databases sessions delete
```
