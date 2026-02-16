  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates an instance and begins preparing it to begin serving. The returned long-running operation can be used to track the progress of preparing the new instance. The instance name is assigned by the caller. If the named instance already exists, `  instances.create  ` returns `  ALREADY_EXISTS  ` .

Immediately upon completion of this request:

  - The instance is readable via the API, with all requested attributes but no allocated resources. Its state is `  CREATING  ` .

Until completion of the returned operation:

  - Cancelling the operation renders the instance immediately unreadable via the API.
  - The instance can be deleted.
  - All other attempts to modify the instance are rejected.

Upon completion of the returned operation:

  - Billing for all successfully-allocated resources begins (some types may have lower than the requested levels).
  - Databases can be created in the instance.
  - The instance's allocated resource levels are readable via the API.
  - The instance's state becomes `  READY  ` .

The returned long-running operation will have a name of the format `  <instance_name>/operations/<operationId>  ` and can be used to track creation of the instance. The metadata field type is `  CreateInstanceMetadata  ` . The response field type is `  Instance  ` , if successful.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*}/instances  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the project in which to create the instance. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instances.create  `

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
  &quot;instanceId&quot;: string,
  &quot;instance&quot;: {
    object (Instance)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instanceId  `

`  string  `

Required. The ID of the instance to create. Valid identifiers are of the form `  [a-z][-a-z0-9]*[a-z0-9]  ` and must be between 2 and 64 characters in length.

`  instance  `

`  object ( Instance  ` )

Required. The instance to create. The name may be omitted, but if specified must be `  <parent>/instances/<instanceId>  ` .

### Response body

If successful, the response body contains a newly created instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
