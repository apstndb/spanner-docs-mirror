---
name: documents/docs.cloud.google.com/spanner-omni/install-manage-license
uri: https://docs.cloud.google.com/spanner-omni/install-manage-license
title: Install and manage a Spanner Omni license
description: Learn how to store, install, update, and verify a Spanner Omni license key in VM and Kubernetes deployments.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document explains how you store and handle your license key, install it in your deployment, update expiring keys, and verify the installation across all nodes.

To use advanced features in Spanner Omni—such as TLS/mTLS encryption, audit logging, and backup and restore—or to run a production environment, you must install a Spanner Omni license key.

To learn about available license types, editions, and features, see [Spanner Omni editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview#editions-comparison) .

## Store your license key

Treat your Spanner Omni license key as a highly sensitive credential. If someone compromises a license key, Google invalidates it in future Spanner Omni releases. To prevent key exposure, implement the following lifecycle guidelines. We recommend that you use automated systems to inject the key rather than writing it to persistent storage in plaintext. The following sections describe how you store your license key, protect automated deployment files, and configure runtime transmission.

### Storage in central secret managers

Don't commit your license key to source control or hardcode it in configuration files. Always store the key in a centralized credential store, such as:

  - **Cloud:** Google Cloud [Secret Manager](https://docs.cloud.google.com/secret-manager/docs/overview) , Amazon Web Services (AWS) [Secrets Manager](https://aws.amazon.com/secrets-manager/) , or Microsoft's [Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault) .
  - **Platform agnostic:** [HashiCorp Vault](https://www.hashicorp.com/en/products/vault) .

### Deployment automation

When you deploy through automation tools like Terraform or Ansible, don't pass the license key as plain text configuration variables:

  - **Terraform:** Fetch the license key dynamically at deployment runtime using data sources. Mark the variable using the `sensitive = true` attribute to prevent the key from appearing in execution console logs.
  - **Ansible:** Retain the key in memory during configuration execution through secret manager plugins (such as `google.cloud.gcp_secret_manager` or `community.hashicorp.vault` ), or encrypt the key with Ansible Vault if it's stored in a repository.

### Transmit your license key at runtime

Use the following techniques to deliver the license key to database processes without exposing it to unauthorized users:

#### VM-based deployments (cloud or on-premises)

  - **IAM-based injection:** Associate your database VMs with a managed service account or IAM role. During startup or provisioning (for example, with Ansible), use this identity to retrieve the license key from your secret manager into memory or a directory with restricted access.
  - **Operating system permissions:** If you write the key to disk, restrict file access so only the user running the Spanner Omni process can view the file (for example, `chmod 400 /path/to/license` ). Severely restrict SSH access to the host machines.

#### Kubernetes deployments

  - **Use the Secrets Store CSI Driver:** Use the standard [Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/) to mount your license key directly from your external secret manager into Spanner Omni pods as a temporary memory volume ( `tmpfs` ). The credential exists only in memory and disappears when the pod terminates.
  - **Avoid built-in Kubernetes Secrets:** Do not use built-in Kubernetes Secrets, which only use base64 encoding and persist in etcd.

## Install the license key

To install or update a license key, make the key accessible to your Spanner Omni servers as a local path (for example, `/path/to/your/license` ).

To pass the path of the license key to the server, use the `--license-file-path` flag or the `SPANNER_LICENSE_FILE_PATH` environment variable when running the `spanner start` or `spanner start-single-server` command.

For example:

    spanner start --root \
      --server-address=RESOLVABLE_HOSTNAME \
      --zone=ZONE_NAME \
      --base-dir=SPANNER_BASE_DIR \
      --license-file-path=LICENSE_KEY_FILE_PATH

Or, use the environment variable to supply the license path:

    SPANNER_LICENSE_FILE_PATH=LICENSE_KEY_FILE_PATH \
      spanner start --root \
        --server-address=RESOLVABLE_HOSTNAME \
        --zone=ZONE_NAME \
        --base-dir=SPANNER_BASE_DIR

Replace the following:

  - `  RESOLVABLE_HOSTNAME  ` : The resolvable hostname or IP address of the node server.
  - `  ZONE_NAME  ` : The zone name for the deployment.
  - `  SPANNER_BASE_DIR  ` : The base directory where the server files are stored.
  - `  LICENSE_KEY_FILE_PATH  ` : The local path to your license key file.

Alternatively, you can place the license key in the path where the server expects it: `BASE_DIR/license/license` . For Kubernetes deployments, `BASE_DIR` defaults to `/spanner` .

### Update an expiring license

To update or replace an expiring license across an active deployment, supply the path to the new license and initiate a *rolling restart* of your Spanner Omni servers:

1.  Make the new license file accessible to each node.
2.  Perform a rolling restart by restarting one server node at a time.
3.  Monitor deployment health during the rollout. If any node fails to restart or encounters errors, roll back to the previously working configuration and inspect your server logs to ensure the new license is provided correctly.

### Install the license key with Helm (Kubernetes)

If you deploy using the Google-provided Helm chart, you can pass the path to the license file dynamically during upgrades.

Because Helm's `--set-file` flag requires a physical path on disk, don't save the license key to a permanent file. Instead, fetch the key from your secret manager into an ephemeral temporary file or an in-memory path (for example, `/dev/shm` on Linux). Pass this path to Helm, and immediately delete or shred the file afterward. This ensures the system doesn't leave the plaintext credential on disk, preventing it from persisting locally or being accidentally committed to version control.

    helm upgrade --install spanner-omni \
      --set-file licenseKey=LICENSE_KEY_FILE_PATH

Replace the following:

  - `  LICENSE_KEY_FILE_PATH  ` : The local path to your license key file.

## Verify license installation

Because you install the license key on every active server node in a cluster, you can verify the installation by inspecting the status of one of the server nodes that handles your requests. Use the `describe` command for the endpoint:

    spanner deployment describe --deployment-endpoint=SERVER_ADDRESS

Replace the following:

  - `  SERVER_ADDRESS  ` : The address of the server node.

## What's next

  - [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms)
  - [Create a deployment on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-on-kubernetes)
  - [CLI quickstart](https://docs.cloud.google.com/spanner-omni/cli-quickstart)
