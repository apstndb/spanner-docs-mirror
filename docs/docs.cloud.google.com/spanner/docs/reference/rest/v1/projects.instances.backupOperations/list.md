---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list
title: 'Method: projects.instances.backupOperations.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.ListBackupOperationsResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#try-it)

Lists the backup long-running operations in the given instance. A backup operation has a name of the form `projects/<project>/instances/<instance>/backups/<backup>/operations/<operation>` . The long-running operation metadata field type `metadata.type_url` describes the type of the metadata. Operations returned include those that have completed/failed/canceled within the last 7 days, and pending operations. Operations returned are ordered by `operation.metadata.value.progress.start_time` in descending order starting from the most recently started operation.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/backupOperations`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>parent</code></td>
<td><p><code>string</code></p>
<p>Required. The instance of the backup operations. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.backupOperations.list</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>filter</code></td>
<td><p><code>string</code></p>
<p>An expression that filters the list of returned backup operations.</p>
<p>A filter expression consists of a field name, a comparison operator, and a value for filtering. The value must be a string, a number, or a boolean. The comparison operator must be one of: <code>&lt;</code> , <code>&gt;</code> , <code>&lt;=</code> , <code>&gt;=</code> , <code>!=</code> , <code>=</code> , or <code>:</code> . Colon <code>:</code> is the contains operator. Filter rules are not case sensitive.</p>
<p>The following fields in the operation are eligible for filtering:</p>
<ul>
<li><code>name</code> - The name of the long-running operation</li>
<li><code>done</code> - False if the operation is in progress, else true.</li>
<li><code>metadata.@type</code> - the type of metadata. For example, the type string for <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateBackupMetadata"><code>CreateBackupMetadata</code></a> is <code>type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata</code> .</li>
<li><code>metadata.&lt;field_name&gt;</code> - any field in metadata.value. <code>metadata.@type</code> must be specified first if filtering on metadata fields.</li>
<li><code>error</code> - Error associated with the long-running operation.</li>
<li><code>response.@type</code> - the type of response.</li>
<li><code>response.&lt;field_name&gt;</code> - any field in response.value.</li>
</ul>
<p>You can combine multiple expressions by enclosing each expression in parentheses. By default, expressions are combined with AND logic, but you can specify AND, OR, and NOT logic explicitly.</p>
<p>Here are a few examples:</p>
<ul>
<li><code>done:true</code> - The operation is complete.</li>
<li><code>(metadata.@type=type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata) AND</code> \ <code>metadata.database:prod</code> - Returns operations where:
<ul>
<li>The operation's metadata type is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateBackupMetadata"><code>CreateBackupMetadata</code></a> .</li>
<li>The source database name of backup contains the string "prod".</li>
</ul></li>
<li><code>(metadata.@type=type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata) AND</code> \ <code>(metadata.name:howl) AND</code> \ <code>(metadata.progress.start_time &lt; \"2018-03-28T14:50:00Z\") AND</code> \ <code>(error:*)</code> - Returns operations where:
<ul>
<li>The operation's metadata type is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateBackupMetadata"><code>CreateBackupMetadata</code></a> .</li>
<li>The backup name contains the string "howl".</li>
<li>The operation started before 2018-03-28T14:50:00Z.</li>
<li>The operation resulted in an error.</li>
</ul></li>
<li><code>(metadata.@type=type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) AND</code> \ <code>(metadata.source_backup:test) AND</code> \ <code>(metadata.progress.start_time &lt; \"2022-01-18T14:50:00Z\") AND</code> \ <code>(error:*)</code> - Returns operations where:
<ul>
<li>The operation's metadata type is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CopyBackupMetadata"><code>CopyBackupMetadata</code></a> .</li>
<li>The source backup name contains the string "test".</li>
<li>The operation started before 2022-01-18T14:50:00Z.</li>
<li>The operation resulted in an error.</li>
</ul></li>
<li><code>((metadata.@type=type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata) AND</code> \ <code>(metadata.database:test_db)) OR</code> \ <code>((metadata.@type=type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) AND</code> \ <code>(metadata.source_backup:test_bkp)) AND</code> \ <code>(error:*)</code> - Returns operations where:
<ul>
<li>The operation's metadata matches either of criteria:</li>
<li>The operation's metadata type is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateBackupMetadata"><code>CreateBackupMetadata</code></a> AND the source database name of the backup contains the string "test_db"</li>
<li>The operation's metadata type is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CopyBackupMetadata"><code>CopyBackupMetadata</code></a> AND the source backup name contains the string "test_bkp"</li>
<li>The operation resulted in an error.</li>
</ul></li>
</ul></td>
</tr>
<tr class="even">
<td><code>pageSize</code></td>
<td><p><code>integer</code></p>
<p>Number of operations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.</p></td>
</tr>
<tr class="odd">
<td><code>pageToken</code></td>
<td><p><code>string</code></p>
<p>If non-empty, <code>pageToken</code> should contain a <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.ListBackupOperationsResponse.FIELDS.next_page_token"><code>nextPageToken</code></a> from a previous <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#body.ListBackupOperationsResponse"><code>ListBackupOperationsResponse</code></a> to the same <code>parent</code> and with the same <code>filter</code> .</p></td>
</tr>
</tbody>
</table>

### Request body

The request body must be empty.

### Response body

The response for [`backupOperations.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#google.spanner.admin.database.v1.DatabaseAdmin.ListBackupOperations) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "operations": [
    {
      object (Operation)
    }
  ],
  "nextPageToken": string
}
```

| Fields          |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `operations[]`  | `object ( `[`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation)` )` The list of matching backup long-running operations. Each operation's name will be prefixed by the backup's name. The operation's metadata field type `metadata.type_url` describes the type of the metadata. Operations returned include those that are pending or have completed/failed/canceled within the last 7 days. Operations returned are ordered by `operation.metadata.value.progress.start_time` in descending order starting from the most recently started operation. |
| `nextPageToken` | `string` `nextPageToken` can be sent in a subsequent [`backupOperations.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#google.spanner.admin.database.v1.DatabaseAdmin.ListBackupOperations) call to fetch more of the matching metadata.                                                                                                                                                                                                                                                                                                                       |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
