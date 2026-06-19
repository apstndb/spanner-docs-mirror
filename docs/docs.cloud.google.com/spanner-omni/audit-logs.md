---
name: documents/docs.cloud.google.com/spanner-omni/audit-logs
uri: https://docs.cloud.google.com/spanner-omni/audit-logs
title: Spanner Omni audit logging
description: Learn about Spanner Omni audit logs, including Admin Activity, Data Access, and System Event audit logs, and how to configure and view them.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Spanner Omni writes audit logs that record administrative activities and data accesses within your Spanner Omni deployments.

Audit logs help you answer "who did what, where, and when?". Use audit logs to help your security, auditing, and compliance entities monitor your Spanner Omni data and systems for possible vulnerabilities or external data misuse.

## Before you begin

Configure your Spanner Omni deployment with TLS or mutual TLS (mTLS) encryption. If you create a deployment without TLS or mTLS encryption, audit logs aren't produced. For instructions on creating a deployment with TLS or mTLS encryption, see the following:

  - [Create a deployment with TLS encryption on VMs](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms)

  - [Create a deployment with TLS encryption on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes)

## Types of audit logs

Spanner Omni supports the following types of audit logs.

### Admin Activity audit logs

Admin Activity audit logs record API calls that modify the configuration or metadata of your deployment. For example, Spanner Omni writes Admin Activity audit logs when a user creates a database.

Admin Activity audit logs are produced by default for deployments configured with TLS and mTLS encryption. You can't configure, exclude, or disable them.

