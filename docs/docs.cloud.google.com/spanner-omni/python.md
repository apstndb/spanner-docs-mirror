---
name: documents/docs.cloud.google.com/spanner-omni/python
uri: https://docs.cloud.google.com/spanner-omni/python
title: Use the Python client library to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

The Python client library for Spanner works with Spanner Omni in the same way it works with Spanner. This document shows you how to establish secure connections to Spanner Omni by configuring the Python client library. You establish these connections by setting client options when you create a client.

The Python client library supports plain text, TLS, TLS with credentials, and mTLS connections.

For more information, see [Get started with Spanner in Python](https://docs.cloud.google.com/spanner/docs/getting-started/python) in the Spanner documentation.

## Before you begin

To use the Python client library with Spanner Omni, use the [Python client library](https://docs.cloud.google.com/python/docs/reference/spanner/latest) version 3.72.0 or later.

To install the Spanner Omni Python package, run the following command:

    pip install google-cloud-spanner>=3.72.0

## Configure the `Client` object

When configuring the [`Client`](https://docs.cloud.google.com/python/docs/reference/spanner/latest/google.cloud.spanner_v1.client.Client) object, specify the Spanner Omni endpoint in `client_options` and specify `instance_type=InstanceType.OMNI` .

The following examples show how to configure the `Client` object for each supported security configuration:

### Plain text

To establish a plain-text connection, specify `instance_type=InstanceType.OMNI` and `use_plain_text=True` :

    from google.cloud import spanner
    from google.cloud.spanner_v1 import InstanceType
    
    spanner_client = spanner.Client(
        client_options={"api_endpoint": "ENDPOINT"},
        instance_type=InstanceType.OMNI,
        use_plain_text=True,
    )

Replace the following:

  - `  ENDPOINT  ` : the endpoint of your Spanner Omni instance.

### TLS

To establish a TLS connection, specify the path to your CA certificate using `ca_certificate` :

    from google.cloud import spanner
    from google.cloud.spanner_v1 import InstanceType
    
    spanner_client = spanner.Client(
        client_options={"api_endpoint": "ENDPOINT"},
        instance_type=InstanceType.OMNI,
        ca_certificate="PATH_TO_CA_CERT",
    )

Replace the following:

  - `  ENDPOINT  ` : the endpoint of your Spanner Omni instance.

  - `  PATH_TO_CA_CERT  ` : the path to your CA certificate file.

### TLS with credentials

To establish a TLS connection with username and password authentication, specify the `username` and `password` parameters alongside the CA certificate:

    from google.cloud import spanner
    from google.cloud.spanner_v1 import InstanceType
    
    spanner_client = spanner.Client(
        client_options={"api_endpoint": "ENDPOINT"},
        instance_type=InstanceType.OMNI,
        ca_certificate="PATH_TO_CA_CERT",
        username="USERNAME",
        password="PASSWORD",
    )

Replace the following:

  - `  ENDPOINT  ` : the endpoint of your Spanner Omni instance.

  - `  PATH_TO_CA_CERT  ` : the path to your CA certificate file.

  - `  USERNAME  ` : the username for your Spanner Omni user.

  - `  PASSWORD  ` : the password for your Spanner Omni user.

### mTLS

To establish a mutual TLS (mTLS) connection, specify the CA certificate, client certificate, and private client key:

    from google.cloud import spanner
    from google.cloud.spanner_v1 import InstanceType
    
    spanner_client = spanner.Client(
        client_options={"api_endpoint": "ENDPOINT"},
        instance_type=InstanceType.OMNI,
        ca_certificate="PATH_TO_CA_CERT",
        client_certificate="PATH_TO_CLIENT_CERT",
        client_key="PATH_TO_CLIENT_KEY",
    )

Replace the following:

  - `  ENDPOINT  ` : the endpoint of your Spanner Omni instance.

  - `  PATH_TO_CA_CERT  ` : the path to your CA certificate file.

  - `  PATH_TO_CLIENT_CERT  ` : the path to your client certificate file.

  - `  PATH_TO_CLIENT_KEY  ` : the path to your client private key file.

## Get a database

After you configure the `Client` object, you can get a database. Because Spanner Omni does not use Google Cloud project or instance IDs, specify `default` for the instance ID when you create an `Instance` :

    instance = spanner_client.instance("default")
    database = instance.database("DATABASE_ID")

Replace the following:

  - `  DATABASE_ID  ` : the ID of your Spanner Omni database.
