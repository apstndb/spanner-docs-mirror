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

MCP Tools

[get\_instance](/spanner/docs/reference/mcp/get_instance)

Get information about a Spanner instance

[list\_instances](/spanner/docs/reference/mcp/list_instances)

List Spanner instances in a given project. \* Response may include next\_page\_token to fetch additional instances using list\_instances tool with page\_token set.

[list\_configs](/spanner/docs/reference/mcp/list_configs)

List instance configs in a given project. \* Response may include next\_page\_token to fetch additional configs using list\_configs tool with page\_token set.

[create\_instance](/spanner/docs/reference/mcp/create_instance)

Create a Spanner instance in a given project.

[delete\_instance](/spanner/docs/reference/mcp/delete_instance)

Delete a Spanner instance.

[create\_database](/spanner/docs/reference/mcp/create_database)

Create a Spanner database in a given instance.

[drop\_database](/spanner/docs/reference/mcp/drop_database)

Drops a Spanner database.

[get\_database\_ddl](/spanner/docs/reference/mcp/get_database_ddl)

Get database schema for a given database.

[list\_databases](/spanner/docs/reference/mcp/list_databases)

List Spanner databases in a given spanner instance. \* Response may include next\_page\_token to fetch additional databases using list\_databases tool with page\_token set.

[create\_session](/spanner/docs/reference/mcp/create_session)

Create a session in a given database for query executions using execute\_sql tool. \* Session can be reused to execute multiple concurrent operations.

[execute\_sql](/spanner/docs/reference/mcp/execute_sql)

Execute SQL statement using a given session. \* execute\_sql tool can be used to execute DQL as well as DML statements. \* Use commit tool to commit result of a DML statement. \* DDL statements are only supported using update\_database\_schema tool.

[commit](/spanner/docs/reference/mcp/commit)

Commit a transaction in a given session. \* If commit is finalizing the result of a DML statement then commit request should include latest precommit\_token returned by execute\_sql tool. \* If response to commit includes another precommit\_token then issue another commit call to finalize the transaction with the latest precommit\_token.

[update\_database\_schema](/spanner/docs/reference/mcp/update_database_schema)

Update schema for a given database.

[get\_operation](/spanner/docs/reference/mcp/get_operation)

Get status of a long-running operation. \* Long running operation may take several minutes to complete. get\_operation tool can be used to poll the status of a long running operation.

### Get MCP tool specifications

To get the MCP tool specifications for all tools in an MCP server, use the `  tools/list  ` method. The following example demonstrates how to use `  curl  ` to list all tools and their specifications currently available within the MCP server.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Curl Request</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="Bash" translate="no"><code>                      
curl --location &#39;https://spanner.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
    &quot;method&quot;: &quot;tools/list&quot;,
    &quot;jsonrpc&quot;: &quot;2.0&quot;,
    &quot;id&quot;: 1
}&#39;
                    </code></pre></td>
</tr>
</tbody>
</table>
