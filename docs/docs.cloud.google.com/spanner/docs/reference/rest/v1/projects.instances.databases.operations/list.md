  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Lists operations that match the specified filter in the request. If the server doesn't support this method, it returns `  UNIMPLEMENTED  ` .

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*/operations}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

The name of the operation's parent resource.

### Query parameters

Parameters

`  filter  `

`  string  `

The standard list filter.

`  pageSize  `

`  integer  `

The standard list page size.

`  pageToken  `

`  string  `

The standard list page token.

`  returnPartialSuccess  `

`  boolean  `

When set to `  true  ` , operations that are reachable are returned as normal, and those that are unreachable are returned in the `  ListOperationsResponse.unreachable  ` field.

This can only be `  true  ` when reading across collections. For example, when `  parent  ` is set to `  "projects/example/locations/-"  ` .

This field is not supported by default and will result in an `  UNIMPLEMENTED  ` error if set unless explicitly documented otherwise in service or product specific documentation.

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of `  ListOperationsResponse  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
