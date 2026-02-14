The Autoscaler tool is designed to allow for flexibility, and can accommodate the existing separation of responsibilities between your operations and application teams. The responsibility to configure the autoscaling of Spanner instances can be centralized with a single operations team, or it can be distributed to the teams closer to the applications served by those Spanner instances.

This document is part of a series that also includes:

  - [Autoscaling Spanner](/spanner/docs/autoscaling-overview)
  - [Autoscaler tool overview](/spanner/docs/autoscaler-tool-overview)
  - [Deploy the Autoscaler tool for Spanner to Google Kubernetes Engine (GKE)](/spanner/docs/set-up-autoscaling-gke)

This series is intended for IT, Operations, and Site Reliability Engineering (SRE) teams who want to reduce operational overhead and to optimize the cost of Spanner deployments.

This page introduces three ways you can deploy the Autoscaler to Cloud Run functions, according to your requirements:

  - [A per-project deployment topology](#per-project_topology) . The Autoscaler infrastructure is deployed in the same project as Spanner that needs to be autoscaled. We recommend this topology for independent teams who want to manage their own Autoscaler configuration and infrastructure. A per-project deployment topology is also a good starting point for testing the capabilities of the Autoscaler.
  - [A centralized deployment topology](#centralized_topology) . The Autoscaler tool is deployed in one project and manages one or more Spanner instances in different projects. We recommend this topology for teams who manage the configuration and infrastructure of one or more Spanner instances while keeping the components and configuration for Autoscaler in a central place. In the centralized topology, in addition to an Autoscaler project, you set up a second project, which in this tutorial is referred to as the *Application project* . The Application project holds the application resources, including Spanner.
  - [A distributed deployment topology](#distributed_topology) . Most of the Autoscaler infrastructure is deployed in one project but some infrastructure components are deployed with the Spanner instances being autoscaled in different projects. We recommend this topology for organizations with multiple teams, where teams who own the Spanner instances want to manage only the Autoscaler configuration parameters for their instances, but the rest of the Autoscaler infrastructure is managed by a central team.

### Serverless for ease of deployment and management

In this model, the Autoscaler tool is built using only serverless and low management Google Cloud tools, such as Cloud Run functions, Pub/Sub, Cloud Scheduler, and Firestore. This approach minimizes the cost and operational overhead of running the Autoscaler tool.

By using built-in Google Cloud tools, the Autoscaler tool can take full advantage of [Identity and Access Management (IAM)](/iam) for authentication and authorization.

### Configuration

The Autoscaler tool manages Spanner instances through the configuration defined in Cloud Scheduler. If multiple Spanner instances need to be polled with the same interval, we recommend that you configure them in the same Cloud Scheduler job. The configuration of each instance is represented as a JSON object. The following is an example of a configuration where two Spanner instances are managed with one Cloud Scheduler job:

``` text
[
  {
    "projectId": "my-spanner-project",
    "instanceId": "my-spanner",
    "scalerPubSubTopic": "projects/my-spanner-project/topics/spanner-scaling",
    "units": "NODES",
    "minSize": 1,
    "maxSize": 3
  },
  {
    "projectId": "different-project",
    "instanceId": "another-spanner",
    "scalerPubSubTopic": "projects/my-spanner-project/topics/spanner-scaling",
    "units": "PROCESSING_UNITS",
    "minSize": 500,
    "maxSize": 3000,
    "scalingMethod": "DIRECT"
  }
]
```

Spanner instances can have multiple configurations on different Cloud Scheduler jobs. For example, an instance can have one Autoscaler configuration with the linear method for normal operations, but also have another Autoscaler configuration with the direct method for planned batch workloads.

When the Cloud Scheduler job runs, it sends a Pub/Sub message to the Polling Pub/Sub topic. The payload of this message is the JSON array of the configuration objects for all the instances configured in the same job. See the complete list of configuration options in the [Poller `  README  ` file](https://github.com/cloudspannerecosystem/autoscaler/blob/main/src/poller/README.md#configuration-parameters) .

## Per-project topology

In a per-project topology deployment, each project with a Spanner instance needing to be autoscaled also has its own independent deployment of the Autoscaler components. We recommend this topology for independent teams who want to manage their own Autoscaler configuration and infrastructure. It's also a good starting point for testing the capabilities of the Autoscaler tool.

The following diagram shows a high-level conceptual view of a per-project deployment.

The per-project deployments depicted in the preceding diagram have these characteristics:

  - Two applications, Application 1 and Application 2, each use their own Spanner instances.
  - Spanner instances (A) live in respective Application 1 and Application 2 projects.
  - An independent Autoscaler (B) is deployed into each project to control the autoscaling of the instances within a project.

A per-project deployment has the following advantages and disadvantages.

Advantages:

  - **Simplest design** : The per-project topology is the simplest design of the three topologies since all the Autoscaler components are deployed alongside the Spanner instances that are being autoscaled.
  - **Configuration** : The control over scheduler parameters belongs to the team that owns the Spanner instance, which gives the team more freedom to adapt the Autoscaler tool to its needs than a centralized or distributed topology.
  - **Clear boundary of infrastructure responsibility** : The design of a per-project topology establishes a clear boundary of responsibility and security over the Autoscaler infrastructure because the team owner of the Spanner instances is also the owner of the Autoscaler infrastructure.

Disadvantages:

  - **More overall maintenance** : Each team is responsible for the Autoscaler configuration and infrastructure so it might become difficult to make sure that all of the Autoscaler tools across the company follow the same update guidelines.
  - **More complex audit** : Because each team has a high level of control, a centralized audit may become more complex.

To learn how to set up the Autoscaler using a per-project topology, see [the step-by-step guide to per-project deployment](https://github.com/cloudspannerecosystem/autoscaler/blob/main/terraform/cloud-functions/per-project/README.md) .

## Centralized topology

As in the per-project topology, in a centralized topology deployment all of the components of the Autoscaler tool reside in the same project. However, the Spanner instances are located in different projects. This deployment is suited for a team managing the configuration and infrastructure of several Spanner instances from a single deployment of the Autoscaler tool in a central place.

The following diagram shows a high-level conceptual view of a centralized-project deployment:

The centralized deployment shown in the preceding diagram has the following characteristics:

  - Two applications, Application 1 and Application 2, each use their own Spanner instances.
  - Spanner instances (A) are in respective Application 1 and Application 2 projects.
  - Autoscaler (B) is deployed into a separate project to control the autoscaling of the Spanner instances in both the Application 1 and Application 2 projects.

A centralized deployment has the following advantages and disadvantages.

Advantages:

  - **Centralized configuration and infrastructure** : A single team controls the scheduler parameters and the Autoscaler infrastructure. This approach can be useful in heavily regulated industries.
  - **Less overall maintenance** : Maintenance and setup are generally less effort to maintain compared to a per-project deployment.
  - **Centralized policies and audit** : Best practices across teams might be easier to specify and enact. Audits might be easier to execute.

Disadvantages:

  - **Centralized configuration** : Any change to the Autoscaler parameters needs to go through the centralized team, even though the team requesting the change owns the Spanner instance.
  - **Potential for additional risk** : The centralized team itself might become a single point of failure even if the Autoscaler infrastructure is designed with high availability in mind.

To learn how to set up the Autoscaler using a centralized topology, see [the step-by-step guide to centralized deployment](https://github.com/cloudspannerecosystem/autoscaler/blob/main/terraform/cloud-functions/centralized/README.md) .

## Distributed topology

In a distributed topology deployment, the Cloud Scheduler and Spanner instances that need to be autoscaled reside in the same project. The remaining components of the Autoscaler tool reside in a centrally managed project. This deployment is a hybrid deployment. Teams that own the Spanner instances manage only the Autoscaler configuration parameters for their instances, and a central team manages the remaining Autoscaler infrastructure.

The following diagram shows a high-level conceptual view of a distributed-project deployment.

The hybrid deployment depicted in the preceding diagram has the following characteristics:

  - Two applications, Application 1 and Application 2, use their own Spanner instances.
  - The Spanner instances (A) are in both Application 1 and Application 2 projects.
  - An independent Cloud Scheduler component (C) is deployed into each project: Application 1 and Application 2.
  - The remaining Autoscaler components (B) are deployed into a separate project.
  - The Autoscaler tool autoscales the Spanner instances in both the Application 1 and Application 2 projects using the configurations sent by the independent Cloud Scheduler components in each project.

### Forwarder function

Cloud Scheduler can only publish messages to topics in the same project, so for the distributed topology, an intermediate component called *the Forwarder function* is required.

The Forwarder function takes messages published to Pub/Sub from Cloud Scheduler, checks their JSON syntax, and forwards them to the Poller Pub/Sub topic. The topic can belong to a separate project to the Cloud Scheduler.

The following diagram shows the components used for the forwarding mechanism:

As shown in the preceding diagram, the Spanner instances are in projects named *Application 1* and *Application 2* :

1.  Cloud Scheduler is the same project as the Spanner instances.

2.  (2a) Cloud Scheduler publishes its messages to the Forwarder topic in the Application 1 and Application 2 projects.
    
    (2b) The Forwarder function reads messages from the Forwarder topic.
    
    (2c) The Forwarder function forwards messages to the Polling topic residing in the Autoscaler project.

3.  The Poller function reads the messages from the polling topic and the process continues, as described in the [Poller](/spanner/docs/autoscaler-tool-overview#poller) section.

A distributed deployment has the following advantages and disadvantages.

Advantages:

  - **Application teams control configuration and schedules** : Cloud Scheduler is deployed alongside the Spanner instances that are being autoscaled, giving application teams more control over configuration and scheduling.
  - **Operations team controls infrastructure** : Core components of the Autoscaler tool are centrally deployed giving operations teams control over the Autoscaler infrastructure.
  - **Centralized maintenance** : Scaler infrastructure is centralized, reducing overhead.

Disadvantages:

  - **More complex configuration** : Application teams need to provide service accounts to write to the polling topic.
  - **Potential for additional risk** : The shared infrastructure might become a single point of failure even if the infrastructure is designed with high availability in mind.

To learn how to set up the Autoscaler using a distributed topology, see [the step-by-step guide to distributed deployment](https://github.com/cloudspannerecosystem/autoscaler/blob/main/terraform/cloud-functions/distributed/README.md) .

## What's next

  - Learn how to [deploy the Autoscaler tool to GKE](/spanner/docs/set-up-autoscaling-gke) .
  - Read more about Spanner [recommended thresholds](/spanner/docs/monitoring-cloud#create-alert) .
  - Read more about Spanner [CPU utilization metrics](/spanner/docs/cpu-utilization) and [latency metrics](/spanner/docs/latency-guide) .
  - Learn about [best practices for Spanner schema design](/spanner/docs/schema-design) to avoid hotspots and for loading data into Spanner.
  - Explore reference architectures, diagrams, and best practices about Google Cloud. Take a look at our [Cloud Architecture Center](/architecture) .
