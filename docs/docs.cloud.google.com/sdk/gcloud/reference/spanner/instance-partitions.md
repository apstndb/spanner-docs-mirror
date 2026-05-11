---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-partitions
title: gcloud spanner instance-partitions
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner instance-partitions - manage Spanner instance partitions

SYNOPSIS

`gcloud spanner instance-partitions` `  COMMAND  ` \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

Manage Spanner instance partitions.

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --help  ` .

Run ` $ gcloud help  ` for details.

COMMANDS

`  COMMAND  ` is one of the following:

  - `  create  `  
    Create a Spanner instance partition.
  - `  delete  `  
    Delete a Spanner instance partition. You can't delete the default instance partition using this command.
  - `  describe  `  
    Describe a Spanner instance partition.
  - `  list  `  
    List the Spanner instance partitions contained within the given instance.
  - `  update  `  
    Update a Spanner instance partition. You can't update the default instance partition using this command.

NOTES

These variants are also available:

    gcloud alpha spanner instance-partitions

    gcloud beta spanner instance-partitions
