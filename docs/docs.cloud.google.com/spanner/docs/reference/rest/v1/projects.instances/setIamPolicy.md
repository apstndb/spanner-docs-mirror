  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Sets the access control policy on an instance resource. Replaces any existing policy.

Authorization requires `  spanner.instances.setIamPolicy  ` on `  resource  ` .

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{resource=projects/*/instances/*}:setIamPolicy  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  resource  `

`  string  `

REQUIRED: The Cloud Spanner resource for which the policy is being set. The format is `  projects/<project ID>/instances/<instance ID>  ` for instance resources and `  projects/<project ID>/instances/<instance ID>/databases/<database ID>  ` for databases resources.

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
  &quot;policy&quot;: {
    object (Policy)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  policy  `

`  object ( Policy  ` )

REQUIRED: The complete policy to be applied to the `  resource  ` . The size of the policy is limited to a few 10s of KB. An empty policy is a valid policy but certain Google Cloud services (such as Projects) might reject them.

### Response body

If successful, the response body contains an instance of `  Policy  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
