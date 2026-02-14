**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Spanner and its multi-model capabilities integrate with Google Cloud's AI services and LangChain, an open-source framework, to help build generative AI applications. You can enhance applications with features like similarity search, retrieval-augmented generation (RAG), and knowledge graphs. Spanner builds this functionality on its foundation of scalability, availability, and consistency.

## Perform similarity searches with Vector Search

Use Spanner with [Vector Search](/vertex-ai/docs/vector-search/overview) to implement similarity search on unstructured text data. Spanner integrates with services like Vertex AI to invoke the generation of [vector embeddings](/spanner/docs/ml-tutorial-embeddings) from unstructured text data. These embeddings are numerical representations that reflect the semantic meaning of the text. To find conceptually similar items, you use vector distance functions to find embedding vectors that are most similar to the embedding of the search request. This process lets you build features such as product or content recommendations.

To get started, [generate and backfill Vertex AI vector embeddings](/spanner/docs/backfill-embeddings) in bulk for existing textual data. You can do this by using SQL and a Vertex AI embedding model, such as the `  text-embedding  ` model, described in the [text embeddings API documentation](/vertex-ai/generative-ai/docs/model-reference/text-embeddings-api) . Spanner supports using [approximate nearest neighbors (ANN)](/spanner/docs/find-approximate-nearest-neighbors) and [K-nearest neighbors (KNN)](/spanner/docs/find-k-nearest-neighbors) with query vector embeddings. ANN uses a vector index for a fast, scalable search that returns approximate results. KNN performs an exhaustive search that returns more accurate results, but can be slow for large datasets. You can use [multiple vector distance functions](/spanner/docs/choose-vector-distance-function) to measure similarity, including:

  - **Cosine distance** : Measures the cosine of the angle between two vectors, which is useful for finding items with similar orientation, regardless of magnitude.

  - **Euclidean distance** : Measures the straight-line distance between two vectors.

  - **Dot product** : Calculates the product of vector magnitudes and the cosine of the angle between them. This can be the most computationally efficient option for normalized vectors.

For more information, see the following topics:

  - [Vector store for Spanner](/spanner/docs/langchain#vector-store)
  - [Vector Search overview](/vertex-ai/docs/vector-search/overview)

## Generate ML predictions with SQL

You can use SQL queries in Spanner to invoke large language models (LLMs) that are deployed in Vertex AI. Direct access to LLMs lets you run predictions for tasks such as sentiment analysis, text classification, and translation on data stored in Spanner.

By using the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) (GoogleSQL) or the [`  spanner.ML_PREDICT_ROW  `](/spanner/docs/reference/postgresql/functions-and-operators#ml) (PostgreSQL) function, you can generate machine learning (ML) predictions without having to move your data or write custom application code to interact with the LLM. This simplifies your application architecture and brings ML capabilities closer to your data. For more information, see [Generate ML predictions using SQL](/spanner/docs/ml-tutorial) .

## Use Model Context Protocol (MCP) to connect to LLM agents

You can connect your Spanner instance to IDEs that support [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) . MCP is an open protocol that you can use for connecting LLMs to your data in Spanner. After connection, your LLM agents can query and interact with your Spanner instance. For more information, see [Connect your IDE to Spanner](/spanner/docs/pre-built-tools-with-mcp-toolbox) .

## Uncover insights with Spanner graphs

For more advanced RAG use cases, Spanner Graph integrates graph database capabilities with Spanner's core strengths. Spanner Graph lets you model, store, and query highly connected data.

Integrate Spanner Graph with LangChain to build GraphRAG applications. This integration can enhance traditional RAG. GraphRAG lets you create applications that capture complex relationships between entities, such as a knowledge graph. The integration uses graph queries in addition to Vector Search to capture complex, implicit relationships in your data. Using graph queries and Vector Search together can provide more accurate and relevant answers from your LLM than using Vector Search alone.

For more information, see [GraphRAG infrastructure for generative AI using Vertex AI and Spanner Graph](https://cloud.google.com/architecture/gen-ai-graphrag-spanner) .

## Build LLM-powered applications with LangChain

Spanner provides several classes to programmatically work with LangChain. LangChain is an LLM orchestration framework that provides the structure, tools, and components to streamline complex LLM workflows. Use LangChain to build generative AI applications and RAG workflows. The available LangChain classes for Spanner include:

  - **[`  SpannerVectorStore  `](https://cloud.google.com/python/docs/reference/langchain-google-spanner/latest/langchain_google_spanner.vector_store.SpannerVectorStore)** : Store and search vector embeddings to enable similarity search within your application with the class.

  - **[`  SpannerLoader  `](https://cloud.google.com/python/docs/reference/langchain-google-spanner/latest/langchain_google_spanner.loader.SpannerLoader)** : Load data from Spanner to be used in embeddings or to provide specific context to LLM chains with the class.

  - **[`  SpannerChatMessageHistory  `](https://cloud.google.com/python/docs/reference/langchain-google-spanner/latest/langchain_google_spanner.chat_message_history.SpannerChatMessageHistory)** : Enable conversational AI applications by storing the history of conversations in a Spanner database.

For more information, see [Build LLM-powered applications using LangChain](/spanner/docs/langchain) and [Spanner client library for LangChain](https://cloud.google.com/python/docs/reference/langchain-google-spanner/latest) .

## Explore use cases

Use Spanner's AI capabilities to build intelligent applications for use cases such as the following:

  - **Ecommerce recommendation engines** : Generate vector embeddings for product descriptions to power a recommendation engine. This engine can suggest similar items to customers, which enhances their shopping experience and increases sales. For more information, see [Use Generative AI to get personalized recommendations in an ecommerce application](/spanner/docs/ml-tutorial-generative-ai) .

  - **Manage chat message history** : Use Spanner and LangChain to store and retrieve conversation history. Spanner stores this data in a database and provides the `  SpannerChatMessageHistory  ` class. This class extends a LangChain base class to save and retrieve messages from a database. For more information, see [Chat message history with Spanner](/spanner/docs/langchain#chat-message-history) .

  - **Financial fraud detection** : Use Spanner Graph to analyze complex relationships between users, accounts, and transactions to identify suspicious patterns and anomalies that are difficult to detect with traditional relational databases.

  - **Customer 360** : With Spanner Graph, gain a holistic view of customers by tracking relationships, preferences, and purchase histories. This provides personalized recommendations, targeted marketing campaigns, and improved customer service experiences.

  - **Social networks** : Model user activities and interactions with Spanner Graph to provide friend recommendations and discover content in social networks.

## What's next

To learn more about implementing AI capabilities in Spanner, see the following topics:

  - [Generate and backfill vector embeddings](/spanner/docs/backfill-embeddings)
  - [Find k-nearest neighbors](/spanner/docs/find-k-nearest-neighbors)
  - [Improve vector search performance with approximate nearest neighbors](/spanner/docs/find-approximate-nearest-neighbors)
  - [LangChain integration](/spanner/docs/langchain)
  - [Spanner Graph overview](/spanner/docs/graph/overview)
