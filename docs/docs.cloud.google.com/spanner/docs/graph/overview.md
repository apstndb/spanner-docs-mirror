**PostgreSQL interface note:** The examples in topics for Spanner Graph are intended for GoogleSQL-dialect databases. Spanner Graph doesn't support the PostgreSQL interface.

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Spanner Graph combines graph database capabilities with [Spanner](/spanner/docs) scalability, availability, and consistency. Spanner Graph supports an ISO Graph Query Language (GQL)-compatible graph query interface and enables interoperability between relational and graph models.

Spanner Graph lets you map tables to property graphs using declarative schema without data migration, bringing graphs to tabular datasets. You can also late-bind data model choices per query, which helps you choose the right tool for your workflows.

To get started with Spanner Graph, see [Set up and query Spanner Graph](/spanner/docs/graph/set-up) and the [Spanner Graph codelab](https://codelabs.developers.google.com/codelabs/spanner-graph-getting-started) .

## Benefits of Spanner Graph databases

Graphs provide a natural mechanism for representing relationships in data. Example use cases for graph databases include fraud detection, recommendations, cybersecurity, community detection, knowledge graphs, customer 360, data cataloging, and lineage tracking.

Traditionally, applications represent this type of graph data as tables in a relational database, using multiple joins to traverse the graph. Expressing graph traversal logic in SQL creates complex queries that are difficult to write, maintain, and debug.

The graph interface in Spanner Graph lets you navigate relationships and identify patterns in the graph in intuitive ways. In addition, Spanner Graph provides graph-optimized storage and query enhancements suited for online analytical and transactional graph workloads, all built into Spanner's core capabilities.

This approach makes Spanner Graph the ideal solution for even mission-critical graph applications. In particular, Spanner's transparent sharding scales elastically to very large datasets. It uses massively parallel processing without user intervention.

## Use cases for Spanner Graph

You can use Spanner Graph to build many types of online Graph applications, including the following:

  - **Detect financial fraud** : Analyze complex relationships among users, accounts, and transactions to identify suspicious patterns and anomalies, such as money laundering and unusual connections between entities, which can be difficult to detect using relational databases.

  - **Track customer relationships** : Track customer relationships, preferences, and purchase histories. Gain a holistic understanding of each customer, enable personalized recommendations, targeted marketing campaigns, and improved customer service experiences.

  - **Capture social networks** : Capture user activities and interactions, and use graph pattern matching for friend recommendations and content discovery.

  - **Manage manufacturing and supply chains** : Model parts, suppliers, orders, availability, and defects in the graph to analyze impact, roll up costs, and check compliance.

  - **Analyze healthcare data** : Capture patient relationships, conditions, diagnoses, and treatments to facilitate patient similarity analysis and treatment planning.

  - **Manage supply chains** : Given a shipment routing plan, evaluate route segments to identify violations of segment rules.

## Key capabilities

Spanner Graph is a multi-model database that integrates graph, relational, search, and AI capabilities. It offers high performance and scalability, delivering the following:

  - **Native graph experience** : The ISO GQL interface offers a familiar, purpose-built graph experience that's based on open standards.

  - **Build GraphRAG workflow applications** : Spanner Graph integrates with LangChain to help you build GraphRAG applications. While conventional retrieval-augmented generation (RAG) uses vector search to provide context to a large language model (LLM), it can't use the implicit relationships in your data. GraphRAG overcomes this limitation by building a graph from your data to capture these complex relationships. It then combines graph search (for relationship-based context) with vector search (for semantic similarity), generating more accurate, relevant, and complete answers than using either method alone. For more information, see [Build LLM-powered applications using LangChain](/spanner/docs/langchain) . To learn how you can use Spanner Graph with Vertex AI to build infrastructure for a GraphRAG-capable generative AI application, see [GraphRAG infrastructure for generative AI using Vertex AI and Spanner Graph](/architecture/gen-ai-graphrag-spanner) .

  - **Unified relational and graph** : Full interoperability between GQL and SQL breaks down data silos. This lets you choose the optimal tool for each use case, without any operational overheads to extract, transform, and load (ETL).

  - **Built-in search capabilities** : Rich vector and full-text search capabilities are integrated with graph, letting you use semantic meaning and keywords in graph analysis.

  - **AI-powered insights** : Deep integration with Vertex AI unlocks a suite of AI models directly in Spanner Graph, helping you accelerate your AI workflows.

  - **Scalability, availability, and consistency** : Spanner's established scalability, availability, and consistency provide a solid foundation.

## What's next

  - Get started with the [Spanner Graph codelab](https://codelabs.developers.google.com/codelabs/spanner-graph-getting-started) .
  - Set up and query [Spanner Graph](/spanner/docs/graph/set-up) .
  - Learn about the [Spanner Graph schema](/spanner/docs/graph/schema-overview) .
  - Learn how to create, update, or drop a [Spanner Graph schema](/spanner/docs/graph/create-update-drop-schema) .
