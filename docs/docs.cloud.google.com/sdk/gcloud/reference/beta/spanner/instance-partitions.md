---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/beta/spanner/instance-partitions
uri: https://docs.cloud.google.com/sdk/gcloud/reference/beta/spanner/instance-partitions
title: gcloud beta spanner instance-partitions
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
update_time: "2026-02-18T19:35:46Z"
---

NAME

gcloud beta spanner instance-partitions - manage Spanner instance partitions

SYNOPSIS

`gcloud beta spanner instance-partitions` `  COMMAND  ` \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

`(BETA)` Manage Spanner instance partitions.

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --help  ` .

Run ` $ gcloud help  ` for details.

COMMANDS

`  COMMAND  ` is one of the following:

  - `  create  `  
    `(BETA)` Create a Spanner instance partition.
  - `  delete  `  
    `(BETA)` Delete a Spanner instance partition. You can't delete the default instance partition using this command.
  - `  describe  `  
    `(BETA)` Describe a Spanner instance partition.
  - `  list  `  
    `(BETA)` List the Spanner instance partitions contained within the given instance.
  - `  update  `  
    `(BETA)` Update a Spanner instance partition. You can't update the default instance partition using this command.

NOTES

This command is currently in beta and might change without notice. These variants are also available:

    gcloud spanner instance-partitions

    gcloud alpha spanner instance-partitions
