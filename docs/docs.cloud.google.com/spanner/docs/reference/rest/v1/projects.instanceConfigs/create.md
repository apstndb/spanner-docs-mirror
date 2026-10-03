---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create
title: 'Method: projects.instanceConfigs.create'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#try-it)

Creates an instance configuration and begins preparing it to be used. The returned long-running operation can be used to track the progress of preparing the new instance configuration. The instance configuration name is assigned by the caller. If the named instance configuration already exists, `instanceConfigs.create` returns `ALREADY_EXISTS` .

Immediately after the request returns:

- The instance configuration is readable via the API, with all requested attributes. The instance configuration's [`reconciling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig.FIELDS.reconciling) field is set to true. Its state is `CREATING` .

While the operation is pending:

- Cancelling the operation renders the instance configuration immediately unreadable via the API.
- Except for deleting the creating resource, all other attempts to modify the instance configuration are rejected.

Upon completion of the returned operation:

- Instances can be created using the instance configuration.
- The instance configuration's [`reconciling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig.FIELDS.reconciling) field becomes false. Its state becomes `READY` .

The returned long-running operation will have a name of the format `<instance_config_name>/operations/<operationId>` and can be used to track creation of the instance configuration. The metadata field type is [`CreateInstanceConfigMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstanceConfigMetadata) . The response field type is [`InstanceConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig) , if successful.

Authorization requires `spanner.instanceConfigs.create` permission on the resource [`parent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#body.PATH_PARAMETERS.parent) .

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{parent=projects/*}/instanceConfigs`

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
<p>Required. The name of the project in which to create the instance configuration. Values are of the form <code>projects/&lt;project&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.instanceConfigs.create</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "instanceConfigId": string,
  "instanceConfig": {
    object (InstanceConfig)
  },
  "validateOnly": boolean
}
```

| Fields             |                                                                                                                                                                                                                                                                                                                                                                                                                             |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instanceConfigId` | `string` Required. The ID of the instance configuration to create. Valid identifiers are of the form `custom-[-a-z0-9]*[a-z0-9]` and must be between 2 and 64 characters in length. The `custom-` prefix is required to avoid name conflicts with Google-managed configurations.                                                                                                                                            |
| `instanceConfig`   | `object ( `[`InstanceConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig)` )` Required. The `InstanceConfig` proto of the configuration to create. `instanceConfig.name` must be `<parent>/instanceConfigs/<instanceConfigId>` . `instanceConfig.base_config` must be a Google-managed configuration name, e.g. /instanceConfigs/us-east1, /instanceConfigs/nam3. |
| `validateOnly`     | `boolean` An option to validate, but not actually execute, a request, and provide the same response.                                                                                                                                                                                                                                                                                                                        |

### Response body

If successful, the response body contains a newly created instance of [`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
