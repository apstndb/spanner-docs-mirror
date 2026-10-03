---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp
title: 'MCP Reference: spanner.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

A [Model Context Protocol (MCP) server](https://modelcontextprotocol.io/docs/learn/server-concepts) acts as a proxy between an external service that provides context, data, or capabilities to a Large Language Model (LLM) or AI application. MCP servers connect AI applications to external systems such as databases and web services, translating their responses into a format that the AI application can understand.

### Server Setup

You must [enable MCP servers](https://docs.cloud.google.com/mcp/enable-disable-mcp-servers) and [set up authentication](https://docs.cloud.google.com/mcp/authenticate-mcp) before use. For more information about using Google and Google Cloud remote MCP servers, see [Google Cloud MCP servers overview](https://docs.cloud.google.com/mcp/overview) .

Spanner MCP Server provides tools to interact with Spanner

### Server Endpoints

An MCP service endpoint is the network address and communication interface (usually a URL) of the MCP server that an AI application (the Host for the MCP client) uses to establish a secure, standardized connection. It is the point of contact for the LLM to request context, call a tool, or access a resource. Google MCP endpoints can be global or regional.

The spanner.googleapis.com MCP server has the following MCP endpoint:

- https://spanner.googleapis.com/mcp

## MCP Tools

An [MCP tool](https://modelcontextprotocol.io/legacy/concepts/tools) is a function or executable capability that an MCP server exposes to a LLM or AI application to perform an action in the real world.

The spanner.googleapis.com MCP server has the following tools:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>MCP Tools</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance">get_instance</a></td>
<td>Get information about a Spanner instance</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_instances">list_instances</a></td>
<td><p>List Spanner instances in a given project.</p>
<ul>
<li>Response may include next_page_token to fetch additional instances using list_instances tool with page_token set.</li>
</ul></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs">list_configs</a></td>
<td><p>List instance configs in a given project.</p>
<ul>
<li>Response may include next_page_token to fetch additional configs using list_configs tool with page_token set.</li>
</ul></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_config">get_config</a></td>
<td>Get information about a specific Spanner instance configuration.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/create_instance">create_instance</a></td>
<td>Create a Spanner instance in a given project.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/update_instance">update_instance</a></td>
<td><p>Update a Spanner instance.</p>
<ul>
<li>While updating an instance always include instance name and config.</li>
</ul></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/create_database">create_database</a></td>
<td>Create a Spanner database in a given instance.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_database_ddl">get_database_ddl</a></td>
<td>Get database schema for a given database.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases">list_databases</a></td>
<td>List Spanner databases in a given spanner instance. * Response may include next_page_token to fetch additional databases using list_databases tool with page_token set.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/create_session">create_session</a></td>
<td><p>Create a session in a given database for query executions using execute_sql tool.</p>
<ul>
<li>Session can be reused to execute multiple concurrent operations.</li>
</ul></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/execute_sql">execute_sql</a></td>
<td><p>Execute SQL statement using a given session.</p>
<ul>
<li>execute_sql tool can be used to execute DQL as well as DML statements.</li>
<li>Prefer using parameterized queries over literal values.</li>
<li>Use commit tool to commit result of a DML statement.</li>
<li>DDL statements are only supported using update_database_schema tool.</li>
</ul></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/execute_sql_readonly">execute_sql_readonly</a></td>
<td><p>Execute SQL query statement using a given session in a read-only transaction.</p>
<ul>
<li>execute_sql_readonly tool can be used to execute DQL statements.</li>
<li>Prefer using parameterized queries over literal values.</li>
<li>DML statements and multi read transactions are only supported via execute_sql tool.</li>
<li>The transaction bit should not be set in the request and will default to single-use read-only transaction.</li>
</ul></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/commit">commit</a></td>
<td><p>Commit a transaction in a given session.</p>
<ul>
<li>If commit is finalizing the result of a DML statement then commit request should include latest precommit_token returned by execute_sql tool.</li>
<li>If response to commit includes another precommit_token then issue another commit call to finalize the transaction with the latest precommit_token.</li>
</ul></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/update_database_schema">update_database_schema</a></td>
<td>Update schema for a given database.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_operation">get_operation</a></td>
<td><p>Get status of a long-running operation.</p>
<ul>
<li>Long running operation may take several minutes to complete. get_operation tool can be used to poll the status of a long running operation.</li>
</ul></td>
</tr>
</tbody>
</table>

### Get MCP tool specifications

To get the MCP tool specifications for all tools in an MCP server, use the `tools/list` method. The following example demonstrates how to use `curl` to list all tools and their specifications currently available within the MCP server.

**Curl Request**

```
curl --location 'https://spanner.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
    "method": "tools/list",
    "jsonrpc": "2.0",
    "id": 1
}'
```
