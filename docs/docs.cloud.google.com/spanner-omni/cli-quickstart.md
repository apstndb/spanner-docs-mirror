---
name: documents/docs.cloud.google.com/spanner-omni/cli-quickstart
uri: https://docs.cloud.google.com/spanner-omni/cli-quickstart
title: Quickstart using the Spanner Omni CLI
description: Learn how to interact with and manage a Spanner Omni deployment using the Spanner Omni CLI.
data_source: docs.cloud.google.com
update_time: "2026-05-08T21:32:37Z"
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document shows you how to use the Spanner Omni command-line interface (CLI) to interact with and manage a Spanner Omni deployment. The Spanner Omni CLI provides commands for common administrative tasks and includes an interactive SQL shell for querying your database.

## Before you begin

Before you can use the Spanner Omni CLI, ensure you meet the following requirements:

1.  A Spanner Omni deployment is set up and running.
2.  The machine running the Spanner Omni CLI has network access to the load balancer of the Spanner Omni deployment or to at least one of the servers in the deployment.

## Step 1: Download and install the Spanner Omni CLI

1.  Download the Spanner Omni CLI from the `spanner-omni` Cloud Storage bucket.

2.  Extract the tar file.
    
        tar -xvf CLI_TAR_FILE
    
    This installs the Spanner Omni CLI binary, called `spanner` , in the `google/spanner/bin` directory.
    
    To run the `spanner` command, add the `google/spanner/bin` directory to your `PATH` environment variable, or use the full path to the binary in the following steps.

## Step 2: Connect to your deployment

By default, the Spanner Omni CLI attempts to connect to `localhost:15000` . To connect to your specific deployment, use the `--deployment-endpoint` flag.

For example, the following command lists the zones in your deployment:

    spanner deployment zones list \
        --deployment-endpoint=LOAD_BALANCER_IP_OR_SERVER_IP:PORT

## Step 3: Run common commands

The following are common administrative commands you can run with the Spanner Omni CLI.

### Get help

To see a list of available commands and global flags, run:

    spanner --help

### Create a database

To create a new database in your deployment, run:

    spanner databases create DATABASE_NAME \
        --deployment-endpoint=LOAD_BALANCER_IP_OR_SERVER_IP:PORT

### List all databases

To list all databases in your deployment, run:

    spanner databases list \
        --deployment-endpoint=LOAD_BALANCER_IP_OR_SERVER_IP:PORT

## Step 4: Start an interactive SQL shell session

The Spanner Omni CLI includes an interactive SQL shell for running queries. By starting the shell for a specific database with the `--database` flag, you can run SQL commands without needing to specify the database or endpoint for every query.

To start the SQL shell, run:

    spanner sql --database=DATABASE_NAME \
        --deployment-endpoint=LOAD_BALANCER_IP_OR_SERVER_IP:PORT

After the shell starts, you see the `sql>` prompt:

    spanner-cli>

To exit the shell, type `exit` .
