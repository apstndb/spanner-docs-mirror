---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/MultiplexedSessionPrecommitToken
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MultiplexedSessionPrecommitToken
title: MultiplexedSessionPrecommitToken
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MultiplexedSessionPrecommitToken#SCHEMA_REPRESENTATION)

When a read-write transaction is executed on a multiplexed session, this precommit token is sent back to the client as a part of the [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Transaction) message in the `sessions.beginTransaction` response and also as a part of the [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) and [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet) responses.

**JSON representation**

```
{
  "precommitToken": string,
  "seqNum": integer
}
```

| Fields           |                                                                                                                                                                                                                 |
|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `precommitToken` | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Opaque precommit token. A base64-encoded string.                                                                         |
| `seqNum`         | `integer` An incrementing seq number is generated on every precommit token that is returned. Clients should remember the precommit token with the highest sequence number from the current transaction attempt. |
