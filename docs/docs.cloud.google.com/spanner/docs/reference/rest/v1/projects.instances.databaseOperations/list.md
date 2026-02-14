  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.ListDatabaseOperationsResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Lists database longrunning-operations. A database operation has a name of the form `  projects/<project>/instances/<instance>/databases/<database>/operations/<operation>  ` . The long-running operation metadata field type `  metadata.type_url  ` describes the type of the metadata. Operations returned include those that have completed/failed/canceled within the last 7 days, and pending operations.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/databaseOperations  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The instance of the database operations. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.databaseOperations.list  `

### Query parameters

Parameters

`  filter  `

`  string  `

An expression that filters the list of returned operations.

A filter expression consists of a field name, a comparison operator, and a value for filtering. The value must be a string, a number, or a boolean. The comparison operator must be one of: `  <  ` , `  >  ` , `  <=  ` , `  >=  ` , `  !=  ` , `  =  ` , or `  :  ` . Colon `  :  ` is the contains operator. Filter rules are not case sensitive.

The following fields in the operation are eligible for filtering:

  - `  name  ` - The name of the long-running operation
  - `  done  ` - False if the operation is in progress, else true.
  - `  metadata.@type  ` - the type of metadata. For example, the type string for `  RestoreDatabaseMetadata  ` is `  type.googleapis.com/google.spanner.admin.database.v1.RestoreDatabaseMetadata  ` .
  - `  metadata.<field_name>  ` - any field in metadata.value. `  metadata.@type  ` must be specified first, if filtering on metadata fields.
  - `  error  ` - Error associated with the long-running operation.
  - `  response.@type  ` - the type of response.
  - `  response.<field_name>  ` - any field in response.value.

You can combine multiple expressions by enclosing each expression in parentheses. By default, expressions are combined with AND logic. However, you can specify AND, OR, and NOT logic explicitly.

Here are a few examples:

  - `  done:true  ` - The operation is complete.
  - `  (metadata.@type=type.googleapis.com/google.spanner.admin.database.v1.RestoreDatabaseMetadata) AND  ` \\ `  (metadata.source_type:BACKUP) AND  ` \\ `  (metadata.backup_info.backup:backup_howl) AND  ` \\ `  (metadata.name:restored_howl) AND  ` \\ `  (metadata.progress.start_time < \"2018-03-28T14:50:00Z\") AND  ` \\ `  (error:*)  ` - Return operations where:
      - The operation's metadata type is `  RestoreDatabaseMetadata  ` .
      - The database is restored from a backup.
      - The backup name contains "backup\_howl".
      - The restored database's name contains "restored\_howl".
      - The operation started before 2018-03-28T14:50:00Z.
      - The operation resulted in an error.

`  pageSize  `

`  integer  `

Number of operations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

If non-empty, `  pageToken  ` should contain a `  nextPageToken  ` from a previous `  ListDatabaseOperationsResponse  ` to the same `  parent  ` and with the same `  filter  ` .

### Request body

The request body must be empty.

### Response body

The response for `  databaseOperations.list  ` .

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
  &quot;operations&quot;: [
    {
      object (Operation)
    }
  ],
  &quot;nextPageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  operations[]  `

`  object ( Operation  ` )

The list of matching database long-running operations. Each operation's name will be prefixed by the database's name. The operation's metadata field type `  metadata.type_url  ` describes the type of the metadata.

`  nextPageToken  `

`  string  `

`  nextPageToken  ` can be sent in a subsequent `  databaseOperations.list  ` call to fetch more of the matching metadata.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