For a list of methods that write Admin Activity audit logs, see [Methods by permission type](https://docs.cloud.google.com/spanner-omni/audit-logs#methods-by-permission-type) .

### Data Access audit logs

Data Access audit logs record API calls that read the configuration or metadata of your deployment, as well as API calls that create, modify, or read user data. For example, Spanner Omni writes Data Access audit logs when a user describes a database, runs a SQL query, or executes a data manipulation language (DML) statement.

Data Access audit logs are disabled by default because they can be large. To record data access events, explicitly enable and configure them. Data Access audit logs help Google Support troubleshoot issues, so we recommend enabling them.

For instructions on configuring these logs, see [Configure Data Access audit logs](https://docs.cloud.google.com/spanner-omni/audit-logs#configure-data-access) . For a list of methods that write Data Access audit logs, see [Methods by permission type](https://docs.cloud.google.com/spanner-omni/audit-logs#methods-by-permission-type) .

### System Event audit logs

System Event audit logs record when the system modifies the configuration of your Spanner Omni resources without direct user action. For example, Spanner Omni writes a System Event audit log when the system automatically optimizes a newly restored database or creates a scheduled backup.

System Event audit logs are produced by default for deployments configured with TLS and mTLS encryption. You can't configure, exclude, or disable them.

For more information about the system events that generate these logs, see [System events](https://docs.cloud.google.com/spanner-omni/audit-logs#system-events-list) .

## Configure Data Access audit logs

To enable [Data Access audit logs](https://docs.cloud.google.com/spanner-omni/audit-logs#data-access) , configure the [Identity and Access Management (IAM) policy](https://docs.cloud.google.com/spanner-omni/iam) for your deployment using the Spanner Omni CLI or the API. You can enable logging for specific permission types and exempt specific users from being logged.

### Required permissions

To configure Data Access audit logs, you need the [`spanner.instances.setIamPolicy`](https://docs.cloud.google.com/spanner/docs/iam#instances) permission.

To view Data Access audit log configurations, you need the [`spanner.instances.getIamPolicy`](https://docs.cloud.google.com/spanner/docs/iam#instances) permission.

### Permission types

API methods check IAM permissions, and each permission has an associated permission type. When you configure Data Access audit logs, specify which permission types to log. The permission types are categorized as follows:

**Data Access permission types:**

  - `ADMIN_READ` : records API methods that read metadata or configuration information. For example, describing a database.
  - `DATA_READ` : records API methods that read user data. For example, executing a SQL query.
  - `DATA_WRITE` : records API methods that write user data. For example, executing a DML statement.

**Admin Activity permission type:**

  - `ADMIN_WRITE` : checked for API methods that write metadata or configuration information (such as creating a database). These generate Admin Activity audit logs and can't be configured.

For a detailed explanation of permission types, see [Permission types](https://docs.cloud.google.com/logging/docs/audit/configure-data-access#permission-types) in Cloud Audit Logs documentation.

### Configure using the Spanner Omni CLI

To configure Data Access audit logs using the Spanner Omni CLI, create a JSON policy file that lists the log types to enable and any exempted members, and then apply the policy.

1.  Create a policy file named `policy.json` :
    
        {
          "auditConfigs": [
            {
              "auditLogConfigs": [
                {
                  "logType": "ADMIN_READ",
                  "exemptedMembers": [
                    "USER_NAME"
                  ]
                },
                {
                  "logType": "DATA_WRITE"
                },
                {
                  "logType": "DATA_READ"
                }
              ],
              "service": "spanner.googleapis.com"
            }
          ]
        }

2.  Apply the policy using the `spanner instances set-iam-policy` command:
    
        spanner instances set-iam-policy default policy.json

To view the current Data Access audit log configuration, run the following command:

    spanner instances get-iam-policy default

For more examples of common configuration use cases, see [Configure Data Access audit logs](https://docs.cloud.google.com/logging/docs/audit/configure-data-access#config-api) in Cloud Audit Logs documentation.

## System events

Spanner Omni generates System Event audit logs for most of the system events supported by the managed version of Spanner. For more information, see [System events](https://docs.cloud.google.com/spanner/docs/audit-logging#system-events) in Spanner documentation.

Spanner Omni supports the `CreateScheduledBackup` and `OptimizeRestoredDatabase` system events. However, Spanner Omni doesn't support the `AutoscaleInstance` system event because autoscaling isn't available in Spanner Omni.

## Audit log files and retention

Spanner Omni writes audit logs to the local file system of your Spanner Omni servers.

Find the audit log files using the following path and naming convention: `  BASE_DIR /logs/ PROCESS /audit.log. BUCKET . TIMESTAMP . PROCESS_ID  `

  - `  BASE_DIR  ` : the base directory specified when starting your Spanner Omni deployment. For Kubernetes deployments, this defaults to `/spanner` .

  - `  PROCESS  ` : Spanner Omni runs multiple processes per server, such as `server` , `zone_services` , `zonal_zone_services` , `zonal_server` , and `base_services` . Audit logs are written to the directory corresponding to the process that generated them.

  - `  BUCKET  ` : Admin Activity and System Event audit logs are written to the `required` bucket, while Data Access audit logs are written to the `default` bucket.

  - `  TIMESTAMP  ` : the timestamp when the file was created.

  - `  PROCESS_ID  ` : the process ID of the process writing the audit log.

Spanner Omni creates a new audit log file when the current file exceeds 50 MiB or when a server process restarts. It also automatically maintains a symlink from `  BASE_DIR /logs/ PROCESS /audit.log. BUCKET  ` to the audit log file being written.

Access to local audit log files is controlled by your file system Access Control Lists (ACLs). Anyone with permission to sign in to the server and access these files can view the audit logs.

We recommend that you restrict access to the Spanner Omni VMs (for example, by restricting SSH access and configuring file system ACLs) so that only authorized personnel can view the logs. For long-term storage, auditing, and compliance, use a logging agent (such as [Fluentd](https://www.fluentd.org/) ) to collect and export the audit logs to an external logging system, such as [AWS CloudWatch](https://aws.amazon.com/cloudwatch/) , [Grafana Loki](https://grafana.com/oss/loki/) , [Elasticsearch](https://www.elastic.co/elasticsearch) , [Datadog](https://www.datadoghq.com/) , [Splunk](https://www.splunk.com/) , or [Cloud Monitoring](https://docs.cloud.google.com/monitoring) .

### File retention

Spanner Omni enforces both size-based and age-based retention policies for local audit log files:

  - **Size limit** : audit log files are retained until their total disk usage exceeds 1 GB. If a separate disk is used for the logs directory, audit log files are allocated a 25% quota of the total space on that disk.

  - **Age limit** : audit log files are retained for up to 14 days, provided the size limit isn't exceeded.

When either limit is reached, Spanner Omni deletes the oldest log files first. If you need to retain audit logs for more than 14 days, export them to an external logging system.

## Audit log entry structure

Spanner Omni audit log entries are formatted as JSON and use the same structural definition as Spanner. For more information, see [Audit log entry structure](https://docs.cloud.google.com/logging/docs/audit#audit-log-entry-structure) in Cloud Audit Logs documentation.

### Log name

Unlike the multi-tenant managed version of Spanner, Spanner Omni runs as a single-tenant deployment on dedicated servers. Because there are no other tenants or projects on these servers, all audit log entries use the project identifier `default` .

    projects/default/logs/cloudaudit.googleapis.com/activity
    projects/default/logs/cloudaudit.googleapis.com/data_access
    projects/default/logs/cloudaudit.googleapis.com/system_event

### Caller identities

Audit logs record the identity of the user who performed the logged operation. The caller's identity is stored in the `AuthenticationInfo` field of the [AuditLog](https://docs.cloud.google.com/logging/docs/reference/audit/auditlog/rest/Shared.Types/AuditLog) object.

In Spanner Omni, the caller's identity is the username of the authenticated user who performed the API call. For information about creating and managing users, see [Authentication and authorization](https://docs.cloud.google.com/spanner-omni/authentication) .

### IP address of the caller

Spanner Omni doesn't support recording the IP address of the caller in audit logs.

## Methods by permission type

When you call an API method, Spanner Omni generates an audit log based on the permission type required to perform the method. Methods requiring `ADMIN_READ` , `DATA_READ` , or `DATA_WRITE` permissions generate Data Access audit logs, while methods requiring `ADMIN_WRITE` permissions generate Admin Activity audit logs.

Spanner Omni supports most Cloud Spanner API methods. For the audit logging classification of methods shared by Spanner and Spanner Omni, see [Methods by permission type](https://docs.cloud.google.com/spanner/docs/audit-logging#permission-type) in Spanner documentation.

Spanner Omni also introduces unique API methods that generate audit logs. Methods marked as long-running operations (LROs) typically generate two audit log entries: one when the operation starts and another when it ends. For more information, see [Long-running operations](https://docs.cloud.google.com/logging/docs/audit/understanding-audit-logs#lro) in Cloud Audit Logs documentation.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Permission type</th>
<th>Methods</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">ADMIN_READ</code></td>
<td><code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.GetLocation</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.ListLocation</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.GetLocationDistance</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.ListLocationDistance</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.GetZone</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.ListZones</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.GetServer</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.ListServers</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.GetDeployment</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.ListBackupDescriptors</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.GetExternalStorage</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.ListExternalStorages</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.GetIamPolicy</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.UsersService.GetUser</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.UsersService.ListUsers</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.UsersService.GetRole</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.UsersService.ListRoles</code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">ADMIN_WRITE</code></td>
<td><code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.CreateLocation</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.DeleteLocation</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.CreateLocationDistance</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.UpdateLocationDistance</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.DeleteLocationDistance</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.CreateZone</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.DeleteZone</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.CreateServer</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.DeleteServer</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.CreateExternalStorage</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.DeleteExternalStorage</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.SetIamPolicy</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.OmniAdmin.ImportBackup</code> (LRO)<br />
<code dir="ltr" translate="no">google.spanner.omni.v1.UsersService.CreateUser</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.UsersService.DeleteUser</code><br />
<code dir="ltr" translate="no">google.spanner.omni.v1.UsersService.UpdateUser</code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">DATA_READ</code></td>
<td><code dir="ltr" translate="no">google.spanner.omni.v1.ImportExportService.ExportDatabase</code> (LRO)</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">DATA_WRITE</code></td>
<td><code dir="ltr" translate="no">google.spanner.omni.v1.ImportExportService.ImportDatabase</code> (LRO)</td>
</tr>
</tbody>
</table>

## Methods that don't produce audit logs

Spanner Omni excludes the same methods from audit logging as Spanner. These methods are excluded because they are high-volume methods involving significant log generation and storage costs, have low auditing value, or because another audit or platform log already provides method coverage. For a complete list of these excluded methods, see [Exempt methods](https://docs.cloud.google.com/spanner/docs/audit-logging#exempt-methods) in the Spanner documentation.

In addition, the `google.spanner.omni.v1.LoginService.Login` method doesn't produce audit logs because it precedes authentication and authorization checks.

## Processing duration

The processing duration field in an audit log records the processing duration of the API, such as the duration of a SQL query execution, not the delay of writing the audit log. For more information, see [Processing duration](https://docs.cloud.google.com/spanner/docs/audit-logging#processing_duration) in Spanner documentation.

## Comparison with the managed version of Spanner

While audit logging in Spanner Omni shares core concepts and formatting with Spanner, there are several key differences in how logs are stored, configured, and managed.

### Differences

  - **Log destination** : Spanner Omni writes audit logs directly to the local file system of your Spanner Omni servers, rather than exporting them to Cloud Logging.

  - **Configuration** : in Spanner, you configure Data Access audit logs at the project level using the Google Cloud console, the Google Cloud CLI, or the Identity and Access Management API. In Spanner Omni, you configure Data Access audit logs using the Spanner Omni CLI or the Spanner Omni API.

  - **TLS and mTLS encryption** : Spanner enforces TLS encryption by default, and it can't be disabled. In Spanner Omni, if you choose to deploy without TLS or mTLS encryption, audit logs aren't produced.

  - **Retention** : Spanner audit logs are retained in Logging according to your Logging retention settings. Spanner Omni retains local audit log files for up to 14 days or until disk size limits are reached, and relies on you to export the logs to an external system for long-term storage.

### Similarities

  - **Log format** : both products write audit logs in JSON format following the standard Google Cloud [AuditLog](https://docs.cloud.google.com/logging/docs/reference/audit/auditlog/rest/Shared.Types/AuditLog) structure.

  - **Data Access configuration** : both products let you enable Data Access audit logs granularly by permission type ( `ADMIN_READ` , `DATA_READ` , `DATA_WRITE` ) and support exempting specific users from being logged.

## What's next

  - [Learn about authentication and authorization in Spanner Omni](https://docs.cloud.google.com/spanner-omni/authentication) .

  - [Learn about IAM in Spanner Omni](https://docs.cloud.google.com/spanner-omni/iam) .

  - [Learn about monitoring your Spanner Omni deployment](https://docs.cloud.google.com/spanner-omni/monitoring-overview) .
