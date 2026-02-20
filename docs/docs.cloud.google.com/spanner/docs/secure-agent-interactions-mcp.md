**Preview**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

Model Context Protocol (MCP) standardizes how generative AI agents connect to Spanner. Due to the inherent risks of autonomous agents, mitigating vulnerabilities like prompt injection requires a shared responsibility model, combining platform controls with secure application design.  
To design and deploy AI applications that use Google Cloud Model Context Protocol (MCP) tools, follow the best practices in this guide.

## Before you begin

When you use MCP tools, your application's security posture depends on the agent's interaction model. To learn how your use of agents impacts the security risks associated with integrating agents with an MCP server, see [AI security and safety](https://docs.cloud.google.com/mcp/ai-security-safety) .

## Security responsibilities

As a customer, you are responsible for the secure configuration and operation of your agent platform.

### Follow the principle of least privilege

Run your agent with a minimally-scoped service account. This is the first and most critical layer of defense.

  - **Dedicated identity:** Create a separate, dedicated service account for each unique agent or application using MCP tools. Do not reuse existing SAs, especially those with broad permissions.
  - **Minimal scopes:** Grant the service account only the necessary Identity and Access Management (IAM) roles—for example, `  alloydb.viewer  ` , not `  alloydb.admin  ` . If the agent only needs read access to a specific dataset, use custom IAM roles to restrict access to the absolute minimum needed for its function.
  - **Separation of duties:** If an agent needs both read access to data and write access to a log or temporary storage, use two separate service accounts—one account for high-risk data access (minimally scoped) and one for low-risk operational tasks.

### Use database-native granular controls

For the strongest defense, combine IAM roles with the granular access controls offered by the database itself. This ensures that even if an attacker compromises the agent's IAM token, the scope of damage is limited by the database engine's internal permissions—for example, preventing a `  DROP TABLE  `

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th><br />
<strong>Product</strong></th>
<th><br />
<strong>Granular Control Mechanism</strong></th>
<th><br />
<strong>Focus</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><br />
<strong>Cloud SQL and AlloyDB</strong></td>
<td><br />
Database-level roles like CREATE ROLE in PostgreSQL and MySQL.</td>
<td><br />
Manage permissions in a specific database instance and schemas.</td>
</tr>
<tr class="even">
<td><br />
<strong>BigQuery</strong></td>
<td><br />
Column-Level Access Control (using policy tags)</td>
<td><br />
Restrict agent access to sensitive columns—for example, PII— even in an authorized table.</td>
</tr>
<tr class="odd">
<td><br />
<strong>Spanner</strong></td>
<td><br />
Fine-Grained Access Control (Database roles with <code dir="ltr" translate="no">       GRANT/REVOKE      </code> )</td>
<td><br />
Enforce precise read/write/update permissions on tables and columns.</td>
</tr>
<tr class="even">
<td><br />
<strong>Firestore</strong></td>
<td><br />
Firestore Security Rules</td>
<td><br />
Restrict document-level or field-level access based on the application's logic or the requesting user context.</td>
</tr>
<tr class="odd">
<td><br />
<strong>Bigtable</strong></td>
<td><br />
IAM roles</td>
<td><br />
Bigtable offers granular control through IAM roles at the project, instance, and table levels.</td>
</tr>
</tbody>
</table>

## Secure agent design

Agent-Only models require robust application-level defenses against prompt injection attacks, which attempt to override the system prompt. For more information, see [AI safety and security](/mcp/ai-security-safety) .

### Treat data and user inputs as untrusted

Treat input from end users, or data fetched by the agent from external sources—like a web search result or a third-party document—as untrusted.

### Implement action-selection patterns

Avoid open-ended *plan and execute* *architectures* , in which the system decouples high-level task specification from mechanical execution. Instead, use design patterns that limit the model's freedom.

  - **Action-selector pattern:** the model's only job is to translate a user request into one of a small, pre-defined set of safe functions. The action logic is hard-coded and can't be modified by the LLM. This helps to make the agent immune to injection attacks targeting control flow.
  - **Dual-LLM pattern:** use a primary LLM (the action LLM) that performs the core task, and a secondary, highly-secure LLM (the guardrail LLM) that pre-screens the user prompt for malicious intent and post-screens the action LLM's output for unauthorized actions or data leakage.

### Prevent unauthorized tool chaining

Agents must only call tools that are necessary for the task. Make sure that your orchestration code prevents the following:

  - **Dynamic tools:** the agent must not be able to dynamically register new tools or change the permissions of existing tools.
  - **Allowlist enforcement:** declare an allowlist of functions or database tables that the agent can access in its initial system prompt and backend code. For a Gemini CLI example, see [Restricting Tool Access](https://google-gemini.github.io/gemini-cli/docs/cli/enterprise.html#restricting-tool-access) .

## Security and safety configurations

Spanner provides Model Armor to enforce safety boundaries at the platform level. You must enable and configure these settings.

### Enable Model Armor

Use the Google Cloud CLI to enable Model Armor on your model deployment. This activates built-in protection against known attack vectors like injection and jailbreaking.

The following example enables Model Armor on a Vertex AI endpoint.

``` text
# Example: Enable Model Armor on a Vertex AI endpoint
gcloud ai endpoints update ENDPOINT_ID \
    --region=REGION \
    --enable-model-armor
```

For more information and examples, see [Configure Model Armor protection for MCP on Google Cloud](/model-armor/model-armor-mcp-google-cloud-integration#:~:text=To%20protect%20your%20MCP%20tool,See%20the%20following%20example%20command:) .

### Enforce minimum safety thresholds for sensitive data operations

Model Armor lets you enforce a minimum safety threshold for sensitive data operations—for example, personally identifiable information (PII) detection and de-identification. Use a Sensitive Data Protection `  DeidentifyTemplate  ` to redact or mask sensitive information before it's returned to the user, even if the model is compromised.

The following is a conceptual example for configuration:

``` text
  # Example: Apply a DeidentifyTemplate to filter PII
gcloud ai endpoints update ENDPOINT_ID \
    --region=REGION \
    --model-armor-config-file=model_armor_config.json
```

In the following example, `  model_armor_config.json  ` might reference a DLP template:

``` text
{
  "safety_thresholds": {
    "injection": "HIGH",
    "harmful_content": "MEDIUM"
  },
  "data_protection_config": {
    "dlp_deidentify_template": "projects/PROJECT_NUMBER/locations/LOCATION/deidentifyTemplates/DLP_TEMPLATE_ID"
  }
}
```

## Auditing and observability

Visibility into agent actions is crucial for post-incident analysis and detection of compromised agents.

### Implement a data recovery strategy

While layered controls like IAM and database-native roles are designed to prevent destructive actions, you must have a recovery plan in case those defenses fail. A compromised or naive agent with write permissions (an "Agent-Only" risk) might be tricked into executing a destructive command like `  DROP TABLE  ` or corrupting data.

Your primary defense against this scenario is a robust recovery strategy.

Nearly all Data Cloud products provide features for data recovery, either through traditional backups, point-in-time recovery (PITR), or data snapshots. You are responsible for enabling and configuring these features.

<table>
<thead>
<tr class="header">
<th><strong>Product</strong></th>
<th><strong>Backup and recovery mechanisms</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Cloud SQL</td>
<td>Supports both on-demand and automated backups, allowing you to restore an instance to a previous state. It also supports Point-in-Time Recovery (PITR).</td>
</tr>
<tr class="even">
<td>AlloyDB</td>
<td>Provides continuous backup and recovery by default. This enables PITR with microsecond granularity, allowing you to restore a cluster to any time in your retention window.</td>
</tr>
<tr class="odd">
<td>BigQuery</td>
<td>Data recovery is achieved using "Time Travel," which lets you access and restore data from any point in the last 7 days. For longer-term retention, you can create Table Snapshots.</td>
</tr>
<tr class="even">
<td>Spanner</td>
<td>Supports both on-demand backups and PITR.</td>
</tr>
<tr class="odd">
<td>Firestore</td>
<td>Supports managed exports/imports as backups. It also offers PITR—disabled by default—to protect against accidental deletions or writes.</td>
</tr>
<tr class="even">
<td>Bigtable</td>
<td>Supports on-demand and automated backups. These backups are fully managed and can be restored to a new table.</td>
</tr>
</tbody>
</table>

### Enable Cloud Audit Logs

Make sure that [Data Access audit logs](/logging/docs/audit#data-access) are enabled for MCP as well as all relevant Google Cloud services like BigQuery, Cloud SQL, AlloyDB, and Spanner. By default, only [Admin Activity audit logs](/logging/docs/audit#admin-activity) are enabled. Data Access audit logs record every read and write operation performed by the agent. For more information, see [Data access audit logs for MCP](/mcp/audit-logging#data-access-audit-logs-for-mcp) .

### Audit sensitive actions

[Configure alerts in Cloud Logging](/logging/docs/alerting/log-based-alerts) to detect anomalous or high-risk actions. The Logs Explorer query identifies service accounts performing *data write* operations in Firestore, for example, which is a common target for exfiltration or destructive attacks:

``` text
resource.type="firestore_database"
# Filter for data write operations
AND protoPayload.methodName="google.firestore.v1.Firestore.Commit"
# Ensure the caller is an agent service account (modify regex as needed)
AND protoPayload.authenticationInfo.principalEmail=~".*@.*.gserviceaccount.com"
# Exclude expected system calls to reduce noise
AND NOT protoPayload.authenticationInfo.principalEmail=~"system-managed-service-account"
```

### Use agent-specific logging

In addition to Cloud Audit Logs, make sure that your application code logs the following data for every agent decision:

  - **Tool execution:** the MCP tool that was called.
  - **Raw command:** the exact command—for example, a SQL query or document path—generated by the LLM.
  - **Final action:** whether the action is executed (Agent-Only model) or approved (Human-in-the-Middle). For more information, see [Understand agent use](/mcp/ai-security-safety#understand-agent-use) .
  - **User and session ID:** the identifier for the end user who initiated the request.
