> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Spanner Omni is a downloadable version of Spanner that lets you deploy Google's distributed database technology across on-premises data centers, public clouds, and on your laptop. It delivers core Spanner capabilities, including horizontal scalability, high availability, ACID compliance, and strong external consistency, by using Paxos-based replication, automatic sharding, and the software-defined TrueTime API.

Spanner Omni integrates relational, graph, vector, and key-value data models with full-text and operational analytics capabilities. Like Spanner, Spanner Omni supports [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/overview) , [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/overview) , and [Spanner Graph Language](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) . You can achieve cross environment resilience, application portability, and a consistent development stack across various environments. Spanner Omni supports deployment options that include:

  - Virtual machines
  - Linux containers
  - Kubernetes clusters

## Spanner Omni Preview version limitations

The [Preview](https://cloud.google.com/products#product-launch-stages) version of Spanner Omni includes the core functionality in Spanner with the following notable exceptions:

  - Enterprise security features and TLS encryption aren't supported.

  - Spanner Omni stops writing data 90 days after you create a deployment.

  - [Backups](https://docs.cloud.google.com/spanner-omni/backups) and [restores](https://docs.cloud.google.com/spanner-omni/restores) aren't supported.

For early access to the edition with full features, [contact Google](https://cloud.google.com/consulting/spanner-omni) .

## Key features

Spanner Omni provides the following features:

  - **Deployment flexibility** : Run Spanner Omni on Linux or macOS. Spanner Omni supports public cloud environments, such as Google Cloud (Google Kubernetes Engine (GKE) and Cloud Storage) and Amazon Elastic Kubernetes Service (Amazon EKS) and Amazon Elastic Compute Cloud (Amazon EC2), on-premises, and laptops.

  - **High scalability** : Spanner Omni can automatically shard your data and balance the shards to achieve elastic scaling for reads and writes.

  - **High availability** : Spanner Omni offers multiple deployment topologies. Spanner Omni offers multi-zone and multi-cluster deployment to provide high availability in the event of zone or cluster failures.

  - **Strong external consistency** : Spanner Omni provides global transactional consistency using the software-based TrueTime that can be deployed in any environment.

  - **Interoperable multi-model** : Spanner Omni supports multiple data models (relational, key-value, graph, vector) and capabilities (full-text search, analytical processing) and provides cross-model queries and ACID-compliant transactions across different data models.

  - **Familiar interface** : Interact with your data using the Spanner CLI, which includes a SQL shell for interactive queries. Spanner Omni supports GoogleSQL, PostgreSQL dialects, and GQL which lets you migrate existing applications.

## Deployment topologies

Spanner Omni uses a hierarchy of regions, zones, and servers to define its deployment configuration. Spanner Omni offers the following deployment configurations:

  - **Single server** - Spanner Omni runs on a single machine. This topology is a good option for local development because it runs on a single machine. Upgrades to this topology cause downtime.

  - **Single-zone (or zonal)** - All Spanner Omni servers are part of one zone. Use a minimum of three servers for this deployment configuration. For optimal uptime, this topology has lower availability targets because a single-zone failure can cause an outage.

  - **Multi-zone (or regional)** - Spanner Omni distributes servers across multiple zones. All the zones are in a single location. For multi-zone deployments, use a minimum of three zones. Each zone must have at least one server. We recommend using three servers in each zone. This deployment configuration offers higher availability than single-zone deployment.

  - **Multi-cluster (or multi-regional)** - Spanner Omni servers reside in multiple zones across multiple clusters. For high availability, use three zones across two or more clusters, and each configure each zone with three or more servers.

## System requirements

To optimize performance, use the following minimum recommended requirements for server deployments:

| Environment                   | OS or platform            | Hardware recommendation                     |
| ----------------------------- | ------------------------- | ------------------------------------------- |
| On-premises                   | Linux (RHEL 9, Ubuntu 22) | 4 GB RAM per vCPU, 20+ GB disk space        |
| Cloud (Google Cloud and AWS)  | VMs or Kubernetes pods    | 4 vCPUs, 16 GB RAM per VM or Kubernetes pod |
| Developer (laptops, desktops) | macOS (M1, M2, M3)        | 4 GB RAM, 10 GB disk space                  |

For storage, dedicated solid-state drive (SSD) storage with an ext4 file system is recommended.

## Connectivity and development

To connect to Spanner Omni and develop applications, consider the following tools:

  - **Client libraries** : You can use existing Spanner client libraries for Java and Go by specifying the Spanner Omni endpoint. For example, in Java, set `setExperimentalHost("http://localhost:15000")` .

  - **PGAdapter** : This proxy lets existing PostgreSQL applications connect to Spanner Omni PostgreSQL-dialect databases using the standard PostgreSQL wire protocol.

  - **Diagnostics** : A dedicated `spanner admin diagnostics create` command collects logs, traces, and thread stacks for troubleshooting.
