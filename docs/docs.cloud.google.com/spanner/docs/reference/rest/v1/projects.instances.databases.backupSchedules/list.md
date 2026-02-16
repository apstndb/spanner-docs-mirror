  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.ListBackupSchedulesResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Lists all the backup schedules for the database.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/backupSchedules  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. Database is the parent resource whose backup schedules should be listed. Values are of the form projects/ /instances/ /databases/

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.backupSchedules.list  `

### Query parameters

Parameters

`  pageSize  `

`  integer  `

Optional. Number of backup schedules to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

Optional. If non-empty, `  pageToken  ` should contain a `  nextPageToken  ` from a previous `  ListBackupSchedulesResponse  ` to the same `  parent  ` .

### Request body

The request body must be empty.

### Response body

The response for `  backupSchedules.list  ` .

If successful, the response body contains data with the following structure:

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;backupSchedules&quot;: [
    {
      object (BackupSchedule)
    }
  ],
  &quot;nextPageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  backupSchedules[]  `

`  object ( BackupSchedule  ` )

The list of backup schedules for a database.

`  nextPageToken  `

`  string  `

`  nextPageToken  ` can be sent in a subsequent `  backupSchedules.list  ` call to fetch more of the schedules.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
