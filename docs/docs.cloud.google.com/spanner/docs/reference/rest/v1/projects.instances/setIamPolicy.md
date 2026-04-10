  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy#body.PATH_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy#body.request_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy#body.response_body)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy#try-it)

Sets the access control policy on an instance resource. Replaces any existing policy.

Authorization requires `  spanner.instances.setIamPolicy  ` on `  resource  ` .

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
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

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
