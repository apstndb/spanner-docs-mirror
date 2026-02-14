  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates an instance configuration and begins preparing it to be used. The returned long-running operation can be used to track the progress of preparing the new instance configuration. The instance configuration name is assigned by the caller. If the named instance configuration already exists, `  instanceConfigs.create  ` returns `  ALREADY_EXISTS  ` .

Immediately after the request returns:

  - The instance configuration is readable via the API, with all requested attributes. The instance configuration's `  reconciling  ` field is set to true. Its state is `  CREATING  ` .

While the operation is pending:

  - Cancelling the operation renders the instance configuration immediately unreadable via the API.
  - Except for deleting the creating resource, all other attempts to modify the instance configuration are rejected.

Upon completion of the returned operation:

  - Instances can be created using the instance configuration.
  - The instance configuration's `  reconciling  ` field becomes false. Its state becomes `  READY  ` .

The returned long-running operation will have a name of the format `  <instance_config_name>/operations/<operationId>  ` and can be used to track creation of the instance configuration. The metadata field type is `  CreateInstanceConfigMetadata  ` . The response field type is `  InstanceConfig  ` , if successful.

Authorization requires `  spanner.instanceConfigs.create  ` permission on the resource `  parent  ` .

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*}/instanceConfigs  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the project in which to create the instance configuration. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instanceConfigs.create  `

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
  &quot;instanceConfigId&quot;: string,
  &quot;instanceConfig&quot;: {
    object (InstanceConfig)
  },
  &quot;validateOnly&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instanceConfigId  `

`  string  `

Required. The ID of the instance configuration to create. Valid identifiers are of the form `  custom-[-a-z0-9]*[a-z0-9]  ` and must be between 2 and 64 characters in length. The `  custom-  ` prefix is required to avoid name conflicts with Google-managed configurations.

`  instanceConfig  `

`  object ( InstanceConfig  ` )

Required. The `  InstanceConfig  ` proto of the configuration to create. `  instanceConfig.name  ` must be `  <parent>/instanceConfigs/<instanceConfigId>  ` . `  instanceConfig.base_config  ` must be a Google-managed configuration name, e.g. /instanceConfigs/us-east1, /instanceConfigs/nam3.

`  validateOnly  `

`  boolean  `

An option to validate, but not actually execute, a request, and provide the same response.

### Response body

If successful, the response body contains a newly created instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
