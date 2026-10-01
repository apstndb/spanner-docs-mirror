---
name: documents/docs.cloud.google.com/spanner-omni/manage-workers
uri: https://docs.cloud.google.com/spanner-omni/manage-workers
title: Deploy and manage workers
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

This document explains how to deploy, scale, decommission, and monitor Spanner Omni workers on virtual machines (VMs) and Kubernetes.

Workers are dedicated, stateless compute nodes designed to offload background and resource-intensive operations from Spanner Omni servers. Workers don't host user data or participate in leader elections, transactions, or other core database activities. Unlike servers, workers aren't associated with a specific zone. Instead, workers register with a location and can run tasks for any zone in that location. Adding and removing workers is lightweight and instantaneous because workers are stateless and don't require data movement or rebalancing.

Workers are required to build vector indexes on large tables (more than 1 million rows) for approximate nearest neighbor (ANN) search queries. For more information, see [Spanner Omni vector search overview](https://docs.cloud.google.com/spanner-omni/vector-search-overview) .

Workers are available only in the [Commercial edition](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition) of Spanner Omni; the Developer edition doesn't support workers. Compute of workers is billed at the same rate as servers in the deployment (per vCPU). For more information, see the [Spanner Omni editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview) .

## Before you begin

Before you add workers to an existing Spanner Omni deployment, ensure that your environment meets the following requirements:

  - [**Download and set up the Spanner Omni binary**](https://docs.cloud.google.com/spanner-omni/deploy-on-vms#step-2-binary) .

  - **Existing deployment** : Verify that you have a running Spanner Omni deployment (not a single-server deployment) in the `READY` state, configured with the Commercial edition. The Developer edition doesn't support workers. Compute of workers is billed at the same rate as servers in the deployment. For more information, see the [Spanner Omni editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview) . Ensure that you have the following information:
    
      - The target location name (for example, `us-central1` ) as defined in your deployment configuration.
      - Either the deployment endpoint ( `  HOST : PORT  ` , such as `my-spanner-deployment:15003` ) or a list of root server addresses ( `  ROOT_HOST_1 : PORT  ` , `  ROOT_HOST_2 : PORT  ` , such as `root-server-1:15000` , `root-server-2:15000` ) for cluster discovery.

  - **System and hardware resources** : Make sure that the compute resources you allocate to the worker are sufficient to perform the required operations in an acceptable amount of time.

  - **vSphere configuration** : If you run Spanner Omni on the vSphere virtualization platform, disable virtualization of the Time Stamp Counter (TSC). Add `monitor_control.virtual_rdtsc = FALSE` to the virtual machine's `.vmx` configuration file.

  - **Network and firewall configuration** : Workers use port `15027` in addition to the standard server communication ports ( `15000` to `15025` ). Ensure that your network configuration allows communication on ports `15000` to `15027` .

## Deploy workers on VMs

To deploy workers on a virtual machine (VM), start the worker process using either the deployment endpoint or a list of root servers.

### Option A: Start using the deployment endpoint

To start a worker using the deployment endpoint, run the `spanner workers start` command:

    spanner workers start \
      --location=LOCATION_NAME \
      --address=WORKER_HOSTNAME:WORKER_PORT_BASE \
      --deployment=DEPLOYMENT_ENDPOINT \
      --base-dir=BASE_DIR \
      --license-file-path=LICENSE_FILE_PATH

Replace the following:

  - `  LOCATION_NAME  ` : The target location name—for example, `us-central1` .
  - `  WORKER_HOSTNAME  ` : The resolvable hostname or IP address of the worker VM.
  - `  WORKER_PORT_BASE  ` : The base port on which the worker is started—for example, `15000` or `20000` .
  - `  DEPLOYMENT_ENDPOINT  ` : The host and port of the deployment endpoint—for example, `my-spanner-deployment:15003` .
  - `  BASE_DIR  ` : The base directory for worker data and logs—for example, `/var/spanner` .
  - `  LICENSE_FILE_PATH  ` : The path to your Spanner Omni license file.

### Option B: Start using a list of root servers

To start a worker using a list of root servers, run the `spanner workers start` command:

    spanner workers start \
      --location=LOCATION_NAME \
      --address=WORKER_HOSTNAME:WORKER_PORT_BASE \
      --join-servers=ROOT_SERVER_1_HOST:ROOT_SERVER_PORT_BASE,\
    ROOT_SERVER_2_HOST:ROOT_SERVER_PORT_BASE \
      --base-dir=BASE_DIR \
      --license-file-path=LICENSE_FILE_PATH

Replace the following:

  - `  LOCATION_NAME  ` : The target location name—for example, `us-central1` .
  - `  WORKER_HOSTNAME  ` : The resolvable hostname or IP address of the worker VM.
  - `  WORKER_PORT_BASE  ` : The base port on which the worker is started—for example, `15000` or `20000` .
  - `  ROOT_SERVER_1_HOST  ` , `  ROOT_SERVER_2_HOST  ` : The hostnames or IP addresses of root servers in your deployment.
  - `  ROOT_SERVER_PORT_BASE  ` : The base port of the root servers—for example, `15000` .
  - `  BASE_DIR  ` : The base directory for worker data and logs—for example, `/var/spanner` .
  - `  LICENSE_FILE_PATH  ` : The path to your Spanner Omni license file.

### Configure encryption

If your Spanner Omni deployment uses TLS or mTLS encryption, configure encryption for each worker:

1.  Update your server certificate to include worker hostnames if you haven't already covered them.

2.  Copy the certificate directory containing `ca.crt` , `server.crt` , and `server.key` to the worker VM.

3.  Add the `--certificate-directory` flag when running `spanner workers start` :
    
        spanner workers start \
          --location=LOCATION_NAME \
          --address=WORKER_HOSTNAME:WORKER_PORT_BASE \
          --deployment=DEPLOYMENT_ENDPOINT \
          --base-dir=BASE_DIR \
          --certificate-directory=CERTIFICATE_DIRECTORY \
          --license-file-path=LICENSE_FILE_PATH
    
    Replace `  CERTIFICATE_DIRECTORY  ` with the directory containing `ca.crt` , `server.crt` , and `server.key` .

For more information about configuring certificates and secure deployments, see [Create a secure deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms) .

## Deploy workers on Kubernetes

In Kubernetes environments such as Google Kubernetes Engine (GKE) or Amazon Elastic Kubernetes Service (Amazon EKS), you deploy workers as part of your existing Spanner Omni Helm release in the same namespace as your cluster. The Helm chart deploys workers as a Kubernetes `StatefulSet` with a headless Service, giving each worker pod a stable network identity and [PersistentVolumeClaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PVCs), which lets root servers communicate reliably with each worker.

By default, the Helm chart schedules worker pods only on nodes labeled `spanner-role=workers` , tolerates the `spanner-role=workers:NoSchedule` taint, and runs at most one worker pod per node. Before you enable workers, add a node pool with this label and taint that has at least as many nodes as `workers.replicas` . Each node needs enough allocatable CPU and memory for one worker pod, as set by `workers.resources.cpu` and `workers.resources.memory` . Kubernetes reserves part of each node's capacity for system components, so choose nodes larger than these values. To use a different label, set `workers.nodeLabelKey` and `workers.nodeLabelValue` . To remove the label requirement, set `workers.nodeLabelKey=""` . To replace the default scheduling rules, set `workers.affinity` .

To enable workers in your existing deployment, run the `helm upgrade` command:

    helm upgrade spanner-omni HELM_CHART_PATH \
      --reuse-values \
      --set workers.enabled=true \
      --namespace NAMESPACE

Replace the following:

  - `  HELM_CHART_PATH  ` : The path to your Spanner Omni Helm chart.
  - `  NAMESPACE  ` : The Kubernetes namespace where your Spanner Omni cluster is deployed—for example, `spanner-ns` .

To let workers run on any node that has enough allocatable CPU and memory, set `workers.nodeLabelKey` to an empty string. This removes both the node label requirement and the taint toleration:

    helm upgrade spanner-omni HELM_CHART_PATH \
      --reuse-values \
      --set workers.enabled=true \
      --set workers.nodeLabelKey="" \
      --namespace NAMESPACE

Optional configuration settings include:

  - ` --set workers.replicas= WORKER_REPLICAS  ` : The number of worker replicas to deploy. The default is `1` .

  - ` --set workers.resources.cpu= CPU_CORES  ` : The CPU limit and request for each worker. The default is `6` .

  - ` --set workers.resources.memory= MEMORY_LIMIT  ` : The memory limit and request for each worker. The default is `24Gi` .

  - ` --set workers.storage.size= STORAGE_SIZE  ` : The storage capacity for each worker. The default is `20Gi` .

  - ` --set workers.storage.storageClassName= STORAGE_CLASS  ` : The storage class to use for worker storage—for example, `hyperdisk-balanced-rwo` on GKE or `aws-gp3` on Amazon EKS. The default is an empty string, which inherits the cluster default storage class.

  - ` --set workers.port= WORKER_PORT  ` : The network port the worker listens on. The default is `deployment.basePort` , which is `15000` .

  - `--set workers.joinServers={ ROOT_HOST_1 : PORT , ROOT_HOST_2 : PORT }` : An explicit comma-separated list of root server addresses to join. The default is an empty list ( `[]` ), which discovers all active root servers from the deployment topology.

  - ` --set workers.nodeLabelKey= NODE_LABEL_KEY  ` : The Kubernetes node label key used for node affinity and tolerations to isolate workers to a dedicated node pool. The default is `spanner-role` . Set to empty string `""` to disable node affinity and tolerations.

  - ` --set workers.nodeLabelValue= NODE_LABEL_VALUE  ` : The Kubernetes node label value used for node affinity and tolerations. The default is `workers` .

  - ` --set workers.pdbMaxUnavailable= MAX_UNAVAILABLE  ` : The maximum number of worker pods that can be unavailable during voluntary disruptions in the `PodDisruptionBudget` . The default is `1` .

  - `workers.affinity` : Custom Kubernetes affinity rules for worker pods. If not specified, default node affinity (using `workers.nodeLabelKey` and `workers.nodeLabelValue` ) and pod anti-affinity across hostnames ( `kubernetes.io/hostname` ) are applied. Because this is a nested object, specify it in a `values.yaml` file using the `-f` flag.

### Verify the worker deployment

To verify that the worker pods are running and ready, run the following command:

    kubectl get pods --namespace NAMESPACE -l app.kubernetes.io/component=spanner-worker

## Scale and decommission workers

Workers don't store user data or participate in database consensus. Scaling and decommissioning workers is instantaneous. You can start a worker before or after starting vector index creation, and decommission the worker immediately after index creation completes.

### Automate worker scaling

To automate creating and scaling workers, monitor the `spanner_box_compute_heavy_workers_required` metric. When the metric value is greater than `0` , the deployment requires one or more workers to complete pending background operations, such as building a vector index on a large table. When the metric value returns to `0` , all pending operations are complete and you can decommission the workers.

### Decommission a VM worker

To stop a worker process running on a VM, press Control + C in the terminal running the worker process, or stop the process by using its process ID (PID):

    kill -TERM PID

Replace `  PID  ` with the process ID of the `spanner workers` process. Alternatively, shut down the worker VM.

### Decommission a Kubernetes worker

To decommission workers on Kubernetes, disable workers in your Helm release or scale down worker replicas directly using `kubectl` :

  - **Disable workers** : To remove the worker `StatefulSet` and service from your cluster while preserving the rest of your deployment, run the `helm upgrade` command with `workers.enabled=false` :
    
        helm upgrade spanner-omni HELM_CHART_PATH \
          --reuse-values \
          --set workers.enabled=false \
          --namespace NAMESPACE
    
    Replace the following:
    
      - `  HELM_CHART_PATH  ` : The path to your Spanner Omni Helm chart.
      - `  NAMESPACE  ` : The Kubernetes namespace where your Spanner Omni cluster is deployed—for example, `spanner-ns` .

  - **Scale down worker replicas** : To scale down worker pods to zero replicas while keeping the worker configuration active in your cluster, run the `kubectl scale` command:
    
        kubectl scale statefulset spanner-worker \
          --replicas=0 \
          --namespace NAMESPACE
    
    Replace `  NAMESPACE  ` with the Kubernetes namespace where your Spanner Omni cluster is deployed—for example, `spanner-ns` .

## Monitor and troubleshoot workers

If your deployment has monitoring enabled, you can monitor workers using Prometheus or [Grafana](https://docs.cloud.google.com/spanner-omni/grafana-dashboards) dashboards. Workers expose metrics similar to Spanner Omni servers. Grafana dashboards include a **Worker Insights** dashboard that lets you monitor the resource utilization of each worker.

Workers write log files to the `logs` subdirectory within the base directory specified by `--base-dir` :

    BASE_DIR/logs

The `spanner admin diagnostics create` command doesn't collect logs or diagnostics from workers. To inspect worker logs, view the files in `  BASE_DIR /logs ` directly on the worker machine or pod, or run `kubectl logs` for Kubernetes worker pods.

For more information about monitoring and configuring dashboards, see [Monitoring overview](https://docs.cloud.google.com/spanner-omni/monitoring-overview) and [Monitor using Grafana dashboards](https://docs.cloud.google.com/spanner-omni/grafana-dashboards) .

### Vector index creation doesn't make progress

If you create a vector index on a large table and index creation remains pending without making progress, verify that at least one worker is running and connected to the deployment.

Spanner Omni lets you create a vector index even when no workers are active so that you can deploy workers only when required. If no worker is active, the index creation operation pauses indefinitely until a worker is deployed. Once a worker starts and registers with the deployment, index creation resumes automatically.
