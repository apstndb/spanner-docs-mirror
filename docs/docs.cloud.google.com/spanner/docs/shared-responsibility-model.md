Spanner is a fully managed, globally distributed, multi-model database service designed for high availability and extreme scale. As a managed service on Google Cloud, security and operational resilience responsibilities are shared between you and Google.

This document outlines the division of responsibility to ensure the security, compliance, and operation of your Spanner instances and data.

## Overview of the shared responsibility model

In a shared responsibility model, Google manages the security of the Spanner service, infrastructure, and underlying global network, while the customer is responsible for the security and management *in* the Spanner instance, including data, application access, and configuration.

| Responsibility Area                    | Google Cloud    | You             |
| -------------------------------------- | --------------- | --------------- |
| **Infrastructure**                     | Responsible     | Not Responsible |
| **Service configuration and security** | Shared          | Shared          |
| **Data and application access**        | Not Responsible | Responsible     |

## Google's responsibilities

Google is responsible for protecting the infrastructure that runs the Spanner service. This includes the physical, hardware, network, and operational components.

### Infrastructure and global availability

  - **Physical security:** Securing the global regions, zones, and physical data centers where Spanner infrastructure resides.

  - **Networking:** Providing the secure and reliable network necessary for Spanner's global consistency and replication.

  - **Hardware and software:** Managing the hardware, host operating systems, and the Spanner service software itself. This includes automated patching, maintenance, and updates.

### Service management and resilience

  - **High availability and scale:** Ensuring the 99.999% SLA (for multi-region configurations) by automatically managing scaling, replication, and failover across regions and zones. Spanner is designed for zero planned downtime. A Spanner instance can be excluded from the [Spanner Service Level Agreement (SLA)](https://cloud.google.com/spanner/sla) if user-controlled configurations cause an outage. To view these configurations, see [Spanner operational guidelines](https://docs.cloud.google.com/spanner/operational-guidelines) .

  - **Durability:** Ensuring the durability of data including backups stored within the Spanner system.

  - **Database software integrity:** Building, maintaining, and updating the Spanner software.

### Compliance and data protection

  - **Encryption at rest and in transit:** Ensures data is encrypted by default.

  - **Data residency:** Lets you manage data placement within specific regions or configurations (for example, [dual-region, multi-region](https://docs.cloud.google.com/spanner/docs/instance-configurations) ).

  - **Google Cloud Dedicated operations:** For Google Cloud Dedicated deployments, Google provides the infrastructure, software build, and updates, where our local trusted partners operate and support cloud services. Google may provide second-level assistance to the partners in performing maintenance and resolving issues.

## Your responsibilities

You maintain primary control over your data, configuration, access management, and application development when using Spanner.

### Data and schema management

  - **Data content and security:** Responsibility for the data content stored in Spanner, including its sensitivity, regulatory compliance, and integrity.

  - **Schema design and optimization:** Defining and managing the database schema, including creating tables, indexes, and managing interleaved tables for performance.

  - **Query optimization:** Designing efficient queries to ensure performance and manage resource allocation. For example, managing transaction spans and understanding locking behavior is critical.

### Access and identity management

  - **[IAM configuration](https://docs.cloud.google.com/spanner/docs/iam) :** Defining and managing Identity and Access Management (IAM) roles and permissions for principals (users and service accounts) accessing the Spanner instance and databases.

  - **[Fine-grained access control (FGAC)](https://docs.cloud.google.com/spanner/docs/fgac-about) :** If implemented, the customer is responsible for defining, managing, and re-granting access to database roles and privileges.

  - **[Audit logging](https://docs.cloud.google.com/spanner/docs/audit-logging) :** Monitoring and analyzing Cloud Audit Logs to track access and actions performed on the Spanner instance and data.

### Operational resilience and disaster recovery

  - **[Configuration management](https://docs.cloud.google.com/spanner/docs/instance-configurations)** : Managing the Spanner instance configuration, node counts, and regional deployments.

  - **[Backup and DR Service](https://docs.cloud.google.com/spanner/docs/backup) :** Implementing a strategy for disaster recovery that includes storing data outside of the Spanner instance itself (for example, in a separate region, instance, or external storage) to protect against scenarios like accidental deletion of the instance or data corruption.

  - **[(Optional) Change streams integration](https://docs.cloud.google.com/spanner/docs/change-streams) :** Configuring and managing Dataflow jobs or other consumers that utilize Spanner change streams for event streaming.

### Security configuration

  - **[Customer-managed encryption keys (CMEK)](https://docs.cloud.google.com/spanner/docs/cmek) :** If utilized, managing the Cloud Key Management Service (Cloud KMS) keys and permissions used to encrypt the Spanner data.

  - **[Request and transaction tagging](https://docs.cloud.google.com/spanner/docs/introspection/troubleshooting-with-tags) :** Applying tags to queries and transactions to enhance observability and performance monitoring.

  - **[Monitoring and alerting](https://docs.cloud.google.com/spanner/docs/monitoring-cloud) :** Setting up and tuning custom monitoring, exporting metrics, and configuring alerts to detect performance degradation or security anomalies.

## Summary of responsibilities

The following table summarizes the shared responsibilities for specific operational and security components:

| Component                    | Our Responsibility                                                                                             | Your Responsibility                                                                                                                                                                |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Data and user access**     | Physical isolation and protection of underlying storage.                                                       | IAM and FGAC management. Defining database roles and privileges.                                                                                                                   |
| **Network security**         | Network paths, firewalls, and segmentation of the Spanner service infrastructure.                              | Configuring Virtual Private Cloud (VPC), Private Service Connect, and client-side network rules.                                                                                   |
| **Backup and DR Service**    | Multi-region replication and 99.999% availability of the service. Point-in-Time Recovery (PITR) functionality. | Implementing a disaster recovery solution to store data outside of the primary Spanner instance, and managing application failover to a new database.                              |
| **Encryption**               | Encryption at rest and in transit by default.                                                                  | Managing and rotating CMEK keys, if utilized.                                                                                                                                      |
| **Backups**                  | Managing the backup service infrastructure and ensuring backup durability.                                     | Defining backup schedules, managing and accessing backups, and copying backups to other instances/regions.                                                                         |
| **Spanner instance**         | Provisioning and managing the underlying infrastructure.                                                       | Configuring the node counts, and locations.                                                                                                                                        |
| **Observability**            | Providing system tables (for example, `SPANNER_SYS` ) for diagnostics.                                         | Implementing custom monitoring, leveraging request/transaction tags, and integrating with external monitoring tools (for example, Prometheus and Grafana).                         |
| **Client applications**      | Providing Spanner client libraries and APIs.                                                                   | Developing, deploying, and securing all client applications that interact with the database.                                                                                       |
| **Configuration management** |                                                                                                                | Checking usage against [quotas](https://docs.cloud.google.com/spanner/quotas) and filing requests as needed. Using tools like Terraform to manage database and instance resources. |
