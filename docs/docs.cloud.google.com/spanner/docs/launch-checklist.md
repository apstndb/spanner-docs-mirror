This launch checklist provides a list of considerations that need to be made prior to launching a production application on Spanner. It isn't intended to be exhaustive, but serves to highlights key considerations to minimize risks, optimize performance, and ensure alignment with business and operational goals, offering a systematic approach to deliver a seamless and reliable Spanner deployment.

This checklist is broken down into the following sections:

  - [Design, development, testing, and optimization](#design-development-testing-and-optimization)
  - [Migration (optional)](#migration)
  - [Deployment](#deployment)
  - [Query optimizer and statistics management](#query-optimizer)
  - [Disaster recovery](#disaster-recovery)
  - [Security](#security)
  - [Logging and monitoring](#logging-and-monitoring)
  - [Client library](#client-library)
  - [Support](#support)
  - [Cost management](#cost-management)

## Design, development, testing, and optimization

Optimizing schema design, transactions and queries is essential to use Spanner's distributed architecture for high performance and scalability. Rigorous at-production scale and end-to-end testing ensures the system can handle real-world workloads, peak loads, and concurrent operations, while minimizing risks of bottlenecks or failures in production.

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Design the schema with scalability and Spanner's distributed architecture in mind.</strong> Follow best practices such as selecting appropriate primary keys and indexes to avoid hotspots and consider optimizations like table interleaving for related data. Review <a href="/spanner/docs/schema-design">Schema design best practices</a> to ensure the schema supports both high performance and scalability under expected workloads.</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Optimize transactions and queries for minimal locking and maximum performance.</strong> Use Spanner's transaction modes, such as locking read-write, strong read-only, and partitioned DML statements, to balance consistency, throughput, and latency. Minimize locking scopes by using <a href="/spanner/docs/transactions#read-only_transactions">read-only transactions</a> for queries, <a href="/spanner/docs/dml-tasks#use-batch">batching</a> for maximum DML throughput or <a href="/spanner/docs/dml-partitioned">partitioned DML statements</a> for large-scale updates and deletes. When migrating from systems with different isolation levels (for example, PostgreSQL or MySQL), use transactions to avoid performance bottlenecks. For more information, see <a href="/spanner/docs/transactions">Transactions</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Conduct rigorous at-scale load testing to validate schema design, transaction behavior, and query performance.</strong> Simulate peak and high-concurrency scenarios that mimic real-world application loads, including diverse transaction shapes and query patterns. Evaluate latency and throughput under these conditions to confirm that the database design and instance topology meets performance requirements. Use load testing iteratively during development to optimize and refine implementation.</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Extend load testing to encompass all interacting services, not just isolated applications.</strong> Simulate comprehensive user journeys alongside parallel processes, such as batch loads or administration tasks that access the database. Run tests on the production Spanner instance configuration, ensuring load test drivers and services are geographically aligned with the intended production deployment topology. This holistic approach identifies potential conflicts in advance and ensures smooth database performance during real-world operations.</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>To ensure predictable query performance, use the optimizer version on which the workload has been tested.</strong> By default, Spanner databases use the latest query optimizer version. Regularly evaluate <a href="/spanner/docs/query-optimizer/versions">new optimizer versions</a> in a controlled environment, and update the default version only after confirming compatibility and performance improvements. For more information, see <a href="/spanner/docs/query-optimizer/overview">Query optimizer overview</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Ensure that <a href="/spanner/docs/query-optimizer/overview#statistics-packages">query optimizer statistics</a> are up-to-date to support efficient query execution plans.</strong> Although statistics are updated automatically, consider manually <a href="/spanner/docs/query-optimizer/overview#construct-statistics-package">constructing a new statistics package</a> in scenarios such as large-scale data modifications (for example, bulk inserts, updates or deletes), addition of new indexes, or schema changes. Keeping the query optimizer statistics current is critical for maintaining optimal query performance.</td>
</tr>
</tbody>
</table>

## Migration (optional)

[Database migration](/spanner/docs/migration-overview) is a comprehensive process that requires a deep dive into the specifics of each individual migration journey. Consider the following in your migration strategy:

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Develop a detailed standard operating procedure (SOP) for the migration cutover.</strong> This includes steps for application rollout, database switchover, and automation to minimize manual intervention. Identify and communicate potential downtime windows to stakeholders well in advance. Implement robust monitoring and alerting mechanisms to track the migration process in real-time and detect any anomalies promptly. Ensure the switchover process includes validation checks to confirm data integrity and application capabilities post-migration.</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Prepare a detailed fallback plan to revert to the source system in the case of critical issues during the migration.</strong> Test the fallback procedures in a staging environment to ensure that they are reliable, and can be executed with minimal downtime. Clearly define conditions that would trigger a fallback and ensure the team is trained to execute this plan swiftly and efficiently.</td>
</tr>
</tbody>
</table>

## Deployment

Proper deployment planning ensures that Spanner configurations meet workload requirements for availability, latency, and scalability, while accounting for geographic and operational considerations. Aligning sizing, resource management, failover scenarios, and automation minimizes risks, ensures optimal performance, and prevents resource constraints or outages during critical operations.

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Ensure your Spanner <a href="/spanner/docs/instance-configurations">instance configuration</a> (whether regional, dual-region, or multi-regional) aligns with your application's workload availability and latency requirements, while also taking geographic considerations into account.</strong> Calculate the target compute capacity based on expected storage sizes, traffic patterns, and <a href="/spanner/docs/cpu-utilization#recommended-max">recommended utilization limits</a> , ensuring sufficient capacity for zonal or regional outages. Plan for traffic peaks by enabling <a href="/spanner/docs/autoscaling-overview">autoscaling</a> . You can set an upper limit for compute capacity to establish cost safeguards. For more information, see <a href="/spanner/docs/compute-capacity">Compute capacity, nodes, and processing units</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>If you're using a dual-region or multi-region instance configuration, choose a leader region that minimizes latency for application writes from services deployed in your most latency-sensitive locations.</strong> Test the implications of different leader regions on operation latency, and adjust to optimize application performance. Plan for failover scenarios by ensuring that the application topology is able to adapt to leader region changes during regional outages. For more information, see <a href="/spanner/docs/modifying-leader-region">Modify the leader region of a database</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Configure tags and labels appropriately for operational clarity and Google Cloud resource tracking.</strong> Use tags to group instances by environment or workload type. Use labels for metadata that aids in cost analysis and permissions management. For more information, see <a href="/spanner/docs/tags">Control access and organize instances with tags</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Evaluate whether Spanner warm up is necessary, especially for services expecting sudden and high traffic upon launch.</strong> Testing latency under high initial loads might reveal the need for pre-launch warm up to ensure optimal performance. If warm up is required, generate artificial load. For more information, see <a href="/spanner/docs/pre-warm-database">Warm up the database before application launch</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Review Spanner limits and quotas before deployment.</strong> If necessary, request quota increases in the Google Cloud console to avoid constraints during peak periods. Be mindful of hard limits (for example, maximum tables per database) to prevent issues post-deployment. For more information, see <a href="/spanner/quotas">Quotas and limits</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Use automation tools like Terraform to provision and manage your Spanner instances, ensuring configurations are efficient and error-proof.</strong> For schema management, consider using tools like <a href="/spanner/docs/use-liquibase">Liquibase</a> to avoid accidental schema drops during updates. For more information, see <a href="/spanner/docs/use-terraform">Use Terraform with Spanner</a> .</td>
</tr>
</tbody>
</table>

## Disaster recovery

Establishing a robust [disaster recovery (DR)](/spanner/docs/backup/disaster-recovery-overview) strategy is essential to protect data, minimize downtime, and ensure business continuity during unexpected failures. Regularly testing of restore procedures and automating backups helps ensure operational readiness, compliance with recovery objectives, and reliable data protection tailored to organizational needs.

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Define a comprehensive disaster recovery strategy for Spanner that includes data protection, recovery objectives and failure scenarios.</strong> Establish clear recovery time objectives (RTO) and recovery point objectives (RPO) that align with business continuity requirements. Specify backup frequency, retention policies, and use <a href="/spanner/docs/pitr">point-in-time recovery (PITR)</a> to minimize data loss in case of failures. Review the <a href="/spanner/docs/backup/disaster-recovery-overview">Disaster recovery overview</a> to identify the right tools and techniques to ensure compliance with availability, reliability, and security for your application. For more information, see the <a href="https://services.google.com/fh/files/misc/spanner_data_protection_whitepaper_final_2.pdf">Data protection and recovery solutions in Spanner</a> whitepaper.</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Create detailed documentation for back up and restore procedures, including step-by-step guides for various recovery scenarios.</strong> Regularly test these procedures to ensure operational readiness and validate RTO and RPO requirements. Testing should simulate real-world failure conditions and scenarios to identify gaps and improve the recovery process. For more information, see <a href="/spanner/docs/backup/restore-backup-overview">Restore overview</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Implement automated backup schedules to ensure consistent and reliable data protection.</strong> Configure frequency and retention settings to match business needs and regulatory obligations. Use Spanner's backup scheduling features to automate the creation, management, and monitoring of backups. For more information, see <a href="/spanner/docs/backup/create-manage-backup-schedules">Create and manage backup schedules</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Align failover procedures with your application's <a href="/spanner/docs/instance-configurations">instance configuration topology</a> to minimize latency impacts in the case of an outage.</strong> Test disaster recovery scenarios, ensuring the application can operate efficiently when the leader region is moved to a failover region. For more information, see <a href="/spanner/docs/modifying-leader-region">Modify the leader region of a database</a> .</td>
</tr>
</tbody>
</table>

## Query optimizer and statistics management

Managing query optimizer versions and statistics is important for maintaining predictable and efficient query performance. Using tested versions and keeping statistics up-to-date ensures stability, prevents unexpected performance changes, and optimizes query execution plans, especially during significant data or schema modifications.

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td>By default, Spanner databases use the latest query optimizer version. <strong>To ensure predictable query performance, use the optimizer version on which the workload has been tested.</strong> Regularly evaluate <a href="/spanner/docs/query-optimizer/versions">new optimizer versions</a> in a controlled environment, and update the default version only after confirming compatibility and performance improvements. For more information, see the <a href="/spanner/docs/query-optimizer/overview">Query optimizer overview</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Ensure that <a href="/spanner/docs/query-optimizer/overview#statistics-packages">query optimizer statistics</a> are up-to-date to support efficient query execution plans.</strong> Although statistics are updated automatically, consider manually <a href="/spanner/docs/query-optimizer/overview#construct-statistics-package">constructing a new statistics package</a> in scenarios such as large-scale data modifications (for example, bulk inserts, updates or deletes), addition of new indexes, or schema changes. Keeping the query optimizer statistics current is critical for maintaining optimal query performance.</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>In certain scenarios, such as after bulk deletes or when new statistics generation might unpredictably impact query performance, pinning a specific statistics package is advisable.</strong> This provides consistent query performance until a new package can be generated and tested. Regularly review the need to pin statistics and unpin once updated packages are validated. For more information, see <a href="/spanner/docs/query-optimizer/overview#statistics-packages">Query optimizer statistics packages</a> .</td>
</tr>
</tbody>
</table>

## Security

Implementing access control measures is essential to protect sensitive data and prevent unauthorized access in Spanner. By enforcing [least-privilege access](/iam/docs/using-iam-securely#least_privilege) , [fine-grained access control (FGAC)](/spanner/docs/fgac-about) , and [database deletion protection](/spanner/docs/prevent-database-deletion) , you can minimize risk, ensure compliance, and safeguard critical assets against accidental or malicious actions.

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Review and implement Identity and Access Management (IAM) policies following the least-privilege principle for all users and service accounts accessing your database.</strong> Assign only the necessary permissions required to perform specific tasks and regularly audit access control permissions to ensure adherence to this model. Use service accounts with minimal privileges for automated processes to reduce the risk of unauthorized access. For more information, see the <a href="/spanner/docs/iam">IAM overview</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>If the application requires restricted access to specific rows, columns, or cells within a table, implement fine-grained access control (FGAC).</strong> Design and apply conditional access policies based on user attributes or data values to enforce granular access rules. Regularly review and update these policies to align with evolving security and compliance requirements. For more information, see the <a href="/spanner/docs/fgac-about">Fine-grained access control overview</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Implement automated backup schedules to ensure consistent and reliable data protection.</strong> Configure frequency and retention settings to match business needs and regulatory obligations. Use Spanner's backup scheduling features to automate the creation, management, and monitoring of backups. For more information, see <a href="/spanner/docs/backup/create-manage-backup-schedules">Create and manage backup schedules</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Enable database deletion protection to prevent accidental or unauthorized deletions.</strong> Combine this with strict IAM controls to limit deletion privileges to a small, trusted set of users or service accounts. Additionally, configure infrastructure automation tools like Terraform to include safeguards against unintentional deletion of your databases. This layered approach minimizes risks to critical data assets. For more information, see <a href="/spanner/docs/prevent-database-deletion">Prevent accidental database deletion</a> .</td>
</tr>
</tbody>
</table>

## Logging and monitoring

Effective logging and monitoring are critical for maintaining visibility into database operations, detecting anomalies, and ensuring system health. By using audit logs, distributed tracing, dashboards, and proactive alerts, you can quickly identify and resolve issues, optimize performance, and meet compliance requirements.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Enable audit logging to capture detailed information about database activities.</strong> Configure audit log levels appropriately based on compliance and operational requirements to monitor access patterns and detect anomalies effectively. Be aware that audit logs might grow large especially for <code dir="ltr" translate="no">        DATA_READ       </code> and <code dir="ltr" translate="no">        DATA_WRITE       </code> requests since all SQL and DML statements are logged for these respective requests. For more information, see <a href="/spanner/docs/audit-logging">Spanner audit logging</a> .<br />
<br />
<a href="/logging/docs/routing/overview">Routing these logs</a> to a user-defined log bucket lets you optimize your log retention costs (the first 30 days aren't charged) and to granularly control log access using <a href="/logging/docs/logs-views">log views</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Collect client-side metrics by instrumenting your application logic with OpenTelemetry to distribute tracing and observability.</strong> Set up OpenTelemetry instrumentation to capture traces and metrics from Spanner, ensuring end-to-end visibility into application performance and database interactions. For more information, see <a href="/spanner/docs/capture-custom-metrics-opentelemetry">Capture custom client-side metrics using OpenTelemetry</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Create and configure monitoring metrics to visualize query performance, latency, CPU utilization, and storage usage.</strong> Use these metrics for real-time tracking and historical analysis of database performance. For more information, see <a href="/spanner/docs/monitoring-cloud">Monitor instances with Cloud Monitoring</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Define threshold-based monitoring alerts for critical metrics to proactively detect and address issues.</strong> Configure alerts for conditions like high query latency, low storage availability, or unexpected spikes in traffic. Integrate these alerts with incident response tools for prompt action. For more information, see <a href="/spanner/docs/monitoring-cloud#create-alert">Create alerts for Spanner metrics</a> .</td>
</tr>
</tbody>
</table>

## Client library

Configuring operation tagging, session pools, and retry policies is vital for optimizing performance, debugging issues, and maintaining resilience in Spanner. These measures enhance observability, reduce latency, and ensure efficient handling of workload demands and transient errors, aligning system behavior with application requirements.

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Configure the client library to use meaningful query request and transaction tags.</strong> You can use request and transaction tags to develop an understanding of your queries, reads, and transactions. As a best practice, use contextual metadata such as application component, request type, or user context, in your tags to enable enhanced debugging and introspection. Ensure tags are visible in query statistics and logs to facilitate performance analysis and troubleshooting. For more information, see <a href="/spanner/docs/introspection/troubleshooting-with-tags">Troubleshoot with request tags and transaction tags</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Optimize session management by enabling session pooling in the client library.</strong> Configure pool settings, such as minimum and maximum sessions, to match workload demands while minimizing latency. Regularly monitor session usage to fine-tune these parameters and ensure that the session pool provides consistent performance benefits. For more information, see <a href="/spanner/docs/sessions">Sessions</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>In rare scenarios, the default client library parameters for retries, including maximum attempts and exponential backoff intervals, need to be configured to balance resilience with performance.</strong> Test these policies thoroughly to ensure that they align with application needs. For more information, see <a href="/spanner/docs/custom-timeout-and-retry">Configure custom timeouts and retries</a> .</td>
</tr>
</tbody>
</table>

## Support

To minimize downtime and impact, define clear incident roles and responsibilities to ensure prompt and coordinated responses to Spanner-related issues. For more information, see [Get support](/spanner/docs/getting-support) .

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Establish a clear incident response framework, defining roles and responsibilities for all team members involved in managing Spanner-related incidents.</strong> Designate incident roles such as Incident Commander, Communications Lead, and Subject Matter Experts (SMEs) to ensure efficient coordination and communication during incidents. Develop and document processes for identifying, escalating, mitigating and resolving issues. Follow best practices outlined in the <a href="https://sre.google/workbook/incident-response/">Google SRE Workbook on Incident Response</a> and <a href="https://sre.google/sre-book/managing-incidents/">Managing Incidents</a> . Conduct regular incident response training and simulations to ensure readiness and improve the team's ability to manage high-pressure scenarios effectively.</td>
</tr>
</tbody>
</table>

## Cost management

Implementing cost management strategies like autoscaling and incremental backups ensure efficient resource utilization and significant cost savings. Aligning resource provisioning with workload demands and optimizing non-production environments further reduces expenses while maintaining performance and flexibility.

<table>
<thead>
<tr class="header">
<th>Checkbox</th>
<th>Activity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>❑</td>
<td><strong>Evaluate and purchase CUDs for Spanner to lower costs on predictable workloads.</strong> These commitments might provide significant savings compared to on-demand pricing. Analyze historical usage patterns to determine optimal CUD commitments. For more information see <a href="/spanner/docs/cuds">Committed use discounts</a> and <a href="https://cloud.google.com/spanner/pricing">Spanner pricing</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Monitor compute capacity utilization and adjust provisioned resources to maintain recommended CPU utilization levels.</strong> Over-provisioning compute resources might lead to unnecessary costs, while under-provisioning might impact performance. Follow the recommended <a href="/spanner/docs/cpu-utilization#recommended-max">maximum Spanner CPU utilization guidelines</a> to ensure cost-effective resource alignment.</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Enable autoscaling to dynamically adjust compute capacity based on workload demands.</strong> This ensures optimal performance during peak loads while reducing costs during periods of low activity. Configure scaling policies with upper and lower limits to control cost and avoid over-scaling. For more information, see <a href="/spanner/docs/autoscaling-overview">Autoscaling overview</a> .</td>
</tr>
<tr class="even">
<td>❑</td>
<td><strong>Use incremental backups to reduce backup storage costs.</strong> Incremental backups only store data changes since the last backup. This significantly lowers storage requirements compared to full backups. Incorporate incremental backups into your backup strategy. For more information, see <a href="/spanner/docs/backup#incremental-backups">Incremental backups</a> .</td>
</tr>
<tr class="odd">
<td>❑</td>
<td><strong>Optimize costs for non-production environments by selecting the most optimal instance configuration and deprovisioning resources when environments aren't in use.</strong> For example, downsize non-critical environments after hours or automate resource scaling for development and testing scenarios. This approach minimizes costs while maintaining operational flexibility.</td>
</tr>
</tbody>
</table>
