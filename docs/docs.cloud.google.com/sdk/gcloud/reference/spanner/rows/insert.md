---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/rows/insert
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/rows/insert
title: gcloud spanner rows insert
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner rows insert - insert a row in a Cloud Spanner database

SYNOPSIS

`gcloud spanner rows insert` [`--data`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/rows/insert#--data) = \[ `COLUMN_NAME` = `VALUE` , …\] [`--table`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/rows/insert#--table) = `TABLE` ( [`--database`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/rows/insert#--database) = `DATABASE` : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/rows/insert#--instance) = `INSTANCE` ) \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/rows/insert#GCLOUD-WIDE-FLAGS)` …` \]

EXAMPLES

To insert a row with SingerId=1,SingName=abc in table Singers under my-database and my-instance, run:

```
gcloud spanner rows insert --table=Singers --database=my-database --instance=my-instance --data=SingerId=1,SingerName=abc
```

```
gcloud spanner rows insert --table=Singers --database=my-database --instance=my-instance --flags-file=path/to/file.yaml
```

REQUIRED FLAGS

`--data` =\[ `COLUMN_NAME` = `VALUE` ,…\]

The column names and values for the row being added. For complicated input values, such as arrays, use the `--flags-file` flag. See \$ [gcloud topic flags-file](https://docs.cloud.google.com/sdk/gcloud/reference/topic/flags-file) for more information.

`--table` = `TABLE`

The Cloud Spanner table name.

Database resource - The Cloud Spanner database in which to insert a row. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

To set the `project` attribute:

- provide the argument `--database` on the command line with a fully specified name;
- provide the argument `--project` on the command line;
- set the property `core/project` .

This must be specified.

`--database` = `DATABASE`  
ID of the database or fully qualified identifier for the database.

To set the `database` attribute:

- provide the argument `--database` on the command line.

This flag argument must be specified if any of the other arguments in this group are specified.

`--instance` = `INSTANCE`  
The Cloud Spanner instance for the database.

To set the `instance` attribute:

- provide the argument `--database` on the command line with a fully specified name;
- provide the argument `--instance` on the command line;
- set the property `spanner/instance` .

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner rows insert
```

```
gcloud beta spanner rows insert
```
