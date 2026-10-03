---
name: documents/docs.cloud.google.com/spanner-omni/install-manage-license
uri: https://docs.cloud.google.com/spanner-omni/install-manage-license
title: Install and manage a Spanner Omni license
description: Learn how to store, install, update, and verify a Spanner Omni license key in VM and Kubernetes deployments.
data_source: docs.cloud.google.com
---

> [Download Spanner Omni](https://docs.cloud.google.com/spanner-omni/download) to try it out at no charge. If you decide you want to use Spanner Omni for production use, [contact Google](https://cloud.google.com/consulting/spanner-omni) to learn about acquiring a license for the [commercial edition](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition) .

This document explains how you store and handle your license key, install it in your deployment, update expiring keys, and verify the installation across all nodes.

To run a production environment, you must install a commercial Spanner Omni license key. In a non-production environment, you can use backup and restore features by deploying Spanner Omni in a single-server configuration of four vCPUs or fewer without a license key. The workers feature isn't supported under the Default or Developer license, regardless of how many servers or vCPUs are used. To purchase a commercial license to use workers, [contact Google](https://cloud.google.com/consulting/spanner-omni) .

To learn about available license types, editions, and features, see [Spanner Omni editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview#compare-editions-licenses) .

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

- **Identity and Access Management (IAM)-based injection:** Associate your database VMs with a managed service account or IAM role. During startup or provisioning (for example, with Ansible), use this identity to retrieve the license key from your secret manager into memory or a directory with restricted access.
- **Operating system permissions:** If you write the key to disk, restrict file access so only the user running the Spanner Omni process can view the file (for example, `chmod 400 /path/to/license` ). Severely restrict SSH access to the host machines.

#### Kubernetes deployments

- **Use the Secrets Store CSI Driver:** Use the standard [Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/) to mount your license key directly from your external secret manager into Spanner Omni pods as a temporary memory volume ( `tmpfs` ). The credential exists only in memory and disappears when the pod terminates.
- **Avoid built-in Kubernetes Secrets:** Don't use built-in Kubernetes Secrets, which only use base64 encoding and persist in etcd.

## Install the license key

To install or update a license key, make the key accessible to your Spanner Omni servers as a local path (for example, `/path/to/your/license` ).

To pass the path of the license key to the server, use the `--license-file-path` flag or the `SPANNER_LICENSE_FILE_PATH` environment variable when running the `spanner start` or `spanner start-single-server` command.

For example:

```
spanner start --root \
  --license-file-path=LICENSE_KEY_FILE_PATH \
  --server-address=RESOLVABLE_HOSTNAME \
  --zone=ZONE_NAME \
  --base-dir=SPANNER_BASE_DIR
```

Or, use the environment variable to supply the license path:

```
SPANNER_LICENSE_FILE_PATH=LICENSE_KEY_FILE_PATH \
  spanner start --root \
    --server-address=RESOLVABLE_HOSTNAME \
    --zone=ZONE_NAME \
    --base-dir=SPANNER_BASE_DIR
```

Replace the following:

- `LICENSE_KEY_FILE_PATH` : The local path to your license key file.
- `RESOLVABLE_HOSTNAME` : The resolvable hostname or IP address of the node server.
- `ZONE_NAME` : The zone name for the deployment.
- `SPANNER_BASE_DIR` : The base directory where the server files are stored.

Alternatively, you can place the license key in the path where the server expects it: `BASE_DIR/license/license` . For Kubernetes deployments, `BASE_DIR` defaults to `/spanner` .

### Update an expiring license

To update, extend, or renew an expiring Developer edition license across an active deployment, [request a perpetual license](https://forms.gle/Ex9NcszwJFuHbtnB9) . To extend an expiring proof-of-concept license or purchase a commercial license, [contact Google](https://cloud.google.com/consulting/spanner-omni) . After you receive the new license, supply its path and initiate a *rolling restart* of your Spanner Omni servers:

1.  Make the new license file accessible to each node.
2.  Perform a rolling restart by restarting one server node at a time.
3.  Monitor deployment health during the rollout. If any node fails to restart or encounters errors, roll back to the previously working configuration and inspect your server logs to ensure the new license is provided correctly.

### Install the license key with Helm (Kubernetes)

If you deploy using the Google-provided Helm chart, you can pass the path to the license file dynamically during upgrades.

Because Helm's `--set-file` flag requires a physical path on disk, don't save the license key to a permanent file. Instead, fetch the key from your secret manager into an ephemeral temporary file or an in-memory path (for example, `/dev/shm` on Linux). Pass this path to Helm, and immediately delete or shred the file afterward. This ensures the system doesn't leave the plaintext credential on disk, preventing it from persisting locally or being accidentally committed to version control.

```
helm upgrade --install spanner-omni \
  --set-file licenseKey=LICENSE_KEY_FILE_PATH
```

Replace the following:

- `LICENSE_KEY_FILE_PATH` : The local path to your license key file.

## Verify license installation

Because you install the license key on every active server node in a cluster, you can verify the installation by inspecting the status of one of the server nodes that handles your requests. Use the `describe` command for the endpoint:

```
spanner deployment describe --deployment-endpoint=SERVER_ADDRESS
```

Replace the following:

- `SERVER_ADDRESS` : The address of the server node.

### Run a cluster-wide audit

To verify that all nodes across your VM cluster or Kubernetes StatefulSet have the correct license key installed, run the cluster-wide audit command:

```
spanner admin license check
```

A cluster-wide audit inspects every server in the database topology. Because of this, ensure that all node servers are online and network-accessible before you run this command.

#### Interpret the audit results

Use the following table to understand the results of the cluster-wide audit:

| Cluster state               | Output type                             | Description and action required                                                                                                                                                          |
|-----------------------------|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Uniform (expected)**      | Single license identifier               | All nodes are using the same license key and type. No further action is required.                                                                                                        |
| **Mixed (action required)** | List of license identifiers and servers | Different license keys are detected across nodes (for example, during a stalled rolling update). Investigate the listed nodes to ensure the new license file has been correctly applied. |

## Maintain Commercial license compliance

If you use a Spanner Omni [commercial license](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-license) , you must maintain compliance with your license agreement. Maintaining compliance helps prevent billing disputes and tracks and reports your cluster resource usage to Google. Complete the following steps to maintain compliance with a Spanner Omni Commercial license agreement:

1.  Run the Spanner Omni Usage Tool at the end of each billing cycle to collect cluster vCPU usage metrics:

    ```
    spanner admin license report --row-format=CSV
    ```

2.  Work with your Technical Account Manager (TAM) or Google account team to confirm your organization's preferred submission channel.

3.  Submit the resulting payload to Google by the 5th of the following month.

## What's next

- [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms)
- [Create a deployment on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-on-kubernetes)
- [CLI quickstart](https://docs.cloud.google.com/spanner-omni/cli-quickstart)
