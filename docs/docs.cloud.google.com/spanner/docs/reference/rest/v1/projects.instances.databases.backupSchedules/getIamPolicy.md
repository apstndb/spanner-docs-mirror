  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Gets the access control policy for a database or backup resource. Returns an empty policy if a database or backup exists but does not have a policy set.

Authorization requires `  spanner.databases.getIamPolicy  ` permission on `  resource  ` . For backups, authorization requires `  spanner.backups.getIamPolicy  ` permission on `  resource  ` . For backup schedules, authorization requires `  spanner.backupSchedules.getIamPolicy  ` permission on `  resource  ` .

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{resource=projects/*/instances/*/databases/*/backupSchedules/*}:getIamPolicy  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  resource  `

`  string  `

REQUIRED: The Cloud Spanner resource for which the policy is being retrieved. The format is `  projects/<project ID>/instances/<instance ID>  ` for instance resources and `  projects/<project ID>/instances/<instance ID>/databases/<database ID>  ` for database resources.

### Request body

The request body contains data with the following structure:

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
  &quot;options&quot;: {
    object (GetPolicyOptions)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  options  `

`  object ( GetPolicyOptions  ` )

OPTIONAL: A `  GetPolicyOptions  ` object for specifying options to `  backupSchedules.getIamPolicy  ` .

### Response body

If successful, the response body contains an instance of `  Policy  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
