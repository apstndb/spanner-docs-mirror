This page describes change streams in Spanner for GoogleSQL-dialect databases and PostgreSQL-dialect databases, including:

  - The split-based partitioning model
  - The format and content of change stream records
  - The low-level syntax used to query those records
  - An example of the query workflow

You use the Spanner API to [query change streams directly](/spanner/docs/change-streams/details#query) . Applications that instead [use Dataflow to read change stream data](/spanner/docs/change-streams/use-dataflow) don't need to work directly with the data model described here.

For a broader introductory guide to change streams, see [Change streams overview](/spanner/docs/change-streams) .

## Change stream partitions

When a change occurs on a table that is watched by a change stream, Spanner writes a corresponding change stream record in the database, synchronously in the same transaction as the data change. This means that if the transaction succeeds, Spanner has also successfully captured and persisted the change. Internally, Spanner co-locates the change stream record and the data change so that they are processed by the same server to minimize write overhead.

As part of the DML to a particular split, Spanner appends the write to the corresponding change stream data split in the same transaction. Because of this colocation, change streams don't add extra coordination across serving resources, which minimizes the transaction commit overhead.

Spanner scales by dynamically splitting and merging data based on database load and size, and distributing splits across serving resources.

To enable change streams writes and reads to scale, Spanner splits and merges the internal change stream storage along with the database data, automatically avoiding [hotspots](/spanner/docs/schema-design) . To support reading change stream records in near real-time as database writes scale, the Spanner API is designed for a change stream to be queried concurrently using change stream partitions. Change stream partitions map to change stream data splits that contain the change stream records. A change stream's partitions change dynamically over time and are correlated to how Spanner dynamically splits and merges the database data.

A change stream partition contains records for an immutable key range for a specific time range. Any change stream partition can split into one or more change stream partitions, or be merged with other change stream partitions. When these split or merge events happen, child partitions are created to capture the changes for their respective immutable key ranges for the next time range. In addition to data change records, a change stream query returns child partition records to notify readers of new change stream partitions that need to be queried, as well as heartbeat records to indicate forward progress when no writes have occurred recently.

When querying a particular change stream partition, the change records are returned in commit timestamp order. Each change record is returned exactly once. Across change stream partitions, change record ordering is not guaranteed. Change records for a particular primary key are returned only on one partition for a particular time range.

Due to the parent-child partition lineage, in order to process changes for a particular key in commit timestamp order, records returned from child partitions should be processed only after records from all parent partitions have been processed.

## Change stream read functions and query syntax

### GoogleSQL

To query change streams, use the [`  ExecuteStreamingSql  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql) API. Spanner automatically creates a special read function along with the change stream. The read function provides access to the change stream's records. The read function naming convention is `  READ_ change_stream_name  ` .

Assuming a change stream `  SingersNameStream  ` exists in the database, the query syntax for GoogleSQL is the following:

``` text
SELECT ChangeRecord
FROM READ_SingersNameStream (
    start_timestamp,
    end_timestamp,
    partition_token,
    heartbeat_milliseconds,
    read_options
)
```

The read function accepts the following arguments:

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Argument name</th>
<th>Type</th>
<th>Required?</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         start_timestamp        </code></td>
<td><code dir="ltr" translate="no">         TIMESTAMP        </code></td>
<td>Required</td>
<td>Specifies that records with <code dir="ltr" translate="no">         commit_timestamp        </code> greater than or equal to <code dir="ltr" translate="no">         start_timestamp        </code> should be returned. The value must be within the change stream retention period, and should be less than or equal to the current time, and greater than or equal to the timestamp of the change stream's creation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         end_timestamp        </code></td>
<td><code dir="ltr" translate="no">         TIMESTAMP        </code></td>
<td>Optional (Default: <code dir="ltr" translate="no">         NULL        </code> )</td>
<td>Specifies that records with a <code dir="ltr" translate="no">         commit_timestamp        </code> less than or equal to <code dir="ltr" translate="no">         end_timestamp        </code> should be returned. The value must be within the change stream retention period and greater or equal than the <code dir="ltr" translate="no">         start_timestamp        </code> . The query finishes either after returning all <code dir="ltr" translate="no">         ChangeRecords        </code> up to the <code dir="ltr" translate="no">         end_timestamp        </code> or when you terminate the connection. If <code dir="ltr" translate="no">         end_timestamp        </code> is set to <code dir="ltr" translate="no">         NULL        </code> or isn't specified, the query continues execution until all <code dir="ltr" translate="no">         ChangeRecords        </code> are returned or until you terminate the connection.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         partition_token        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Optional (Default: <code dir="ltr" translate="no">         NULL        </code> )</td>
<td>Specifies which change stream partition to query, based on the content of <a href="#child-partitions-records">child partitions records</a> . If <code dir="ltr" translate="no">         NULL        </code> or not specified, this means the reader is querying the change stream for the first time, and has not obtained any specific partition tokens to query from.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         heartbeat_milliseconds        </code></td>
<td><code dir="ltr" translate="no">         INT64        </code></td>
<td>Required</td>
<td>Determines how frequently a heartbeat <code dir="ltr" translate="no">         ChangeRecord        </code> is returned in case there are no transactions committed in this partition.<br />
<br />
The value must be between <code dir="ltr" translate="no">         1,000        </code> (one second) and <code dir="ltr" translate="no">         300,000        </code> (five minutes).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         read_options        </code></td>
<td><code dir="ltr" translate="no">         ARRAY        </code></td>
<td>Optional (Default: <code dir="ltr" translate="no">         NULL        </code> )</td>
<td>Adds read options reserved for future use. The only permitted value is <code dir="ltr" translate="no">         NULL        </code> .</td>
</tr>
</tbody>
</table>

We recommend making a helper method for building the text of the read function query and binding parameters to it, as shown in the following example.

### Java

``` text
    private static final String SINGERS_NAME_STREAM_QUERY_TEMPLATE =
    "SELECT ChangeRecord FROM READ_SingersNameStream"
        + "("
        + "   start_timestamp => @startTimestamp,"
        + "   end_timestamp => @endTimestamp,"
        + "   partition_token => @partitionToken,"
        + "   heartbeat_milliseconds => @heartbeatMillis"
        + ")";

    // Helper method to conveniently create change stream query texts and
    // bind parameters.
    public static Statement getChangeStreamQuery(
          String partitionToken,
          Timestamp startTimestamp,
          Timestamp endTimestamp,
          long heartbeatMillis) {
      return Statement.newBuilder(SINGERS_NAME_STREAM_QUERY_TEMPLATE)
                        .bind("startTimestamp")
                        .to(startTimestamp)
                        .bind("endTimestamp")
                        .to(endTimestamp)
                        .bind("partitionToken")
                        .to(partitionToken)
                        .bind("heartbeatMillis")
                        .to(heartbeatMillis)
                        .build();
    }
    
```

### PostgreSQL

To query change streams, use the [`  ExecuteStreamingSql  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql) API. Spanner automatically creates a special read function along with the change stream. The read function provides access to the change stream's records. The read function naming convention is `  spanner.read_json_ change_stream_name  ` .

Assuming a change stream `  SingersNameStream  ` exists in the database, the query syntax for PostgreSQL is the following:

``` text
SELECT *
FROM "spanner"."read_json_SingersNameStream" (
    start_timestamp,
    end_timestamp,
    partition_token,
    heartbeat_milliseconds,
    null
)
```

The read function accepts the following arguments:

<table>
<thead>
<tr class="header">
<th>Argument name</th>
<th>Type</th>
<th>Required?</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         start_timestamp        </code></td>
<td><code dir="ltr" translate="no">         timestamp with time zone        </code></td>
<td>Required</td>
<td>Specifies that change records with <code dir="ltr" translate="no">         commit_timestamp        </code> greater than or equal to <code dir="ltr" translate="no">         start_timestamp        </code> should be returned. The value must be within the change stream retention period, and should be less than or equal to the current time, and greater than or equal to the timestamp of the change stream's creation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         end_timestamp        </code></td>
<td><code dir="ltr" translate="no">         timestamp with timezone        </code></td>
<td>Optional (Default: <code dir="ltr" translate="no">         NULL        </code> )</td>
<td>Specifies that change records with <code dir="ltr" translate="no">         commit_timestamp        </code> less than or equal to <code dir="ltr" translate="no">         end_timestamp        </code> should be returned. The value must be within the change stream retention period and greater or equal than the <code dir="ltr" translate="no">         start_timestamp        </code> . The query finishes either after returning all change records up to the <code dir="ltr" translate="no">         end_timestamp        </code> or until you terminate the connection. If <code dir="ltr" translate="no">         NULL        </code> , the query continues execution until all change records are returned or until you terminate the connection.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         partition_token        </code></td>
<td><code dir="ltr" translate="no">         text        </code></td>
<td>Optional (Default: <code dir="ltr" translate="no">         NULL        </code> )</td>
<td>Specifies which change stream partition to query, based on the content of <a href="#child-partitions-records">child partitions records</a> . If <code dir="ltr" translate="no">         NULL        </code> or not specified, this means the reader is querying the change stream for the first time, and has not obtained any specific partition tokens to query from.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         heartbeat_milliseconds        </code></td>
<td><code dir="ltr" translate="no">         bigint        </code></td>
<td>Required</td>
<td>Determines how frequently a heartbeat <code dir="ltr" translate="no">         ChangeRecord        </code> is returned when there are no transactions committed in this partition. The value must be between <code dir="ltr" translate="no">         1,000        </code> (one second) and <code dir="ltr" translate="no">         300,000        </code> (five minutes).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         null        </code></td>
<td><code dir="ltr" translate="no">         null        </code></td>
<td>Required</td>
<td>Reserved for future use</td>
</tr>
</tbody>
</table>

We recommend making a helper method for building the text of the read function and binding parameters to it, as shown in the following example.

### Java

``` text
private static final String SINGERS_NAME_STREAM_QUERY_TEMPLATE =
        "SELECT * FROM \"spanner\".\"read_json_SingersNameStream\""
            + "($1, $2, $3, $4, null)";

// Helper method to conveniently create change stream query texts and
// bind parameters.
public static Statement getChangeStreamQuery(
      String partitionToken,
      Timestamp startTimestamp,
      Timestamp endTimestamp,
      long heartbeatMillis) {

  return Statement.newBuilder(SINGERS_NAME_STREAM_QUERY_TEMPLATE)
                    .bind("p1")
                    .to(startTimestamp)
                    .bind("p2")
                    .to(endTimestamp)
                    .bind("p3")
                    .to(partitionToken)
                    .bind("p4")
                    .to(heartbeatMillis)
                    .build();
}
```

## Change streams record format

### GoogleSQL

The change streams read function returns a single `  ChangeRecord  ` column of type `  ARRAY<STRUCT<...>>  ` . In each row, this array always contains a single element.

The array elements have the following type:

``` googlesql
STRUCT <
  data_change_record ARRAY<STRUCT<...>>,
  heartbeat_record ARRAY<STRUCT<...>>,
  child_partitions_record ARRAY<STRUCT<...>>
>
```

There are three fields in this `  STRUCT  ` : `  data_change_record  ` , `  heartbeat_record  ` and `  child_partitions_record  ` , each of type `  ARRAY<STRUCT<...>>  ` . In any row that the change stream read function returns, only one of these three fields contains a value; the others two are empty or `  NULL  ` . These array fields contain, at most, one element.

The following sections examine each of these three record types.

### PostgreSQL

The change streams read function returns a single `  ChangeRecord  ` column of type `  JSON  ` with the following structure:

``` text
{
  "data_change_record" : {},
  "heartbeat_record" : {},
  "child_partitions_record" : {}
}
```

There are three possible keys in this object: `  data_change_record  ` , `  heartbeat_record  ` and `  child_partitions_record  ` , the corresponding value type is `  JSON  ` . In any row that the change stream read function returns, only one of these three keys exists.

The following sections examine each of these three record types.

### Data change records

A data change record contains a set of changes to a table with the same modification type (insert, update, or delete) committed at the same commit timestamp in one change stream partition for the same transaction. Multiple data change records can be returned for the same transaction across multiple change stream partitions.

All data change records have `  commit_timestamp  ` , `  server_transaction_id  ` , and `  record_sequence  ` fields, which together determine the order in the change stream for a stream record. These three fields are sufficient to derive the ordering of changes and provide external consistency.

Note that multiple transactions can have the same commit timestamp if they touch non-overlapping data. The `  server_transaction_id  ` field offers the ability to distinguish which set of changes (potentially across change stream partitions) were issued within the same transaction. Pairing it with the `  record_sequence  ` and `  number_of_records_in_transaction  ` fields lets you buffer and order all the records from a particular transaction, as well.

The fields of a data change record include the following:

### GoogleSQL

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Field</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         commit_timestamp        </code></td>
<td><code dir="ltr" translate="no">         TIMESTAMP        </code></td>
<td>Indicates the timestamp in which the change was committed.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         record_sequence        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates the sequence number for the record within the transaction. Sequence numbers are unique and monotonically increasing (but not necessarily contiguous) within a transaction. Sort the records for the same <code dir="ltr" translate="no">         server_transaction_id        </code> by <code dir="ltr" translate="no">         record_sequence        </code> to reconstruct the ordering of the changes within the transaction. Spanner might optimize this ordering for better performances and it might not always match the original ordering that you provide.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         server_transaction_id        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Provides a globally unique string that represents the transaction in which the change was committed. The value should only be used in the context of processing change stream records and is not correlated with the transaction id in Spanner's API.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         is_last_record_in_transaction_in_partition        </code></td>
<td><code dir="ltr" translate="no">         BOOL        </code></td>
<td>Indicates whether this is the last record for a transaction in the current partition.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         table_name        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Name of the table affected by the change.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         value_capture_type        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td><p>Describes the value capture type that was specified in the change stream configuration when this change was captured.</p>
<p>The value capture type can be one of the following:</p>
<ul>
<li><code dir="ltr" translate="no">           OLD_AND_NEW_VALUES          </code></li>
<li><code dir="ltr" translate="no">           NEW_ROW          </code></li>
<li><code dir="ltr" translate="no">           NEW_VALUES          </code></li>
<li><code dir="ltr" translate="no">           NEW_ROW_AND_OLD_VALUES          </code></li>
</ul>
<p>By default, it is <code dir="ltr" translate="no">          OLD_AND_NEW_VALUES         </code> . For more information, see <a href="/spanner/docs/change-streams#value-capture-type">value capture types</a> .</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         column_types        </code></td>
<td><div class="sourceCode" id="cb1" dir="ltr" data-is-upgraded="" translate="no"><pre class="sourceCode json"><code class="sourceCode json"><span id="cb1-1"><a href="#cb1-1"></a><span class="ot">[</span></span>
<span id="cb1-2"><a href="#cb1-2"></a>  <span class="fu">{</span></span>
<span id="cb1-3"><a href="#cb1-3"></a>      <span class="dt">&quot;name&quot;</span><span class="fu">:</span> <span class="st">&quot;STRING&quot;</span><span class="fu">,</span></span>
<span id="cb1-4"><a href="#cb1-4"></a>      <span class="dt">&quot;type&quot;</span><span class="fu">:</span> <span class="fu">{</span></span>
<span id="cb1-5"><a href="#cb1-5"></a>        <span class="dt">&quot;code&quot;</span><span class="fu">:</span> <span class="st">&quot;STRING&quot;</span></span>
<span id="cb1-6"><a href="#cb1-6"></a>      <span class="fu">},</span></span>
<span id="cb1-7"><a href="#cb1-7"></a>      <span class="dt">&quot;is_primary_key&quot;</span><span class="fu">:</span> <span class="er">BOOLEAN</span></span>
<span id="cb1-8"><a href="#cb1-8"></a>      <span class="st">&quot;ordinal_position&quot;</span><span class="er">:</span> <span class="er">NUMBER</span></span>
<span id="cb1-9"><a href="#cb1-9"></a>    <span class="fu">}</span><span class="ot">,</span></span>
<span id="cb1-10"><a href="#cb1-10"></a>    <span class="er">...</span></span>
<span id="cb1-11"><a href="#cb1-11"></a><span class="ot">]</span></span></code></pre></div></td>
<td>Indicates the name of the column, the column type, whether it is a primary key, and the position of the column as defined in the schema ( <code dir="ltr" translate="no">         ordinal_position        </code> ). The first column of a table in the schema would have an ordinal position of <code dir="ltr" translate="no">         1        </code> . The column type may be nested for array columns. The format matches the type structure described in the <a href="/spanner/docs/reference/rest/v1/Type">Spanner API reference</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         mods        </code></td>
<td><div class="sourceCode" id="cb2" dir="ltr" data-is-upgraded="" translate="no"><pre class="sourceCode json"><code class="sourceCode json"><span id="cb2-1"><a href="#cb2-1"></a><span class="ot">[</span></span>
<span id="cb2-2"><a href="#cb2-2"></a>  <span class="fu">{</span></span>
<span id="cb2-3"><a href="#cb2-3"></a>    <span class="dt">&quot;keys&quot;</span><span class="fu">:</span> <span class="fu">{</span><span class="dt">&quot;STRING&quot;</span> <span class="fu">:</span> <span class="st">&quot;STRING&quot;</span><span class="fu">},</span></span>
<span id="cb2-4"><a href="#cb2-4"></a>    <span class="dt">&quot;new_values&quot;</span><span class="fu">:</span> <span class="fu">{</span></span>
<span id="cb2-5"><a href="#cb2-5"></a>      <span class="dt">&quot;STRING&quot;</span> <span class="fu">:</span> <span class="st">&quot;VALUE-TYPE&quot;</span><span class="fu">,</span></span>
<span id="cb2-6"><a href="#cb2-6"></a>      <span class="er">[...]</span></span>
<span id="cb2-7"><a href="#cb2-7"></a>    <span class="fu">},</span></span>
<span id="cb2-8"><a href="#cb2-8"></a>    <span class="dt">&quot;old_values&quot;</span><span class="fu">:</span> <span class="fu">{</span></span>
<span id="cb2-9"><a href="#cb2-9"></a>      <span class="dt">&quot;STRING&quot;</span> <span class="fu">:</span> <span class="st">&quot;VALUE-TYPE&quot;</span><span class="fu">,</span></span>
<span id="cb2-10"><a href="#cb2-10"></a>      <span class="er">[...]</span></span>
<span id="cb2-11"><a href="#cb2-11"></a>    <span class="fu">},</span></span>
<span id="cb2-12"><a href="#cb2-12"></a>  <span class="fu">}</span><span class="ot">,</span></span>
<span id="cb2-13"><a href="#cb2-13"></a>  <span class="ot">[</span><span class="er">...</span><span class="ot">]</span></span>
<span id="cb2-14"><a href="#cb2-14"></a><span class="ot">]</span></span></code></pre></div></td>
<td>Describes the changes that were made, including the primary key values, the old values, and the new values of the changed or tracked columns. The availability and content of the old and new values depends on the configured <code dir="ltr" translate="no">         value_capture_type        </code> . The <code dir="ltr" translate="no">         new_values        </code> and <code dir="ltr" translate="no">         old_values        </code> fields only contain the non-key columns.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         mod_type        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Describes the type of change. One of <code dir="ltr" translate="no">         INSERT        </code> , <code dir="ltr" translate="no">         UPDATE        </code> , or <code dir="ltr" translate="no">         DELETE        </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         number_of_records_in_transaction        </code></td>
<td><code dir="ltr" translate="no">         INT64        </code></td>
<td>Indicates the number of data change records that are part of this transaction across all change stream partitions.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         number_of_partitions_in_transaction        </code></td>
<td><code dir="ltr" translate="no">         INT64        </code></td>
<td>Indicates the number of partitions that return data change records for this transaction.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         transaction_tag        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates the <a href="/spanner/docs/introspection/troubleshooting-with-tags#transaction_tags">Transaction tag</a> associated with this transaction.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         is_system_transaction        </code></td>
<td><code dir="ltr" translate="no">         BOOL        </code></td>
<td>Indicates whether the transaction is a system transaction.</td>
</tr>
</tbody>
</table>

### PostgreSQL

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Field</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         commit_timestamp        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates the timestamp at which the change was committed.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         record_sequence        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates the sequence number for the record within the transaction. Sequence numbers are unique and monotonically increasing (but not necessarily contiguous) within a transaction. Sort the records for the same <code dir="ltr" translate="no">         server_transaction_id        </code> by <code dir="ltr" translate="no">         record_sequence        </code> to reconstruct the ordering of the changes within the transaction.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         server_transaction_id        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Provides a globally unique string that represents the transaction in which the change was committed. The value should only be used in the context of processing change stream records and is not correlated with the transaction id in Spanner's API</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         is_last_record_in_transaction_in_partition        </code></td>
<td><code dir="ltr" translate="no">         BOOLEAN        </code></td>
<td>Indicates whether this is the last record for a transaction in the current partition.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         table_name        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates the name of the table affected by the change.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         value_capture_type        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td><p>Describes the value capture type that was specified in the change stream configuration when this change was captured.</p>
<p>The value capture type can be one of the following:</p>
<ul>
<li><code dir="ltr" translate="no">           OLD_AND_NEW_VALUES          </code></li>
<li><code dir="ltr" translate="no">           NEW_ROW          </code></li>
<li><code dir="ltr" translate="no">           NEW_VALUES          </code></li>
<li><code dir="ltr" translate="no">           NEW_ROW_AND_OLD_VALUES          </code></li>
</ul>
<p>By default, it is <code dir="ltr" translate="no">          OLD_AND_NEW_VALUES         </code> . For more information, see <a href="/spanner/docs/change-streams#value-capture-type">value capture types</a> .</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         column_types        </code></td>
<td><div class="sourceCode" id="cb1" dir="ltr" data-is-upgraded="" translate="no"><pre class="sourceCode json"><code class="sourceCode json"><span id="cb1-1"><a href="#cb1-1"></a><span class="ot">[</span></span>
<span id="cb1-2"><a href="#cb1-2"></a>  <span class="fu">{</span></span>
<span id="cb1-3"><a href="#cb1-3"></a>      <span class="dt">&quot;name&quot;</span><span class="fu">:</span> <span class="st">&quot;STRING&quot;</span><span class="fu">,</span></span>
<span id="cb1-4"><a href="#cb1-4"></a>      <span class="dt">&quot;type&quot;</span><span class="fu">:</span> <span class="fu">{</span></span>
<span id="cb1-5"><a href="#cb1-5"></a>        <span class="dt">&quot;code&quot;</span><span class="fu">:</span> <span class="st">&quot;STRING&quot;</span></span>
<span id="cb1-6"><a href="#cb1-6"></a>      <span class="fu">},</span></span>
<span id="cb1-7"><a href="#cb1-7"></a>      <span class="dt">&quot;is_primary_key&quot;</span><span class="fu">:</span> <span class="er">BOOLEAN</span></span>
<span id="cb1-8"><a href="#cb1-8"></a>      <span class="st">&quot;ordinal_position&quot;</span><span class="er">:</span> <span class="er">NUMBER</span></span>
<span id="cb1-9"><a href="#cb1-9"></a>    <span class="fu">}</span><span class="ot">,</span></span>
<span id="cb1-10"><a href="#cb1-10"></a>    <span class="er">...</span></span>
<span id="cb1-11"><a href="#cb1-11"></a><span class="ot">]</span></span></code></pre></div></td>
<td>Indicates the name of the column, the column type, whether it's a primary key, and the position of the column as defined in the schema ( <code dir="ltr" translate="no">         ordinal_position        </code> ). The first column of a table in the schema would have an ordinal position of <code dir="ltr" translate="no">         1        </code> . The column type may be nested for array columns. The format matches the type structure described in the <a href="/spanner/docs/reference/rest/v1/Type">Spanner API reference</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         mods        </code></td>
<td><div class="sourceCode" id="cb2" dir="ltr" data-is-upgraded="" translate="no"><pre class="sourceCode json"><code class="sourceCode json"><span id="cb2-1"><a href="#cb2-1"></a><span class="ot">[</span></span>
<span id="cb2-2"><a href="#cb2-2"></a>  <span class="fu">{</span></span>
<span id="cb2-3"><a href="#cb2-3"></a>    <span class="dt">&quot;keys&quot;</span><span class="fu">:</span> <span class="fu">{</span><span class="dt">&quot;STRING&quot;</span> <span class="fu">:</span> <span class="st">&quot;STRING&quot;</span><span class="fu">},</span></span>
<span id="cb2-4"><a href="#cb2-4"></a>    <span class="dt">&quot;new_values&quot;</span><span class="fu">:</span> <span class="fu">{</span></span>
<span id="cb2-5"><a href="#cb2-5"></a>      <span class="dt">&quot;STRING&quot;</span> <span class="fu">:</span> <span class="st">&quot;VALUE-TYPE&quot;</span><span class="fu">,</span></span>
<span id="cb2-6"><a href="#cb2-6"></a>      <span class="er">[...]</span></span>
<span id="cb2-7"><a href="#cb2-7"></a>    <span class="fu">},</span></span>
<span id="cb2-8"><a href="#cb2-8"></a>    <span class="dt">&quot;old_values&quot;</span><span class="fu">:</span> <span class="fu">{</span></span>
<span id="cb2-9"><a href="#cb2-9"></a>      <span class="dt">&quot;STRING&quot;</span> <span class="fu">:</span> <span class="st">&quot;VALUE-TYPE&quot;</span><span class="fu">,</span></span>
<span id="cb2-10"><a href="#cb2-10"></a>      <span class="er">[...]</span></span>
<span id="cb2-11"><a href="#cb2-11"></a>    <span class="fu">},</span></span>
<span id="cb2-12"><a href="#cb2-12"></a>  <span class="fu">}</span><span class="ot">,</span></span>
<span id="cb2-13"><a href="#cb2-13"></a>  <span class="ot">[</span><span class="er">...</span><span class="ot">]</span></span>
<span id="cb2-14"><a href="#cb2-14"></a><span class="ot">]</span></span></code></pre></div></td>
<td>Describes the changes that were made, including the primary key values, the old values, and the new values of the changed or tracked columns. The availability and content of the old and new values depends on the configured <code dir="ltr" translate="no">         value_capture_type        </code> . The <code dir="ltr" translate="no">         new_values        </code> and <code dir="ltr" translate="no">         old_values        </code> fields only contain the non-key columns.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         mod_type        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Describes the type of change. One of <code dir="ltr" translate="no">         INSERT        </code> , <code dir="ltr" translate="no">         UPDATE        </code> , or <code dir="ltr" translate="no">         DELETE        </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         number_of_records_in_transaction        </code></td>
<td><code dir="ltr" translate="no">         INT64        </code></td>
<td>Indicates the number of data change records that are part of this transaction across all change stream partitions.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         number_of_partitions_in_transaction        </code></td>
<td><code dir="ltr" translate="no">         NUMBER        </code></td>
<td>Indicates the number of partitions that return data change records for this transaction.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         transaction_tag        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates the <a href="/spanner/docs/introspection/troubleshooting-with-tags#transaction_tags">Transaction tag</a> associated with this transaction.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         is_system_transaction        </code></td>
<td><code dir="ltr" translate="no">         BOOLEAN        </code></td>
<td>Indicates whether the transaction is a system transaction.</td>
</tr>
</tbody>
</table>

#### Example data change record

A pair of example data change records follow. They describe a single transaction where there is a transfer between two accounts. The two accounts are in separate change stream partitions.

``` text
"data_change_record": {
  "commit_timestamp": "2022-09-27T12:30:00.123456Z",
  // record_sequence is unique and monotonically increasing within a
  // transaction, across all partitions.
  "record_sequence": "00000000",
  "server_transaction_id": "6329047911",
  "is_last_record_in_transaction_in_partition": true,

  "table_name": "AccountBalance",
  "column_types": [
    {
      "name": "AccountId",
      "type": {"code": "STRING"},
      "is_primary_key": true,
      "ordinal_position": 1
    },
    {
      "name": "LastUpdate",
      "type": {"code": "TIMESTAMP"},
      "is_primary_key": false,
      "ordinal_position": 2
    },
    {
       "name": "Balance",
       "type": {"code": "INT"},
       "is_primary_key": false,
       "ordinal_position": 3
    }
  ],
  "mods": [
    {
      "keys": {"AccountId": "Id1"},
      "new_values": {
        "LastUpdate": "2022-09-27T12:30:00.123456Z",
        "Balance": 1000
      },
      "old_values": {
        "LastUpdate": "2022-09-26T11:28:00.189413Z",
        "Balance": 1500
      },
    }
  ],
  "mod_type": "UPDATE", // options are INSERT, UPDATE, DELETE
  "value_capture_type": "OLD_AND_NEW_VALUES",
  "number_of_records_in_transaction": 2,
  "number_of_partitions_in_transaction": 2,
  "transaction_tag": "app=banking,env=prod,action=update",
  "is_system_transaction": false,
}
```

``` text
"data_change_record": {
  "commit_timestamp": "2022-09-27T12:30:00.123456Z",
  "record_sequence": "00000001",
  "server_transaction_id": "6329047911",
  "is_last_record_in_transaction_in_partition": true,

  "table_name": "AccountBalance",
  "column_types": [
    {
      "name": "AccountId",
      "type": {"code": "STRING"},
      "is_primary_key": true,
      "ordinal_position": 1
    },
    {
      "name": "LastUpdate",
      "type": {"code": "TIMESTAMP"},
      "is_primary_key": false,
      "ordinal_position": 2
    },
    {
      "name": "Balance",
      "type": {"code": "INT"},
      "is_primary_key": false,
      "ordinal_position": 3
    }
  ],
  "mods": [
    {
      "keys": {"AccountId": "Id2"},
      "new_values": {
        "LastUpdate": "2022-09-27T12:30:00.123456Z",
        "Balance": 2000
      },
      "old_values": {
        "LastUpdate": "2022-01-20T11:25:00.199915Z",
        "Balance": 1500
      },
    },
    ...
  ],
  "mod_type": "UPDATE", // options are INSERT, UPDATE, DELETE
  "value_capture_type": "OLD_AND_NEW_VALUES",
  "number_of_records_in_transaction": 2,
  "number_of_partitions_in_transaction": 2,
  "transaction_tag": "app=banking,env=prod,action=update",
  "is_system_transaction": false,
}
```

The following data change record is an example of a record with the value capture type `  NEW_VALUES  ` . Note that only new values are populated. Only the `  LastUpdate  ` column was modified, so only that column was returned.

``` text
"data_change_record": {
  "commit_timestamp": "2022-09-27T12:30:00.123456Z",
  // record_sequence is unique and monotonically increasing within a
  // transaction, across all partitions.
  "record_sequence": "00000000",
  "server_transaction_id": "6329047911",
  "is_last_record_in_transaction_in_partition": true,
  "table_name": "AccountBalance",
  "column_types": [
    {
      "name": "AccountId",
      "type": {"code": "STRING"},
      "is_primary_key": true,
      "ordinal_position": 1
    },
    {
      "name": "LastUpdate",
      "type": {"code": "TIMESTAMP"},
      "is_primary_key": false,
      "ordinal_position": 2
    }
  ],
  "mods": [
    {
      "keys": {"AccountId": "Id1"},
      "new_values": {
        "LastUpdate": "2022-09-27T12:30:00.123456Z"
      },
      "old_values": {}
    }
  ],
  "mod_type": "UPDATE", // options are INSERT, UPDATE, DELETE
  "value_capture_type": "NEW_VALUES",
  "number_of_records_in_transaction": 1,
  "number_of_partitions_in_transaction": 1,
  "transaction_tag": "app=banking,env=prod,action=update",
  "is_system_transaction": false
}
```

The following data change record is an example of a record with the value capture type `  NEW_ROW  ` . Only the `  LastUpdate  ` column was modified, but all tracked columns are returned.

``` text
"data_change_record": {
  "commit_timestamp": "2022-09-27T12:30:00.123456Z",
  // record_sequence is unique and monotonically increasing within a
  // transaction, across all partitions.
  "record_sequence": "00000000",
  "server_transaction_id": "6329047911",
  "is_last_record_in_transaction_in_partition": true,

  "table_name": "AccountBalance",
  "column_types": [
    {
      "name": "AccountId",
      "type": {"code": "STRING"},
      "is_primary_key": true,
      "ordinal_position": 1
    },
    {
      "name": "LastUpdate",
      "type": {"code": "TIMESTAMP"},
      "is_primary_key": false,
      "ordinal_position": 2
    },
    {
       "name": "Balance",
       "type": {"code": "INT"},
       "is_primary_key": false,
       "ordinal_position": 3
    }
  ],
  "mods": [
    {
      "keys": {"AccountId": "Id1"},
      "new_values": {
        "LastUpdate": "2022-09-27T12:30:00.123456Z",
        "Balance": 1000
      },
      "old_values": {}
    }
  ],
  "mod_type": "UPDATE", // options are INSERT, UPDATE, DELETE
  "value_capture_type": "NEW_ROW",
  "number_of_records_in_transaction": 1,
  "number_of_partitions_in_transaction": 1,
  "transaction_tag": "app=banking,env=prod,action=update",
  "is_system_transaction": false
}
```

The following data change record is an example of a record with the value capture type `  NEW_ROW_AND_OLD_VALUES  ` . Only the `  LastUpdate  ` column was modified, but all tracked columns are returned. This value capture type captures the new value and old value of `  LastUpdate  ` .

``` text
"data_change_record": {
  "commit_timestamp": "2022-09-27T12:30:00.123456Z",
  // record_sequence is unique and monotonically increasing within a
  // transaction, across all partitions.
  "record_sequence": "00000000",
  "server_transaction_id": "6329047911",
  "is_last_record_in_transaction_in_partition": true,

  "table_name": "AccountBalance",
  "column_types": [
    {
      "name": "AccountId",
      "type": {"code": "STRING"},
      "is_primary_key": true,
      "ordinal_position": 1
    },
    {
      "name": "LastUpdate",
      "type": {"code": "TIMESTAMP"},
      "is_primary_key": false,
      "ordinal_position": 2
    },
    {
       "name": "Balance",
       "type": {"code": "INT"},
       "is_primary_key": false,
       "ordinal_position": 3
    }
  ],
  "mods": [
    {
      "keys": {"AccountId": "Id1"},
      "new_values": {
        "LastUpdate": "2022-09-27T12:30:00.123456Z",
        "Balance": 1000
      },
      "old_values": {
        "LastUpdate": "2022-09-26T11:28:00.189413Z"
      }
    }
  ],
  "mod_type": "UPDATE", // options are INSERT, UPDATE, DELETE
  "value_capture_type": "NEW_ROW_AND_OLD_VALUES",
  "number_of_records_in_transaction": 1,
  "number_of_partitions_in_transaction": 1,
  "transaction_tag": "app=banking,env=prod,action=update",
  "is_system_transaction": false
}
```

### Heartbeat records

When a heartbeat record is returned, it indicates that all changes with `  commit_timestamp  ` less than or equal to the heartbeat record's `  timestamp  ` have been returned, and future data records in this partition must have higher commit timestamps than that returned by the heartbeat record. Heartbeat records are returned when there are no data changes written to a partition. When there are data changes written to the partition, `  data_change_record.commit_timestamp  ` can be used instead of `  heartbeat_record.timestamp  ` to tell that the reader is making forward progress in reading the partition.

You can use heartbeat records returned on partitions to synchronize readers across all partitions. Once all readers have received either a heartbeat greater than or equal to some timestamp `  A  ` or have received data or child partition records greater than or equal to timestamp `  A  ` , the readers know they have received all records committed at or before that timestamp `  A  ` and can start processing the buffered records—for example, sorting the cross-partition records by timestamp and grouping them by `  server_transaction_id  ` .

A heartbeat record contains only one field:

### GoogleSQL

<table>
<thead>
<tr class="header">
<th>Field</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         timestamp        </code></td>
<td><code dir="ltr" translate="no">         TIMESTAMP        </code></td>
<td>Indicates the heartbeat record's timestamp.</td>
</tr>
</tbody>
</table>

### PostgreSQL

<table>
<thead>
<tr class="header">
<th>Field</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         timestamp        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates the heartbeat record's timestamp.</td>
</tr>
</tbody>
</table>

#### Example heartbeat record

An example heartbeat record, communicating that all records with timestamps less or equal than this record's timestamp have been returned:

``` text
heartbeat_record: {
  "timestamp": "2022-09-27T12:35:00.312486Z"
}
```

### Child partition records

Child partition records returns information about child partitions: their partition tokens, the tokens of their parent partitions, and the `  start_timestamp  ` that represents the earliest timestamp that the child partitions contain change records for. Records whose commit timestamps are immediately prior to the `  child_partitions_record.start_timestamp  ` are returned in the current partition. After returning all the child partition records for this partition, this query returns with a success status, indicating all records have been returned for this partition.

The fields of a child partition record includes the following:

### GoogleSQL

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Field</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         start_timestamp        </code></td>
<td><code dir="ltr" translate="no">         TIMESTAMP        </code></td>
<td>Indicates that the data change records returned from child partitions in this child partition record have a commit timestamp greater than or equal to <code dir="ltr" translate="no">         start_timestamp        </code> . When querying a child partition, the query should specify the child partition token and a <code dir="ltr" translate="no">         start_timestamp        </code> greater than or equal to <code dir="ltr" translate="no">         child_partitions_token.start_timestamp        </code> . All child partitions records returned by a partition have the same <code dir="ltr" translate="no">         start_timestamp        </code> and the timestamp always falls between the query's specified <code dir="ltr" translate="no">         start_timestamp        </code> and <code dir="ltr" translate="no">         end_timestamp        </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         record_sequence        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates a monotonically increasing sequence number that can be used to define the ordering of the child partition records when there are multiple child partition records returned with the same <code dir="ltr" translate="no">         start_timestamp        </code> in a particular partition. The partition token, <code dir="ltr" translate="no">         start_timestamp        </code> and <code dir="ltr" translate="no">         record_sequence        </code> uniquely identify a child partition record.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         child_partitions        </code></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>[
  {
    &quot;token&quot; : &quot;STRING&quot;,
    &quot;parent_partition_tokens&quot; : [&quot;STRING&quot;]
  }
]</code></pre></td>
<td>Returns a set of child partitions and their associated information. This includes the partition token string used to identify the child partition in queries, as well as the tokens of its parent partitions.</td>
</tr>
</tbody>
</table>

