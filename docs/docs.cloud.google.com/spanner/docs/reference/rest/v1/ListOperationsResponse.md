---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/ListOperationsResponse
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ListOperationsResponse
title: ListOperationsResponse
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ListOperationsResponse#SCHEMA_REPRESENTATION)

The response message for [`Operations.ListOperations`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.operations/list#google.longrunning.Operations.ListOperations) .

**JSON representation**

```
{
  "operations": [
    {
      object (Operation)
    }
  ],
  "nextPageToken": string,
  "unreachable": [
    string
  ]
}
```

| Fields          |                                                                                                                                                                                                                                                 |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `operations[]`  | `object ( `[`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation)` )` A list of operations that matches the specified filter in the request.                                 |
| `nextPageToken` | `string` The standard List next-page token.                                                                                                                                                                                                     |
| `unreachable[]` | `string` Unordered list. Unreachable resources. Populated when the request sets `ListOperationsRequest.return_partial_success` and reads across collections. For example, when attempting to list all resources across all supported locations. |
