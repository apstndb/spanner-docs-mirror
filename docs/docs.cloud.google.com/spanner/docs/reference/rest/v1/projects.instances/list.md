  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.ListInstancesResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Lists all instances in the given project.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{parent=projects/*}/instances  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the project for which a list of instances is requested. Values are of the form `  projects/<project>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instances.list  `

### Query parameters

Parameters

`  pageSize  `

`  integer  `

Number of instances to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

If non-empty, `  pageToken  ` should contain a `  nextPageToken  ` from a previous `  ListInstancesResponse  ` .

`  filter  `

`  string  `

An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:

  - `  name  `
  - `  displayName  `
  - `  labels.key  ` where key is the name of a label

Some examples of using filters are:

  - `  name:*  ` --\> The instance has a name.
  - `  name:Howl  ` --\> The instance's name contains the string "howl".
  - `  name:HOWL  ` --\> Equivalent to above.
  - `  NAME:howl  ` --\> Equivalent to above.
  - `  labels.env:*  ` --\> The instance has the label "env".
  - `  labels.env:dev  ` --\> The instance has the label "env" and the value of the label contains the string "dev".
  - `  name:howl labels.env:dev  ` --\> The instance's name contains "howl" and it has the label "env" with its value containing "dev".

`  instanceDeadline  `

`  string ( Timestamp  ` format)

Deadline used while retrieving metadata for instances. Instances whose metadata cannot be retrieved within this deadline will be added to `  unreachable  ` in `  ListInstancesResponse  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

### Request body

The request body must be empty.

### Response body

The response for `  instances.list  ` .

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
  &quot;instances&quot;: [
    {
      object (Instance)
    }
  ],
  &quot;nextPageToken&quot;: string,
  &quot;unreachable&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instances[]  `

`  object ( Instance  ` )

The list of requested instances.

`  nextPageToken  `

`  string  `

`  nextPageToken  ` can be sent in a subsequent `  instances.list  ` call to fetch more of the matching instances.

`  unreachable[]  `

`  string  `

The list of unreachable instances. It includes the names of instances whose metadata could not be retrieved within `  instanceDeadline  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