### PostgreSQL

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Field</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         start_timestamp        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates that the data change records returned from child partitions in this child partition record have a commit timestamp greater than or equal to <code dir="ltr" translate="no">         start_timestamp        </code> . When querying a child partition, the query should specify the child partition token and a <code dir="ltr" translate="no">         start_timestamp        </code> greater than or equal to <code dir="ltr" translate="no">         child_partitions_token.start_timestamp        </code> . All child partitions records returned by a partition have the same <code dir="ltr" translate="no">         start_timestamp        </code> and the timestamp always falls between the query's specified <code dir="ltr" translate="no">         start_timestamp        </code> and <code dir="ltr" translate="no">         end_timestamp        </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         record_sequence        </code></td>
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td>Indicates a monotonically increasing sequence number that can be used to define the ordering of the child partition records when there are multiple child partition records returned with the same <code dir="ltr" translate="no">         start_timestamp        </code> in a particular partition. The partition token, <code dir="ltr" translate="no">         start_timestamp        </code> and <code dir="ltr" translate="no">         record_sequence        </code> uniquely identify a child partition record.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         child_partitions        </code></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>[
  {
    &quot;token&quot;: &quot;STRING&quot;,
    &quot;parent_partition_tokens&quot;: [&quot;STRING&quot;],
  }, [...]
]</code></pre></td>
<td>Returns an array of child partitions and their associated information. This includes the partition token string used to identify the child partition in queries, as well as the tokens of its parent partitions.</td>
</tr>
</tbody>
</table>

