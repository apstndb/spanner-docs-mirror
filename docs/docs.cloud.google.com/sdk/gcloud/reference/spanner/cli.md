---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/cli
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli
title: gcloud spanner cli
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner cli - an interactive shell for Spanner

SYNOPSIS

`gcloud spanner cli` ( [`DATABASE`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#DATABASE) : [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--instance) = `INSTANCE` ) \[ [`--database-role`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--database-role) = `DATABASE_ROLE` \] \[ [`--delimiter`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--delimiter) = `DELIMITER` ; default=";"\] \[ [`--directed-read`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--directed-read) = `DIRECTED_READ` \] \[ [`--execute`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--execute) = `EXECUTE` \] \[ [`--host`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--host) = `HOST` ; default="localhost"\] \[ [`--html`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--html) \] \[ [`--idle-transaction-timeout`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--idle-transaction-timeout) = `IDLE_TRANSACTION_TIMEOUT` ; default=60\] \[ [`--init-command`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--init-command) = `INIT_COMMAND` \] \[ [`--init-command-add`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--init-command-add) = `INIT_COMMAND_ADD` \] \[ [`--port`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--port) = `PORT` \] \[ [`--prompt`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--prompt) = `PROMPT` ; default="spanner-cli\> "\] \[ [`--proto-descriptor-file`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--proto-descriptor-file) = `PROTO_DESCRIPTOR_FILE` \] \[ [`--skip-column-names`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--skip-column-names) \] \[ [`--skip-system-command`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--skip-system-command) \] \[ [`--source`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--source) = `SOURCE` \] \[ [`--system-command`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--system-command) = `SYSTEM_COMMAND` ; default="ON"\] \[ [`--table`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--table) \] \[ [`--tee`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--tee) = `TEE` \] \[ [`--xml`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#--xml) \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/cli#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

An interactive shell for Spanner.

EXAMPLES

To start an interactive shell with your Spanner example database, run the following command:

```
gcloud spanner cli example-database --instance=example-instance
```

POSITIONAL ARGUMENTS

Database resource - The Cloud Spanner database to use within the interactive shell. The arguments in this group can be used to specify the attributes of this resource. (NOTE) Some attributes are not given arguments in this group but can be set in other ways.

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

`--database-role` = `DATABASE_ROLE`  
Database role user used to access the database.

`--delimiter` = `DELIMITER` ; default=";"  
Set the statement delimiter.

`--directed-read` = `DIRECTED_READ`  
Enables directed reads to provide the flexibility to route read-only transactions and single reads to a specific replica type or region (replica_location:replica_type). The replica_type is optional and can be either READ_ONLY or READ_WRITE.

`--execute` = `EXECUTE`  
Execute the statement and then exits.

`--host` = `HOST` ; default="localhost"  
Host on which Spanner server is located.

`--html`  
Show output in HTML format.

`--idle-transaction-timeout` = `IDLE_TRANSACTION_TIMEOUT` ; default=60  
Set the idle transaction timeout. The default timeout is 60 seconds.

`--init-command` = `INIT_COMMAND`  
SQL statement to execute after startup.

`--init-command-add` = `INIT_COMMAND_ADD`  
Additional SQL statement to execute after startup.

`--port` = `PORT`  
Port number that gcloud uses to connect to Spanner.

`--prompt` = `PROMPT` ; default="spanner-cli\> "  
Set the prompt to the specified format.

`--proto-descriptor-file` = `PROTO_DESCRIPTOR_FILE`  
Path of a file that contains a protobuf-serialized google.protobuf.FileDescriptorSet message to use in this invocation.

`--skip-column-names`  
Do not show column names in output.

`--skip-system-command`  
Do not allow system command.

`--source` = `SOURCE`  
Execute the statement from a file and then exits.

`--system-command` = `SYSTEM_COMMAND` ; default="ON"  
Enable or disable system commands. Default: ON. `SYSTEM_COMMAND` must be one of: `ON` , `OFF` .

`--table`  
Show output in table format.

`--tee` = `TEE`  
Append a copy of the output to a named file.

`--xml`  
Show output in XML format.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

This variant is also available:

```
gcloud alpha spanner cli
```
