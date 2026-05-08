> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes how to use the Spanner Python client library to connect to a Spanner Omni deployment.

The Spanner Python client supports three security configurations:

  - [Plain text](https://docs.cloud.google.com/spanner-omni/python#plain-text)

  - [Transport Layer Security (TLS)](https://docs.cloud.google.com/spanner-omni/python#tls)

  - [Mutual TLS (mTLS)](https://docs.cloud.google.com/spanner-omni/python#mtls)

You implement these configurations by defining specific client options during initialization.

The Preview version of Spanner Omni supports unencrypted deployments. To get the features that let you create deployments with encryption, [contact Google](https://cloud.google.com/consulting/spanner-omni) to request early access to the full version of Spanner Omni.

## Prerequisites

To use Spanner Omni in a Python environment, ensure the [Python client library](https://docs.cloud.google.com/python/docs/reference/spanner/latest) is version 3.65.0 or later.

## Initialize the Spanner Omni client

To initialize a connection to Spanner Omni, specify the host and security options.

### Use plain text

To initialize a connection using plain text communication, run the following commands:

    from google.cloud import spanner
    
    spanner_client = spanner.Client(
        experimental_host="OMNI_ENDPOINT:PORT",
        use_plain_text=True
    )

### Use TLS

To establish a secure TLS connection, run the following commands:

    from google.cloud import spanner
    
    spanner_client = spanner.Client(
        experimental_host="OMNI_ENDPOINT:PORT",
        ca_certificate="PATH_TO_CA_CERT"
    )

### Use mTLS

To configure a mutual TLS (mTLS) connection, run the following commands:

    from google.cloud import spanner
    
    spanner_client = spanner.Client(
        experimental_host="OMNI_ENDPOINT:PORT",
        ca_certificate="PATH_TO_CA_CERT",
        client_certificate="PATH_TO_CLIENT_CERT",
        client_key="PATH_TO_CLIENT_KEY"
    )

## What's next

  - [Explore graph data with the Spanner Graph notebook](https://docs.cloud.google.com/spanner-omni/graph-notebook) .