#### Example child partition record

The following is an example of a child partition record:

``` text
child_partitions_record: {
  "start_timestamp": "2022-09-27T12:40:00.562986Z",
  "record_sequence": "00000001",
  "child_partitions": [
    {
      "token": "child_token_1",
      // To make sure changes for a key is processed in timestamp
      // order, wait until the records returned from all parents
      // have been processed.
      "parent_partition_tokens": ["parent_token_1", "parent_token_2"]
    }
  ],
}
```

## Change streams query workflow

Run change stream queries using the [`  ExecuteStreamingSql  `](/spanner/docs/reference/rpc/google.spanner.v1#%0Agoogle.spanner.v1.Spanner.ExecuteStreamingSql) API, with a single-use [read-only transaction](/spanner/docs/transactions#read-only_transactions) and a strong [timestamp bound](/spanner/docs/timestamp-bounds) . The change stream read function lets you specify the `  start_timestamp  ` and `  end_timestamp  ` for the time range of interest. All change records within the retention period are accessible using the strong read-only timestamp bound.

All other [`  TransactionOptions  `](/spanner/docs/reference/rest/v1/TransactionOptions) are invalid for change stream queries. In addition, if `  TransactionOptions.read_only.return_read_timestamp  ` is set to `  true  ` , a special value of `  kint64max - 1  ` is returned in the `  Transaction  ` message that describes the transaction, instead of a valid read timestamp. This special value should be discarded and not used for any subsequent queries.

Each change stream query can return any number of rows, each containing either a data change record, heartbeat record, or child partitions record. There is no need to set a deadline for the request.

### Example change stream query workflow

The streaming query workflow begins with issuing the very first change stream query by specifying the `  partition_token  ` to `  NULL  ` . The query needs to specify the read function for the change stream, start and end timestamp of interest, and the heartbeat interval. When the `  end_timestamp  ` is `  NULL  ` , the query keeps returning data changes until the partition ends.

### GoogleSQL

``` text
SELECT ChangeRecord FROM READ_SingersNameStream (
  start_timestamp => "2022-05-01T09:00:00Z",
  end_timestamp => NULL,
  partition_token => NULL,
  heartbeat_milliseconds => 10000
);
```

### PostgreSQL

``` text
SELECT *
FROM "spanner"."read_json_SingersNameStream" (
  '2022-05-01T09:00:00Z',
  NULL,
  NULL,
  10000,
  NULL
) ;
```

Process data records from this query until all child partition records are returned. In the following example, two child partition records and three partition tokens are returned, then the query terminates. Child partition records from a specific query always shares the same `  start_timestamp  ` .

``` text
child_partitions_record: {
  "record_type": "child_partitions",
  "start_timestamp": "2022-05-01T09:00:01Z",
  "record_sequence": "1000012389",
  "child_partitions": [
    {
      "token": "child_token_1",
      // Note parent tokens are null for child partitions returned
        // from the initial change stream queries.
      "parent_partition_tokens": [NULL]
    }
    {
      "token": "child_token_2",
      "parent_partition_tokens": [NULL]
    }
  ],
}
```

``` text
child_partitions_record: {
  "record_type": "child_partitions",
  "start_timestamp": "2022-05-01T09:00:01Z",
  "record_sequence": "1000012390",
  "child_partitions": [
    {
      "token": "child_token_3",
      "parent_partition_tokens": [NULL]
    }
  ],
}
```

To process changes after `  2022-05-01T09:00:01Z  ` , create three new queries and run them in parallel. Used together, the three queries return data changes for the same key range their parent covers. Always set the `  start_timestamp  ` to the `  start_timestamp  ` in the same child partition record and use the same `  end_timestamp  ` and heartbeat interval to process the records consistently across all queries.

### GoogleSQL

``` text
SELECT ChangeRecord FROM READ_SingersNameStream (
  start_timestamp => "2022-05-01T09:00:01Z",
  end_timestamp => NULL,
  partition_token => "child_token_1",
  heartbeat_milliseconds => 10000
);
```

``` text
SELECT ChangeRecord FROM READ_SingersNameStream (
  start_timestamp => "2022-05-01T09:00:01Z",
  end_timestamp => NULL,
  partition_token => "child_token_2",
  heartbeat_milliseconds => 10000
);
```

``` text
SELECT ChangeRecord FROM READ_SingersNameStream (
  start_timestamp => "2022-05-01T09:00:01Z",
  end_timestamp => NULL,
  partition_token => "child_token_3",
  heartbeat_milliseconds => 10000
);
```

### PostgreSQL

``` text
SELECT *
FROM "spanner"."read_json_SingersNameStream" (
  '2022-05-01T09:00:01Z',
  NULL,
  'child_token_1',
  10000,
  NULL
);
```

``` text
SELECT *
FROM "spanner"."read_json_SingersNameStream" (
  '2022-05-01T09:00:01Z',
  NULL,
  'child_token_2',
  10000,
  NULL
);
```

``` text
SELECT *
FROM "spanner"."read_json_SingersNameStream" (
  '2022-05-01T09:00:01Z',
  NULL,
  'child_token_3',
  10000,
  NULL
);
```

The query on `  child_token_2  ` finishes after returning another child partition record. This record indicates that a new partition is covering changes for both `  child_token_2  ` and `  child_token_3  ` starting at `  2022-05-01T09:30:15Z  ` . The exact same record is returned by the query on `  child_token_3  ` , because both are the parent partitions of the new `  child_token_4  ` . To ensure a strict ordered processing of data records for a particular key, the query on `  child_token_4  ` must start after all the parents have finished. In this case, the parents are `  child_token_2  ` and `  child_token_3  ` . Only create one query for each child partition token. The query workflow design should appoint one parent to wait and schedule the query on `  child_token_4  ` .

``` text
child_partitions_record: {
  "record_type": "child_partitions",
  "start_timestamp": "2022-05-01T09:30:15Z",
  "record_sequence": "1000012389",
  "child_partitions": [
    {
      "token": "child_token_4",
      "parent_partition_tokens": ["child_token_2", "child_token_3"],
    }
  ],
}
```

### GoogleSQL

``` text
SELECT ChangeRecord FROM READ_SingersNameStream(
  start_timestamp => "2022-05-01T09:30:15Z",
  end_timestamp => NULL,
  partition_token => "child_token_4",
  heartbeat_milliseconds => 10000
);
```

### PostgreSQL

``` text
SELECT *
FROM "spanner"."read_json_SingersNameStream" (
  '2022-05-01T09:30:15Z',
  NULL,
  'child_token_4',
  10000,
  NULL
);
```

Find examples of handling and parsing change stream records in the Apache Beam SpannerIO Dataflow connector on [GitHub](https://github.com/apache/beam/tree/master/sdks/java/io/google-cloud-platform/src/main/java/org/apache/beam/sdk/io/gcp/spanner/changestreams) .
