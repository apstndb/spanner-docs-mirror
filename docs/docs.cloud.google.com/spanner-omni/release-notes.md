---
name: documents/docs.cloud.google.com/spanner-omni/release-notes
uri: https://docs.cloud.google.com/spanner-omni/release-notes
title: Spanner Omni release notes
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

This page documents production updates to Spanner Omni. Check this page for announcements about new or updated features, bug fixes, known issues, and deprecated functionality.

You can see the latest product updates for all of Google Cloud on the [Google Cloud](https://docs.cloud.google.com/release-notes) page, browse and filter all release notes in the [Google Cloud console](https://console.cloud.google.com/release-notes) , or programmatically access release notes in [BigQuery](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=google_cloud_release_notes&t=release_notes&page=table) .

## November 20, 2026

Breaking

Starting with Spanner Omni patch release `2026.r4-lts.3` , servers in a deployment with TLS encryption export traces to a remote trace collector only over TLS. Previously, servers sent traces to remote collectors in plaintext.

This change affects deployments with TLS encryption that export traces to a collector that isn't reached through a Unix domain socket. This includes collectors on `localhost` , a loopback IP address, or the Kubernetes node IP address, and the Jaeger collector that the Helm chart installs with `monitoring.enabled=true` . On VMs, you set the collector with the `SPANNER_TRACE_EXPORTER_ENDPOINT` environment variable. On Kubernetes, you set it with the `deployment.traceExporterEndpoint` Helm value, or the Helm chart sets it to its Jaeger collector. These deployments stop exporting traces, and the `opentelemetry_dropped_spans` metric counts the dropped traces.

To continue exporting traces, enable TLS on the collector, add the CA certificate that issued the collector's certificate as `ca-otlp.crt` (to the server certificate directory on VMs, or to the `tls-certs` secret on Kubernetes), and restart the servers.

For the Jaeger collector that the Helm chart installs, use Helm chart version 2.0.0 or later, and create a `jaeger-tls` secret with the collector's certificate and key in the `monitoring` namespace. Helm chart version 2.0.0 is a breaking change:

- In deployments with TLS encryption, the Jaeger collector serves TLS by default, and `helm install` and `helm upgrade` fail if the `jaeger-tls` secret doesn't exist.
- When you create a new deployment with TLS encryption, `helm install` fails unless `deployment.adminPasswordSecret` names a Kubernetes secret that contains the initial `admin` user password. Upgrades of existing deployments don't need this secret.
- Helm chart version 2.0.0 requires Spanner Omni release `2026.r4-lts.3` or later. With earlier releases, use Helm chart version 1.x.

For more information, see [Export traces over TLS](https://docs.cloud.google.com/spanner-omni/monitoring-overview#export-traces-tls) and [Upgrade a deployment](https://docs.cloud.google.com/spanner-omni/upgrade-deployment) .

## October 29, 2026

Feature

Spanner Omni release `2026.r5` is available.

- Release version `2026.r5` includes all updates in the cloud-based, managed Spanner service up to the [October 27, 2026 release](https://docs.cloud.google.com/spanner/docs/release-notes#October_27_2026) .

For more information, see [Differences between Spanner and Spanner Omni](https://docs.cloud.google.com/spanner-omni/differences) and [Container image versions](https://docs.cloud.google.com/spanner-omni/download#container_image_versions) .

## October 28, 2026

Fixed

Spanner Omni patch release `2026.r4-lts.1` is available. This patch includes security and other critical bug fixes.

## September 30, 2026

Feature

Spanner Omni is generally available ( [GA](https://cloud.google.com/products#product-launch-stages) ) with long-term support (LTS) release `2026.r4-lts` . For more information, see the [Spanner Omni overview](https://docs.cloud.google.com/spanner-omni/overview) .

Release `2026.r4-lts` includes the following updates:

- Includes all Spanner changes up to the [September 18, 2026 release](https://docs.cloud.google.com/spanner/docs/release-notes#September_18_2026) , including [Spanner queues](https://docs.cloud.google.com/spanner/docs/queues/queues-overview) .

- Release `2026.r4-lts` is a long-term support (LTS) release, and Google supports this release with security fixes for up to one year. For information about upgrading from release `2026.r3-beta` , see [Upgrade a deployment](https://docs.cloud.google.com/spanner-omni/upgrade-deployment) .

- Supports approximate nearest neighbor (ANN) vector search for datasets of up to 1 billion vectors using dedicated, stateless compute workers in the [Commercial edition](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition) . For more information, see [Vector search overview](https://docs.cloud.google.com/spanner-omni/vector-search-overview#ann) and [Create and manage workers](https://docs.cloud.google.com/spanner-omni/manage-workers) .

- Supports security configurations, including TLS and mutual TLS (mTLS) encryption and authentication, in the [Developer edition](https://docs.cloud.google.com/spanner-omni/editions-overview#developer-edition) . For more information, see [Spanner Omni editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview) , [Create a deployment with TLS encryption on VMs](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms) , and [Create a deployment with TLS encryption on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes) .

- Updates the [Helm chart](https://docs.cloud.google.com/spanner-omni/create-helm-configuration) to use separate `StatefulSet` configurations for root and non-root servers. For more information, see [Create a deployment on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-on-kubernetes) and [Scale a Kubernetes deployment](https://docs.cloud.google.com/spanner-omni/scale-kubernetes-deployment) .

- Supports username and password login authentication over TLS in the [Java](https://docs.cloud.google.com/spanner-omni/java) (version 6.119.0 or later), [Go](https://docs.cloud.google.com/spanner-omni/go) (version v1.94.0 or later), and [Python](https://docs.cloud.google.com/spanner-omni/python) (version 3.72.0 or later) client libraries.

## September 17, 2026

Feature

Spanner Omni patch release `2026.r3-beta` is available.

- This release includes all the Spanner changes up to the [September 3, 2026 release](https://docs.cloud.google.com/spanner/docs/release-notes#September_03_2026) .

- You must [upgrade to this release](https://docs.cloud.google.com/spanner-omni/upgrade-deployment) before you upgrade to future releases.

## August 20, 2026

Feature

Spanner Omni release `2026.r2.1-beta` is available.

In-place upgrades from earlier versions to `2026.r2.1-beta` aren't supported. If you upgrade from an earlier release, you must migrate data from earlier deployments to a new deployment created with release `2026.r2.1-beta` . For more information, see [Restore a Spanner Omni backup](https://docs.cloud.google.com/spanner-omni/restores) and [Import and export data](https://docs.cloud.google.com/spanner-omni/import-export-data) .

Release `2026.r2.1-beta` includes the following updates:

- Two editions are available: [developer](https://docs.cloud.google.com/spanner-omni/editions-overview#developer-edition) and [commercial](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition) .For more information, see [Spanner Omni editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview) .

- A standalone server package is available to run on virtual machines. For more information, see [Download Spanner Omni](https://docs.cloud.google.com/spanner-omni/download#server_binaries) .

- Support for the `--insecure-mode` and `--enable-client-certificate-authentication` flags for the `spanner start` command has been deprecated. Specify this information when you configure your deployment. For more information, see [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms) and [Create a Helm chart configuration](https://docs.cloud.google.com/spanner-omni/create-helm-configuration) .

## June 18, 2026

Fixed

Spanner Omni patch release `2026.r1-beta.2` is available. This patch includes the following update:

- Fixed an issue related to importing a backup in regional and multi-regional deployments. For more information, see [Restore a Spanner Omni backup](https://docs.cloud.google.com/spanner-omni/restores) .

## June 04, 2026

Fixed

Spanner Omni patch release `2026.r1-beta.1` is available. This patch includes the following updates:

- Fixed a [TrueTime](https://docs.cloud.google.com/spanner-omni/true-time-external-consistency) issue in which time servers entered a repeated failover mode, causing the uncertainty window to spike.

- Enables any client with a network path to the servers to reach health check and metrics endpoints. Earlier releases restricted access to internal IP addresses.

- The following [Helm chart](https://docs.cloud.google.com/spanner-omni/create-helm-configuration) updates:

  - Added support for `nodeSelector` , `tolerations` , and job resource configurations.

  - Added an `init` container to provide permissions to `/dev/vmclock` in Amazon Web Services (AWS) deployments.

## April 22, 2026

Feature

Spanner Omni is available in [Preview](https://cloud.google.com/products?e=48754805#product-launch-stages) . Spanner Omni a self-managed version of [Spanner](https://docs.cloud.google.com/spanner/docs/overview) that you run in your own environment, such as on-premises data centers, public clouds or on your laptop. For more information, see the following:

- [Spanner Omni overview](https://docs.cloud.google.com/spanner-omni/overview) .

- [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms) .

- [Create a deployment on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-on-kubernetes) .
