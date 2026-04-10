  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions#body.PATH_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions#body.request_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions#body.response_body)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions#try-it)

Returns permissions that the caller has on the specified database or backup resource.

Attempting this RPC on a non-existent Cloud Spanner database will result in a NOT\_FOUND error if the user has `  spanner.databases.list  ` permission on the containing Cloud Spanner instance. Otherwise returns an empty set of permissions. Calling this method on a backup that does not exist will result in a NOT\_FOUND error if the user has `  spanner.backups.list  ` permission on the containing instance. Calling this method on a backup schedule that does not exist will result in a NOT\_FOUND error if the user has `  spanner.backupSchedules.list  ` permission on the containing database.

### HTTP request

Choose a location:

global

europe-west8

me-central2

us-central1

us-central2

us-east1

us-east4

us-east5

us-south1

us-west1

us-west2

us-west3

us-west4

us-west8

us-east7

  
`  POST https://spanner.googleapis.com/v1/{resource=projects/*/instances/*/backups/*}:testIamPermissions  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  resource  `

`  string  `

REQUIRED: The Cloud Spanner resource for which permissions are being tested. The format is `  projects/<project ID>/instances/<instance ID>  ` for instance resources and `  projects/<project ID>/instances/<instance ID>/databases/<database ID>  ` for database resources.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;permissions&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  permissions[]  `

`  string  `

REQUIRED: The set of permissions to check for 'resource'. Permissions with wildcards (such as '\*', 'spanner.\*', 'spanner.instances.\*') are not allowed.

### Response body

If successful, the response body contains an instance of `  TestIamPermissionsResponse  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
