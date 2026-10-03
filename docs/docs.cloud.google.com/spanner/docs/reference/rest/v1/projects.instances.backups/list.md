---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list
title: 'Method: projects.instances.backups.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.ListBackupsResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#try-it)

Lists completed and pending backups. Backups returned are ordered by `createTime` in descending order, starting from the most recent `createTime` .

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/backups`

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
<p>Required. The instance to list backups from. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.backups.list</code></li>
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
<p>An expression that filters the list of returned backups.</p>
<p>A filter expression consists of a field name, a comparison operator, and a value for filtering. The value must be a string, a number, or a boolean. The comparison operator must be one of: <code>&lt;</code> , <code>&gt;</code> , <code>&lt;=</code> , <code>&gt;=</code> , <code>!=</code> , <code>=</code> , or <code>:</code> . Colon <code>:</code> is the contains operator. Filter rules are not case sensitive.</p>
<p>The following fields in the <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups#Backup"><code>Backup</code></a> are eligible for filtering:</p>
<ul>
<li><code>name</code></li>
<li><code>database</code></li>
<li><code>state</code></li>
<li><code>createTime</code> (and values are of the format YYYY-MM-DDTHH:MM:SSZ)</li>
<li><code>expireTime</code> (and values are of the format YYYY-MM-DDTHH:MM:SSZ)</li>
<li><code>versionTime</code> (and values are of the format YYYY-MM-DDTHH:MM:SSZ)</li>
<li><code>sizeBytes</code></li>
<li><code>backupSchedules</code></li>
</ul>
<p>You can combine multiple expressions by enclosing each expression in parentheses. By default, expressions are combined with AND logic, but you can specify AND, OR, and NOT logic explicitly.</p>
<p>Here are a few examples:</p>
<ul>
<li><code>name:Howl</code> - The backup's name contains the string "howl".</li>
<li><code>database:prod</code> - The database's name contains the string "prod".</li>
<li><code>state:CREATING</code> - The backup is pending creation.</li>
<li><code>state:READY</code> - The backup is fully created and ready for use.</li>
<li><code>(name:howl) AND (createTime &lt; \"2018-03-28T14:50:00Z\")</code> - The backup name contains the string "howl" and <code>createTime</code> of the backup is before 2018-03-28T14:50:00Z.</li>
<li><code>expireTime &lt; \"2018-03-28T14:50:00Z\"</code> - The backup <code>expireTime</code> is before 2018-03-28T14:50:00Z.</li>
<li><code>sizeBytes &gt; 10000000000</code> - The backup's size is greater than 10GB</li>
<li><code>backupSchedules:daily</code> - The backup is created from a schedule with "daily" in its name.</li>
</ul></td>
</tr>
<tr class="even">
<td><code>pageSize</code></td>
<td><p><code>integer</code></p>
<p>Number of backups to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.</p></td>
</tr>
<tr class="odd">
<td><code>pageToken</code></td>
<td><p><code>string</code></p>
<p>If non-empty, <code>pageToken</code> should contain a <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.ListBackupsResponse.FIELDS.next_page_token"><code>nextPageToken</code></a> from a previous <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#body.ListBackupsResponse"><code>ListBackupsResponse</code></a> to the same <code>parent</code> and with the same <code>filter</code> .</p></td>
</tr>
</tbody>
</table>

### Request body

The request body must be empty.

### Response body

The response for [`backups.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#google.spanner.admin.database.v1.DatabaseAdmin.ListBackups) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "backups": [
    {
      object (Backup)
    }
  ],
  "nextPageToken": string
}
```

| Fields          |                                                                                                                                                                                                                                                                            |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `backups[]`     | `object ( `[`Backup`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups#Backup)` )` The list of matching backups. Backups returned are ordered by `createTime` in descending order, starting from the most recent `createTime` .     |
| `nextPageToken` | `string` `nextPageToken` can be sent in a subsequent [`backups.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/list#google.spanner.admin.database.v1.DatabaseAdmin.ListBackups) call to fetch more of the matching backups. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
