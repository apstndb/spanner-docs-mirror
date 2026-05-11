---
name: documents/docs.cloud.google.com/spanner-omni/prometheus-alerts
uri: https://docs.cloud.google.com/spanner-omni/prometheus-alerts
title: Use Prometheus alerts to monitor Spanner Omni
description: Learn about the Prometheus alerts available for Spanner Omni and how to use them to monitor your deployment.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes the Prometheus alerts available for Spanner Omni. Use these alerts to monitor the status and performance of your Spanner Omni deployment.

## TrueTime alerts

Use the following alerts to monitor the status of TrueTime in your deployment:

| Alert                 | Severity | Duration | Description                                                    |
| --------------------- | -------- | -------- | -------------------------------------------------------------- |
| `TrueTimeUnavailable` | critical | 1 minute | TrueTime is unavailable for more than 1 minute.                |
| `ClockSlaViolation`   | critical | 1 minute | A server has violated the clock service level agreement (SLA). |

## CPU alerts

Use the following alerts to monitor the CPU utilization of your deployment:

| Alert                       | Severity | Duration  | Description                                                               |
| --------------------------- | -------- | --------- | ------------------------------------------------------------------------- |
| `SpannerHighCPUUtilization` | warning  | 5 minutes | Overall CPU utilization has been higher than 65% for more than 5 minutes. |

## Storage alerts

Use the following alerts to monitor the storage utilization of your deployment:

| Alert                               | Severity | Duration  | Description                                     |
| ----------------------------------- | -------- | --------- | ----------------------------------------------- |
| `SpannerStorageUtilizationWarning`  | warning  | 5 minutes | Spanner Omni storage on a server is high (80%). |
| `SpannerStorageUtilizationCritical` | critical | 5 minutes | Spanner Omni storage on a server is high (90%). |
| `SpannerStoragePerVCPUTooHigh`      | warning  | 5 minutes | The storage per vCPU is exceeding 500 GB.       |

## What's next

  - [Use Grafana dashboards to monitor Spanner Omni](https://docs.cloud.google.com/spanner-omni/grafana-dashboards) .

  - [Add encryption to a Spanner Omni Kubernetes deployment](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes) .

  - [Add encryption to a Spanner Omni VM deployment](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms) .
