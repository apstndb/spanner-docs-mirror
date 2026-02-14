The Google Kubernetes Engine (GKE) deployment model is is a good choice for independent teams who want to self-manage the infrastructure and configuration of their own Autoscalers on Kubernetes.

This document is part of a series that also includes:

  - [Autoscaling Spanner](/spanner/docs/autoscaling-overview)
  - [Autoscaler tool overview](/spanner/docs/autoscaler-tool-overview)
  - [Deploy the Autoscaler tool for Spanner to Cloud Run functions](/spanner/docs/set-up-autoscaling-gke)

This series is intended for IT, Operations, and Site Reliability Engineering (SRE) teams who want to reduce operational overhead and to optimize the cost of Spanner deployments.

The GKE deployment has the following advantages and disadvantages:

Advantages:

  - **Kubernetes-based** : For teams that might not be able to use services such as Cloud Run functions, deployment to Kubernetes enables use of the Autoscaler.
  - **Configuration** : The control over scheduler parameters belongs to the team that owns the Spanner instance which gives them the highest privileges to adapt the Autoscaler to their needs.

Disadvantages:

  - **Infrastructure** : Compared to the Cloud Run functions design, some long-lived infrastructure and services are required.
  - **Maintenance** : With each team being responsible for the Autoscaler configuration and infrastructure it might become difficult to make sure that all Autoscalers across the company follow the same update guidelines.
  - **Audit** : Because of the high level of control by each team, a centralized audit might become more complex.

This page introduces two ways you can deploy the Autoscaler to GKE based on your requirements:

  - A [decoupled deployment topology](https://github.com/cloudspannerecosystem/autoscaler/tree/main/terraform/gke#decoupled-model) . The decoupled deployment model has the advantage that you can assign Poller and Scaler components individual permissions so that they run as separate service accounts. This means you have the flexibility to manage and scale the two components to suit your needs. However, this deployment model requires that the Scaler component be deployed as a long-running service, which consumes resources.
  - A [unified deployment topology](https://github.com/cloudspannerecosystem/autoscaler/tree/main/terraform/gke#unified-model) . The unified deployment model has the advantage that the Poller and Scaler components can be deployed as a single pod, which runs as a Kubernetes cron job. When the two components are deployed as a single pod there are no long-running components and only a single service account is required.

For most use cases, we recommend the unified deployment model.

### Configuration

The Autoscaler tool manages Spanner instances through the configuration defined in a [Kubernetes `  ConfigMap  `](https://kubernetes.io/docs/concepts/configuration/configmap) . If multiple Spanner instances need to be polled with the same interval, we recommend that you configure them in the same `  ConfigMap  ` . The following is an example of a configuration where two Spanner instances are managed with one configuration:

``` text
apiVersion: v1
kind: ConfigMap
metadata:
  name: autoscaler-config
  namespace: spanner-autoscaler
data:
  autoscaler-config.yaml: |
    ---
    - projectId: spanner-autoscaler-test
      instanceId: spanner-scaling-linear
      units: NODES
      minSize: 5
      maxSize: 30
      scalingMethod: LINEAR
    - projectId: spanner-autoscaler-test
      instanceId: spanner-scaling-threshold
      units: PROCESSING_UNITS
      minSize: 100
      maxSize: 3000
      metrics:
      - name: high_priority_cpu
        regional_threshold: 40
        regional_margin: 3
```

An instance can have one Autoscaler configuration with the linear method for normal operations, and also have another Autoscaler configuration with the direct method for planned batch workloads. See the complete list of configuration options in the [Poller `  README  ` file](https://github.com/cloudspannerecosystem/autoscaler/blob/main/src/poller/README.md#configuration-parameters) .

## Deploy to GKE

To learn how to deploy the Autoscaler to GKE in either the decoupled or unified deployment model, see the [step-by-step guide to GKE deployment](https://github.com/cloudspannerecosystem/autoscaler/blob/main/terraform/gke/README.md) .

# What's next

  - Learn how to [deploy the Autoscaler tool to Cloud Run functions](/spanner/docs/set-up-autoscaling-cloud-run) .
  - Read more about Spanner [recommended thresholds](/spanner/docs/monitoring-cloud#create-alert) .
  - Read more about Spanner [CPU utilization metrics](/spanner/docs/cpu-utilization) and [latency metrics](/spanner/docs/latency-guide) .
  - Learn about [best practices for Spanner schema design](/spanner/docs/schema-design) to avoid hotspots and for loading data into Spanner.
  - Explore reference architectures, diagrams, and best practices about Google Cloud. Take a look at our [Cloud Architecture Center](/architecture) .
