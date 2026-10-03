---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create
title: 'Method: projects.instances.databases.backupSchedules.create'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#try-it)

Creates a new backup schedule.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/backupSchedules`

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
<p>Required. The name of the database that this backup schedule applies to.</p>
<p>Authorization requires one or more of the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permissions on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.backupSchedules.create</code></li>
<li><code>spanner.databases.createBackup</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

| Parameters         |                                                                                                                                                                                                                                                           |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `backupScheduleId` | `string` Required. The Id to use for the backup schedule. The `backupScheduleId` appended to `parent` forms the full backup schedule name of the form `projects/<project>/instances/<instance>/databases/<database>/backupSchedules/<backupScheduleId>` . |

### Request body

The request body contains an instance of [`BackupSchedule`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule) .

### Response body

If successful, the response body contains a newly created instance of [`BackupSchedule`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
