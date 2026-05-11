---
name: documents/docs.cloud.google.com/spanner-omni/graph-notebook
uri: https://docs.cloud.google.com/spanner-omni/graph-notebook
title: Explore graph data with the Spanner Graph notebook
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

The Spanner Graph notebook lets you explore your data visually. Using [Graph Query Language](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) (GQL) query syntax, you can extract graph insights and relationship patterns, including node and edge properties and neighbor expansion analysis. The tool also provides graph schema metadata visualization, tabular results inspection, and diverse layout topologies.

You can use the notebook to connect to a Spanner Omni deployment. The Spanner Graph notebook supports three security configurations:

  - [Plain text](https://docs.cloud.google.com/spanner-omni/graph-notebook#plain-text)

  - [Transport Layer Security (TLS)](https://docs.cloud.google.com/spanner-omni/graph-notebook#tls)

  - [Mutual TLS (mTLS)](https://docs.cloud.google.com/spanner-omni/graph-notebook#mtls)

The Preview version of Spanner Omni supports unencrypted deployments. To get the features that let you create deployments with encryption, [contact Google](https://cloud.google.com/consulting/spanner-omni) to request early access to the full version of Spanner Omni.

## Before you begin

Use the [Spanner Graph notebook](https://github.com/cloudspannerecosystem/spanner-graph-notebook) version 1.1.10 or later.

## Initialize a Spanner Graph notebook connection

To initialize a connection to the Spanner Graph notebook, run the `%%spanner_graph --database` command with the parameters for the security configuration you're using.

### Use plain text

To initialize a connection using plain text communication, run the following command:

    %%spanner_graph --database DATABASE_NAME --experimental_host OMNI_ENDPOINT:PORT --use_plain_text

### Use TLS

To establish a TLS connection, run the following command:

    %%spanner_graph --database DATABASE_NAME --experimental_host OMNI_ENDPOINT:PORT --ca_certificate PATH_TO_CA_CERT

### Use mTLS

To configure a mutual TLS (mTLS) connection, run the following command:

    %%spanner_graph --database DATABASE_NAME --experimental_host OMNI_ENDPOINT:PORT --ca_certificate PATH_TO_CA_CERT --client_certificate PATH_TO_CLIENT_CERT --client_key PATH_TO_CLIENT_KEY

## What's next

  - [Use the Python client library with Spanner Omni](https://docs.cloud.google.com/spanner-omni/python) .

  - [Learn about using Spanner Graph with Spanner Omni](https://docs.cloud.google.com/spanner-omni/spanner-graph-overview) .
