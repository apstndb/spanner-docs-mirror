---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update
title: gcloud spanner databases ddl update
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner databases ddl update - update the DDL for a Cloud Spanner database

SYNOPSIS

`gcloud spanner databases ddl update` ( [`DATABASE`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update#DATABASE) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update#--instance) = `INSTANCE` ) \[ [`--async`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update#--async) \] \[ [`--ddl`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update#--ddl) = `DDL` \] \[ [`--ddl-file`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update#--ddl-file) = `DDL_FILE` \] \[ [`--proto-descriptors-file`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update#--proto-descriptors-file) = `PROTO_DESCRIPTORS_FILE` \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Update the DDL for a Cloud Spanner database.

EXAMPLES

To add a column to a table in the given Cloud Spanner database, run:

```
gcloud spanner databases ddl update my-database-id --instance=my-instance-id --ddl='ALTER TABLE test_table ADD COLUMN a INT64'
```

POSITIONAL ARGUMENTS

Database resource - The Cloud Spanner database of which the ddl to update. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

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

`--ddl` = `DDL`  
Semi-colon separated DDL (data definition language) statements to run inside the database. If a statement fails, all subsequent statements in the batch are automatically cancelled.

`--ddl-file` = `DDL_FILE`  
Path of a file containing semi-colon separated DDL (data definition language) statements to run inside the database. If a statement fails, all subsequent statements in the batch are automatically cancelled. If --ddl_file is set, --ddl is ignored. One line comments starting with -- are ignored.

`--proto-descriptors-file` = `PROTO_DESCRIPTORS_FILE`  
Path of a file that contains a protobuf-serialized google.protobuf.FileDescriptorSet message. To generate it, install and run `protoc` with --include_imports and --descriptor_set_out.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner databases ddl update
```

```
gcloud beta spanner databases ddl update
```
