---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/commit
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/commit
title: 'MCP Tools Reference: spanner.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Tool: `commit`

Commit a transaction in a given session.

- If commit is finalizing the result of a DML statement then commit request should include latest precommit_token returned by execute_sql tool.
- If response to commit includes another precommit_token then issue another commit call to finalize the transaction with the latest precommit_token.

The following sample demonstrate how to use `curl` to invoke the `commit` MCP tool.

**Curl Request**

```
curl --location 'https://spanner.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
  "method": "tools/call",
  "params": {
    "name": "commit",
    "arguments": {
      // provide these details according to the tool's MCP specification
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}'
```

## Input Schema

The request for `Commit` .

### CommitRequest

**JSON representation**

```
{
  "session": string,
  "transactionId": string,
  "precommitToken": {
    object (MultiplexedSessionPrecommitToken)
  }
}
```

| Fields           |                                                                                                                                                                                                                                                                                                                                                                                 |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `session`        | `string` Required. The session in which the transaction to be committed is running.                                                                                                                                                                                                                                                                                             |
| `transactionId`  | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Required. The transaction in which to commit. Commit a previously-started transaction. A base64-encoded string.                                                                                                                                                                          |
| `precommitToken` | `object ( `[`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/execute_sql#Output.Schema.MultiplexedSessionPrecommitToken)` )` Required. You must include the precommit token with the highest sequence number received in this transaction attempt. Failing to do so results in a `FailedPrecondition` error. |

### MultiplexedSessionPrecommitToken

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

## Output Schema

The response for `Commit` .

### CommitResponse

**JSON representation**

```
{
  "commitTimestamp": string,
  "commitStats": {
    object (CommitStats)
  },
  "snapshotTimestamp": string,

  // Union field MultiplexedSessionRetry can be only one of the following:
  "precommitToken": {
    object (MultiplexedSessionPrecommitToken)
  }
  // End of list of possible types for union field MultiplexedSessionRetry.
}
```

| Fields                                                                                                                                                                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `commitTimestamp`                                                                                                                                                        | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The Cloud Spanner timestamp at which the transaction committed. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                                                                |
| `commitStats`                                                                                                                                                            | `object ( `[`CommitStats`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/commit#Output.Schema.CommitStats)` )` The statistics about this `Commit` . Not returned by default. For more information, see `CommitRequest.return_commit_stats` .                                                                                                                                                                                                                                                                                                       |
| `snapshotTimestamp`                                                                                                                                                      | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` If `TransactionOptions.isolation_level` is set to `IsolationLevel.REPEATABLE_READ` , then the snapshot timestamp is the timestamp at which all reads in the transaction ran. This timestamp is never returned. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| Union field `MultiplexedSessionRetry` . You must examine and retry the commit if the following is populated. `MultiplexedSessionRetry` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `precommitToken`                                                                                                                                                         | `object ( `[`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/execute_sql#Output.Schema.MultiplexedSessionPrecommitToken)` )` If specified, transaction has not committed yet. You must retry the commit with the new precommit token.                                                                                                                                                                                                                                                                             |

### Timestamp

**JSON representation**

```
{
  "seconds": string,
  "nanos": integer
}
```

| Fields    |                                                                                                                                                                                                                                                                                                                      |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `seconds` | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).                            |
| `nanos`   | `integer` Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive. |

### CommitStats

**JSON representation**

```
{
  "mutationCount": string
}
```

| Fields          |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mutationCount` | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` The total number of mutations for the transaction. Knowing the `mutation_count` value can help you maximize the number of mutations in a transaction and minimize the number of API round trips. You can also monitor this value to prevent transactions from exceeding the system [limit](https://cloud.google.com/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . If the number of mutations exceeds the limit, the server returns [INVALID_ARGUMENT](https://cloud.google.com/spanner/docs/reference/rest/v1/Code#ENUM_VALUES.INVALID_ARGUMENT) . |

### MultiplexedSessionPrecommitToken

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

### Tool Annotations

Destructive Hint: ✅ \| Idempotent Hint: ❌ \| Read Only Hint: ❌ \| Open World Hint: ❌
