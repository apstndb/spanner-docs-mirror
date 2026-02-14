  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.ListBackupsResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Lists completed and pending backups. Backups returned are ordered by `  createTime  ` in descending order, starting from the most recent `  createTime  ` .

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/backups  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The instance to list backups from. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.backups.list  `

### Query parameters

Parameters

`  filter  `

`  string  `

An expression that filters the list of returned backups.

A filter expression consists of a field name, a comparison operator, and a value for filtering. The value must be a string, a number, or a boolean. The comparison operator must be one of: `  <  ` , `  >  ` , `  <=  ` , `  >=  ` , `  !=  ` , `  =  ` , or `  :  ` . Colon `  :  ` is the contains operator. Filter rules are not case sensitive.

The following fields in the `  Backup  ` are eligible for filtering:

  - `  name  `
  - `  database  `
  - `  state  `
  - `  createTime  ` (and values are of the format YYYY-MM-DDTHH:MM:SSZ)
  - `  expireTime  ` (and values are of the format YYYY-MM-DDTHH:MM:SSZ)
  - `  versionTime  ` (and values are of the format YYYY-MM-DDTHH:MM:SSZ)
  - `  sizeBytes  `
  - `  backupSchedules  `

You can combine multiple expressions by enclosing each expression in parentheses. By default, expressions are combined with AND logic, but you can specify AND, OR, and NOT logic explicitly.

Here are a few examples:

  - `  name:Howl  ` - The backup's name contains the string "howl".
  - `  database:prod  ` - The database's name contains the string "prod".
  - `  state:CREATING  ` - The backup is pending creation.
  - `  state:READY  ` - The backup is fully created and ready for use.
  - `  (name:howl) AND (createTime < \"2018-03-28T14:50:00Z\")  ` - The backup name contains the string "howl" and `  createTime  ` of the backup is before 2018-03-28T14:50:00Z.
  - `  expireTime < \"2018-03-28T14:50:00Z\"  ` - The backup `  expireTime  ` is before 2018-03-28T14:50:00Z.
  - `  sizeBytes > 10000000000  ` - The backup's size is greater than 10GB
  - `  backupSchedules:daily  ` - The backup is created from a schedule with "daily" in its name.

`  pageSize  `

`  integer  `

Number of backups to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

If non-empty, `  pageToken  ` should contain a `  nextPageToken  ` from a previous `  ListBackupsResponse  ` to the same `  parent  ` and with the same `  filter  ` .

### Request body

The request body must be empty.

### Response body

The response for `  backups.list  ` .

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
  &quot;backups&quot;: [
    {
      object (Backup)
    }
  ],
  &quot;nextPageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  backups[]  `

`  object ( Backup  ` )

The list of matching backups. Backups returned are ordered by `  createTime  ` in descending order, starting from the most recent `  createTime  ` .

`  nextPageToken  `

`  string  `

`  nextPageToken  ` can be sent in a subsequent `  backups.list  ` call to fetch more of the matching backups.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
