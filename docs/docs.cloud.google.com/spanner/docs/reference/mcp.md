The [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) (MCP) standardizes how AI applications and agents connect to outside data sources. MCP servers can be local or remote. Local MCP servers typically run on your local machine and use the standard input and output streams (stdio) for communication between services on the same device. Remote MCP servers run on the service's infrastructure and offer an HTTP endpoint to AI applications for communication between the AI MCP client and the MCP server.

For more information, see [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture) .

Spanner supports two remote servers for usage with MCP:

  - **Spanner server** : Provides tools for managing Spanner instances and databases, and executing SQL queries.
    
      - Learn how to [Use the Spanner remote MCP server](https://docs.cloud.google.com/spanner/docs/use-spanner-mcp) .
      - View the detailed [Spanner MCP tool reference](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp) .

  - **Database insights server** : Provides tools for querying and analyzing database performance and system metrics.
    
      - Learn how to [Use the Database Insights MCP server](https://docs.cloud.google.com/spanner/docs/use-database-insights-mcp) .
      - View the detailed [Database insights MCP tool reference](https://docs.cloud.google.com/spanner/docs/reference/mcp/databaseinsights/mcp) .
