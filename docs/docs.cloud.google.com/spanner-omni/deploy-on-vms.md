---
name: documents/docs.cloud.google.com/spanner-omni/deploy-on-vms
uri: https://docs.cloud.google.com/spanner-omni/deploy-on-vms
title: Create a deployment on VMs
description: Set up an insecure Spanner Omni deployment on VMs. Understand risks, configure topology, and manage databases.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document explains how to deploy Spanner Omni on virtual machines (VMs). This deployment doesn't have encryption. If you want to quickly set up a test or proof-of-concept environment to evaluate Spanner Omni, then creating an insecure deployment is the fastest way to get started because it doesn't require you to configure mTLS or other security measures. However, because of security risks, such as unencrypted network traffic and open access, we don't recommend this configuration for production environments. You can choose between a single-server or a regional deployment across multiple zones.

The [Preview](https://cloud.google.com/products#product-launch-stages) version of Spanner Omni doesn't support TLS encryption and stops writing data 90 days after you create a deployment. For early access to the edition with full features, [contact Google](https://cloud.google.com/consulting/spanner-omni) .

## Before you begin

Before you set up an insecure deployment, ensure you meet the following requirements:

  - **SSH access** : Have SSH access to every machine in the deployment to download and run the Spanner Omni binary.

  - **Connectivity** : All machines in the deployment can connect to each other.

  - **Networking** : Your network configuration allows TCP communication on ports `15000` to `15025` .

  - **Storage** : Each machine has enough storage to host the data the deployment handles.

  - **System requirements** : Ensure your setup meets all [system requirements](https://docs.cloud.google.com/spanner-omni/system-requirements) .

  - **vSphere configuration** : If you are running Spanner Omni on the vSphere virtualization platform, disable virtualization of the Time Stamp Counter (TSC). Add `monitor_control.virtual_rdtsc = FALSE` to the virtual machine's `.vmx` configuration file.

## Step 1: Decide the deployment topology

Spanner Omni uses a hierarchy of locations, zones, and servers to define its deployment topology. You can choose from the following topologies based on your availability targets:

  - **Single Server** : The deployment has a single server which runs on a single machine.

  - **Single Zone** : The deployment runs on multiple servers spread across a single zone.

  - **Single Location, Multiple Zones** (Replicated Deployment): The deployment runs on multiple servers distributed across multiple zones in one location.

  - **Multiple Locations, Multiple Zones** : The deployment runs on multiple servers distributed across multiple locations and multiple zones.

For more information, see [Spanner Omni key terms](https://docs.cloud.google.com/spanner-omni/key-terms) and [Spanner Omni deployment configurations](https://docs.cloud.google.com/spanner-omni/overview#deployment-topologies) .

For any deployment other than a single server, create a YAML configuration file named `deployment.yaml` that defines the topology. Only specify the root servers in this file. Add non-root servers later.

### Example: Replicated multi-zone deployment

The following example shows a configuration for a regional deployment across three zones.

    name: regional-deployment
    location:
      - name: us-central1
    zone:
      - name: us-central1-a
        location: us-central1
        single_server: false
        root_server:
          - host: rootserver1
      - name: us-central1-b
        location: us-central1
        single_server: false
        root_server:
          - host: rootserver2
      - name: us-central1-c
        location: us-central1
        single_server: false
        root_server:
          - host: rootserver3

### Example: Multi-location deployment

The following example shows a configuration for a deployment across three locations.

    name: multi-location-deployment
    location:
      - name: us-central1
      - name: europe-west2
      - name: asia-southeast1
    zone:
      - name: us-central1-a
        location: us-central1
        single_server: false
        root_server:
          - host: rootserver1
      - name: europe-west2-a
        location: europe-west2
        single_server: false
        root_server:
          - host: rootserver2
      - name: asia-southeast1-a
        location: asia-southeast1
        single_server: false
        root_server:
          - host: rootserver3

## Step 2: Download and set up the binary

Repeat these steps for each machine in the deployment:

1.  Create a directory for the binary and navigate to it.

2.  Download the binary:
    
        gsutil cp gs://spanner-omni/VERSION/spanner-omni-VERSION-linux-x86_64.tar.gz .
    
    You can also use `scp` or other file transfer tools to copy the binary to your VMs.

3.  Extract the binary:
    
        tar -xvf spanner-omni-VERSION-linux-x86_64.tar.gz

## Step 3: Start the servers

Create a base directory on each server to store data, metadata, and logs. If a server needs to restart, specify the same directory for continuity.

### Option A: Single server deployment

For a single server deployment, run the following command:

    spanner start-single-server --base-dir=SPANNER_BASE_DIR

After the server starts, proceed to [Step 6: Interact with the deployment](https://docs.cloud.google.com/spanner-omni/deploy-on-vms#step-6-interact) .

### Option B: Scale-out deployment

For scale-out deployments, start the server on each machine. The `--server-address` and `--zone` flags match the values in your deployment configuration. The network resolves the `server_address` .

    spanner start --root --server-address=RESOLVABLE_HOSTNAME --zone=ZONE_NAME --base-dir=SPANNER_BASE_DIR

For example:

    spanner start --root --server-address=rootserver1 --zone=us-central-1a --base-dir=./spanbasedir

## Step 4: Create the deployment

To create the deployment, follow these steps:

1.  Copy the `deployment.yaml` file that you created in [Step 1: Decide the deployment topology](https://docs.cloud.google.com/spanner-omni/deploy-on-vms#step-1-decide-deployment-topology) to one of the root servers.

2.  On that root server, run the following command to create the deployment:
    
        spanner deployment create --config-file=deployment.yaml
    
    Each machine's console displays messages indicating that the deployment is ready.

3.  Validate the deployment by listing the zones:
    
        spanner deployment zones list
    
    The output lists the locations, zones, and servers you specified in `deployment.yaml` .

## Step 5: (Optional) Configure a load balancer

Set up a TCP load balancer with the following details:

| Setting                | Value                                                        |
| ---------------------- | ------------------------------------------------------------ |
| **Protocol**           | TCP                                                          |
| **Backend IP**         | The IP addresses of your servers                             |
| **Port**               | `15000` (or the port you use in the `--server-address` flag) |
| **Health check URL**   | `http://         SERVER_IP        :15012/healthz`            |
| **Balancing strategy** | Round robin                                                  |

For multi-location deployments, you can set up one load balancer per location and a primary load balancer to distribute traffic across all locations.

## Step 6: Interact with the deployment

After your deployment is ready, you can interact with it using the CLI from any VM or a local machine. If you run the CLI from a separate machine, such as a developer laptop, download and extract the CLI package. For more information, see [Quickstart using the Spanner Omni CLI](https://docs.cloud.google.com/spanner-omni/cli-quickstart) .

To interact with your deployment, follow these steps:

1.  Create a database:
    
        spanner --deployment-endpoint=LOAD_BALANCER_IP_OR_SERVER_IP databases create DATABASE_NAME

2.  Open the SQL shell:
    
        spanner sql --database=DATABASE_NAME

3.  Create a table and insert data:
    
        CREATE TABLE names (
          nameId INT64 NOT NULL,
          name STRING(100)
        ) PRIMARY KEY (nameId);
        
        INSERT INTO names (nameId, name) VALUES (1, "Jack");

4.  Verify the database and data:
    
    To list databases: `bash spanner databases list`
    
    The output looks similar to the following:
    
    | NAME           | STATE | VERSION\_RETENTION\_PERIOD | EARLIEST\_VERSION\_TIME | ENABLE\_DROP\_PROTECTION |
    | -------------- | ----- | -------------------------- | ----------------------- | ------------------------ |
    | DATABASE\_NAME | READY | 1h                         | 2025-02-07T12:25:30Z    | false                    |
    

    To query the table: ` bash spanner sql --database= DATABASE_NAME  `

5.  Run the following SQL commands:
    
        SHOW TABLES;
        SELECT * FROM names;

## Step 7: (Optional) Scale the deployment

To scale capacity within a zone, you can add non-root servers. Run the following command on each new server:

    spanner start --server-address=NON_ROOT_HOSTNAME --join-servers=SERVER1:PORT1,SERVER2:PORT2,SERVER3:PORT3 --zone=ZONE_NAME --base-dir=SPANNER_BASE_DIR

## Step 8: (Optional) Observe the deployment

To monitor the health and performance of your deployment, you can set up metrics collection, visualization, and alerting. You can also collect distributed traces to analyze request latency.

### Set up Prometheus

Spanner Omni servers export metrics in Prometheus format on port `15012` . Add the following to your `scrape_configs` in `prometheus.yml` :

    scrape_configs:
      - job_name: 'spanner'
        static_configs:
          - targets: ['HOST1:15012', 'HOST2:15012', 'HOSTN:15012']

### Set up Grafana

To view monitoring data in Grafana:

1.  Provision the Prometheus data source by creating a file in `provisioning/datasources` :
    
        apiVersion: 1
        datasources:
          - name: Prometheus
            uid: prometheus
            type: prometheus
            access: proxy
            url: http://PROMETHEUS_HOST:PROMETHEUS_PORT
            jsonData:
              httpMethod: POST
              manageAlerts: false
              prometheusType: Prometheus
              prometheusVersion: PROMETHEUS_VERSION
              cacheLevel: 'High'
              disableRecordingRules: false
              incrementalQueryOverlapWindow: 10m

2.  Create a dashboard in the Grafana UI or provision it using `provisioning/dashboards` .

### Set up alerts

Configure alerts using the Grafana UI or configuration files. For example, you can trigger an alert if the 95th percentile (p95) of transaction latency exceeds 100 milliseconds. For a list of available alerts, see [Use Prometheus alerts to monitor Spanner Omni](https://docs.cloud.google.com/spanner-omni/prometheus-alerts) .

### Collect and analyze traces

Collect distributed tracing information in OTLP format and visualize it with tools like [Jaeger](https://www.jaegertracing.io/) .

1.  Set up an OTLP-compatible trace collector. For Jaeger:
    
        export COLLECTOR_OTLP_ENABLED=true
        jaeger-all-in-one
    
    Ensure the OTLP port allows network traffic between Spanner Omni and the collector.

2.  Set the `SPANNER_BOX_OTLP_TRACE_EXPORTER_ENDPOINT` environment variable and start the Spanner Omni servers:
    
        export SPANNER_BOX_OTLP_TRACE_EXPORTER_ENDPOINT=COLLECTOR_HOST:COLLECTOR_PORT
        spanner start --root --server-address=RESOLVABLE_HOSTNAME --zone=ZONE_NAME --base-dir=SPANNER_BASE_DIR

## What's next

  - [Add encryption to your VM deployment](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms)
