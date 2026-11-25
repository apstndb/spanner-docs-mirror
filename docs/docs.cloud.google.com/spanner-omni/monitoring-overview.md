---
name: documents/docs.cloud.google.com/spanner-omni/monitoring-overview
uri: https://docs.cloud.google.com/spanner-omni/monitoring-overview
title: Monitoring overview
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

Monitoring in Spanner Omni works similarly to monitoring in Spanner, with some exceptions. This document provides explanations of those exceptions and points you to related information in Spanner monitoring documentation. As you read the Spanner documentation, refer back to this page to confirm how Spanner Omni differs.

## Spanner features that aren't supported

Spanner Omni supports all Spanner monitoring features except [end-to-end tracing](https://docs.cloud.google.com/spanner/docs/tracing-overview#end-to-end-side-tracing) .

## Capture telemetry signals

To gain insights into performance, usage, and potential issues, you can capture signals from your Spanner Omni database.

### Signal capture overview

Telemetry signals help you understand how your database is performing.

The following signals are available for Spanner Omni:

| Signal type       | Description                                        |
|-------------------|----------------------------------------------------|
| Metrics           | Numeric data about database health or performance. |
| Logs              | Generated records of system activity over time.    |
| Traces            | Request paths through your application.            |
| Statistics tables | Built-in tables for querying database information. |

In Spanner Omni, client-side metrics aren't supported. Client-side metrics include latencies for GFE metrics, client-side metric collection, and custom client-side metric collection. For more information, see [Signal capture overview](https://docs.cloud.google.com/spanner/docs/signal-capture-overview) in the Spanner documentation.

### Trace collection

Traces represent the path of a request through your application. They let you follow the flow of a request and help you identify the root cause of an issue. For example, tracing can provide information for every request from a client, such as spans with timestamps of when the client sent and received RPC requests. This helps you monitor and debug database requests.

For more information, see [Trace collection overview](https://docs.cloud.google.com/spanner/docs/tracing-overview) in the Spanner documentation.

#### Set up trace collection using OpenTelemetry

You can configure Spanner client libraries to export traces using [OpenTelemetry](https://opentelemetry.io/) APIs. This process involves configuring the OpenTelemetry SDK and using an exporter to send trace data to an observability backend.

Client-side tracing isn't available for Spanner Omni.

For more information, see [Set up trace collection using OpenTelemetry](https://docs.cloud.google.com/spanner/docs/set-up-tracing) in the Spanner documentation.

### Export server traces to a collector

Spanner Omni servers can export their own traces in OTLP format to a trace collector, such as [Jaeger](https://www.jaegertracing.io/) . To set the collector, do one of the following:

- On VMs, set the `SPANNER_TRACE_EXPORTER_ENDPOINT` environment variable before you start the servers. For more information, see [Collect and analyze traces](https://docs.cloud.google.com/spanner-omni/deploy-on-vms#collect-analyze-traces) .

- On Kubernetes, set the `deployment.traceExporterEndpoint` Helm value:

  ```
  helm upgrade spanner-omni oci://us-docker.pkg.dev/spanner-omni/charts/spanner-omni \
      --version 2.0.0 \
      --namespace NAMESPACE \
      --reuse-values \
      --set deployment.traceExporterEndpoint=dns:///COLLECTOR_HOST:COLLECTOR_PORT
  ```

  If you install the Helm chart with `monitoring.enabled=true` and don't set `deployment.traceExporterEndpoint` , servers export traces to the Jaeger collector that the Helm chart installs, at `dns:///jaeger.monitoring.svc.cluster.local:4317` .

Servers export traces to a Unix domain socket endpoint (a target that starts with `unix:` or `unix-abstract:` ) without TLS, because the traces don't leave the host. Servers treat every other collector as remote, including a collector on `localhost` , a loopback IP address, another pod or service, or the Kubernetes node's IP address (for example, `$HOST_IP:4317` for a DaemonSet collector).

#### Export traces over TLS

Starting with Spanner Omni release `2026.r4-lts.3` , servers in a deployment with TLS encryption export traces to a remote collector only over TLS. Earlier releases export traces to remote collectors in plaintext and don't support TLS trace export. To export traces to a remote collector, do the following:

1.  Create the trace collector CA ( `ca-otlp.crt` ) and a collector certificate for the collector hostname in the trace exporter endpoint:

    - On VMs, see [Create certificate authorities](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#create-ca) and [Create the trace collector certificate](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#create-collector-cert) .
    - On Kubernetes, see [Create certificate authorities](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes#create-ca) and [Create the trace collector certificate](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes#create-collector-cert) .

2.  Enable TLS on the collector's OTLP gRPC receiver with the collector certificate. The following examples use the `collector.crt` and `collector.key` files in the `otlp-certs` directory. Run the commands from the directory where you created the certificates, or replace `otlp-certs` with the full path to that directory.

    - For Jaeger v1 ( `jaeger-all-in-one` ) on a VM:

      ```
      export COLLECTOR_OTLP_ENABLED=true
      export COLLECTOR_OTLP_GRPC_TLS_ENABLED=true
      export COLLECTOR_OTLP_GRPC_TLS_CERT=otlp-certs/collector.crt
      export COLLECTOR_OTLP_GRPC_TLS_KEY=otlp-certs/collector.key
      jaeger-all-in-one
      ```

      Jaeger v2 configures TLS in its configuration file instead. For more information, see the [Jaeger security documentation](https://www.jaegertracing.io/docs/latest/security/) .

    - For the Jaeger collector that the Helm chart installs with `monitoring.enabled=true` , create a TLS secret named `jaeger-tls` in the `monitoring` namespace:

      ```
      kubectl create secret tls jaeger-tls -n monitoring \
          --cert=otlp-certs/collector.crt --key=otlp-certs/collector.key
      ```

      Starting with Helm chart version 2.0.0, the Jaeger collector serves TLS by default in deployments with TLS encryption, using the `jaeger-tls` secret. In these deployments, `helm install` and `helm upgrade` fail if the secret doesn't exist, unless you set `deployment.traceExporterEndpoint` to your own collector. To use a secret with a different name, set `monitoring.jaeger.tls.secretName` . Jaeger reloads the certificate every `monitoring.jaeger.tls.reloadInterval` (default: `1h` ), so you can rotate the certificate without a restart.

      Helm chart version 2.0.0 requires Spanner Omni release `2026.r4-lts.3` or later. With earlier releases, use Helm chart version 1.x.

3.  Distribute `ca-otlp.crt` to the servers together with the other certificates:

    - On VMs, copy it to the certificate directory of each server. For more information, see [Distribute the certificates to all servers](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms#distribute-certs) .

    - On Kubernetes, add it to the `tls-certs` secret. For more information, see [Push the certificates, wrapper keyset, and administrator password to the Kubernetes cluster](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes#push-certs) . To add it to an existing `tls-certs` secret without changing the other keys, run the following command:

      ```
      kubectl patch secret tls-certs -n NAMESPACE --type=merge \
          -p "{\"data\":{\"ca-otlp.crt\":\"$(base64 -w 0 certs/ca-otlp.crt)\"}}"
      ```

4.  Start or restart the servers. Servers decide whether to use TLS at startup, based on whether `ca-otlp.crt` exists, so adding or removing the file takes effect only after a restart. Changes to an existing file, such as a certificate rotation, take effect without a restart.

    On Kubernetes, adding a key to the `tls-certs` secret doesn't restart the servers. To restart them, run the following command:

    ```
    kubectl rollout restart statefulset -n NAMESPACE \
        -l app.kubernetes.io/instance=spanner-omni
    ```

Servers verify that the collector's certificate is issued by `ca-otlp.crt` and matches the hostname in the trace exporter endpoint. Servers don't present a client certificate, so don't configure the collector to require client certificates. If `ca-otlp.crt` exists, servers also use TLS for remote collectors in deployments without TLS encryption.

#### Monitor dropped traces

In a deployment with TLS encryption, if `ca-otlp.crt` doesn't exist, servers don't export traces to a remote collector. Servers log a rate-limited warning that explains how to fix the configuration, and count the dropped traces in the `opentelemetry_dropped_spans` Prometheus metric, with the `reason` label set to `secure_mode` .

Before you create the deployment, servers can't determine whether the deployment uses TLS encryption. Until then, if `ca-otlp.crt` doesn't exist, servers don't export traces to a remote collector and count them with the `reason` label set to `deployment_state_unavailable` .

If `ca-otlp.crt` isn't a valid CA certificate, or if the collector's certificate isn't issued by `ca-otlp.crt` or doesn't match the hostname in the trace exporter endpoint, connections to the collector fail until you fix the configuration. The `opentelemetry_dropped_spans` metric doesn't count these failures. To detect them, check the server logs for TLS handshake and trace export errors. Servers also log a warning at startup if `ca-otlp.crt` doesn't contain a PEM certificate.

### Audit logs

Audit logs track administrative changes and data access events for security and compliance. You can use audit logs to monitor request latencies by referring to the processing duration fields. For more information, see [Audit logs](https://docs.cloud.google.com/spanner-omni/audit-logs) .

## Monitor instances

Monitor your instances to ensure they are performing as expected and to help you troubleshoot potential issues.

### Monitor instance performance using insights

The system insights dashboard provides charts and metrics for latency, CPU utilization, storage, throughput, and other performance statistics. You can use this dashboard to monitor Spanner Omni instances and databases. For more information, see [Monitor instances with system insights](https://docs.cloud.google.com/spanner/docs/monitoring-console) in the Spanner documentation.

## What's next

- [Use Grafana dashboards to monitor Spanner Omni](https://docs.cloud.google.com/spanner-omni/grafana-dashboards) .

- [Use Prometheus alerts to monitor Spanner Omni](https://docs.cloud.google.com/spanner-omni/prometheus-alerts) .

- [Learn about audit logging in Spanner Omni](https://docs.cloud.google.com/spanner-omni/audit-logs) .
