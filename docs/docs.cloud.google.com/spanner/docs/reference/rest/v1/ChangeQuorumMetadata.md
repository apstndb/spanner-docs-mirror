---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/ChangeQuorumMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ChangeQuorumMetadata
title: ChangeQuorumMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ChangeQuorumMetadata#SCHEMA_REPRESENTATION)
- [ChangeQuorumRequest](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ChangeQuorumMetadata#ChangeQuorumRequest)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ChangeQuorumMetadata#ChangeQuorumRequest.SCHEMA_REPRESENTATION)

Metadata type for the long-running operation returned by [`databases.changequorum`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#google.spanner.admin.database.v1.DatabaseAdmin.ChangeQuorum) .

**JSON representation**

```
{
  "request": {
    object (ChangeQuorumRequest)
  },
  "startTime": string,
  "endTime": string
}
```

| Fields      |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `request`   | `object ( `[`ChangeQuorumRequest`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ChangeQuorumMetadata#ChangeQuorumRequest)` )` The request for [`databases.changequorum`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#google.spanner.admin.database.v1.DatabaseAdmin.ChangeQuorum) .                                                                                       |
| `startTime` | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Time the request was received. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                 |
| `endTime`   | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` If set, the time at which this operation failed or was completed successfully. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |

## ChangeQuorumRequest

The request for [`databases.changequorum`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#google.spanner.admin.database.v1.DatabaseAdmin.ChangeQuorum) .

**JSON representation**

```
{
  "name": string,
  "quorumType": {
    object (QuorumType)
  },
  "etag": string
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                                                                                                          |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`       | `string` Required. Name of the database in which to apply `databases.changequorum` . Values are of the form `projects/<project>/instances/<instance>/databases/<database>` .                                                                                                                                                                                                                             |
| `quorumType` | `object ( `[`QuorumType`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#QuorumType)` )` Required. The type of this quorum.                                                                                                                                                                                                                                   |
| `etag`       | `string` Optional. The etag is the hash of the `QuorumInfo` . The `databases.changequorum` operation is only performed if the etag matches that of the `QuorumInfo` in the current database resource. Otherwise the API returns an `ABORTED` error. The etag is used for optimistic concurrency control as a way to help prevent simultaneous change quorum requests that could create a race condition. |
