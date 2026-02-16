**Preview — LangChain**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

This page introduces how to build LLM-powered applications using [LangChain](https://www.langchain.com/) . The overviews on this page link to procedure guides in GitHub.

## What is LangChain?

LangChain is an LLM orchestration framework that helps developers build generative AI applications or retrieval-augmented generation (RAG) workflows. It provides the structure, tools, and components to streamline complex LLM workflows.

For more information about LangChain, see the [Google LangChain](https://python.langchain.com/docs/integrations/platforms/google) page. For more information about the LangChain framework, see the [LangChain](https://python.langchain.com/docs/get_started/introduction) product documentation.

## LangChain components for Spanner

Spanner offers the following LangChain interfaces:

  - [Vector store](#vector-store)
  - [Document loader](#document-loader)
  - [Chat message history](#chat-message-history)
  - [Graph store](#graph-store)
  - [Graph QA](#graph-qa)

## Vector store for Spanner

Vector store retrieves and stores documents and metadata from a vector database. Vector store gives an application the ability to perform semantic searches that interpret the meaning of a user query. This type of search is a called a vector search, and it can find topics that match the query conceptually. At query time, vector store retrieves the embedding vectors that are most similar to the embedding of the search request. In LangChain, a vector store takes care of storing embedded data and performing the vector search for you.

To work with vector store in Spanner, use the `  SpannerVectorStore  ` class.

For more information, see the [LangChain Vector Stores](https://python.langchain.com/docs/concepts/vectorstores/) product documentation.

### Vector store procedure guide

The [Spanner guide for vector store](https://github.com/googleapis/langchain-google-spanner-python/blob/main/docs/vector_store.ipynb) shows you how to do the following:

  - Install the integration package and LangChain
  - Initialize a table for the vector store
  - Set up an embedding service using `  VertexAIEmbeddings  `
  - Initialize `  SpannerVectorStore  `
  - Add and delete documents
  - Search for similar documents
  - Create a custom vector store to connect to a pre-existing Spanner database that has a table with vector embeddings

## Document loader for Spanner

The document loader saves, loads, and deletes a LangChain `  Document  ` objects. For example, you can load data for processing into embeddings and either store it in vector store or use it as a tool to provide specific context to [chains](https://python.langchain.com/docs/modules/chains/) .

To load documents from Spanner, use the `  SpannerLoader  ` class. Use the `  SpannerDocumentSaver  ` class to save and delete documents.

For more information, see the [LangChain Document loaders](https://python.langchain.com/docs/modules/data_connection/document_loaders/) topic.

### Document loader procedure guide

The [Spanner guide for document loader](https://github.com/googleapis/langchain-google-spanner-python/blob/main/docs/document_loader.ipynb) shows you how to do the following:

  - Install the integration package and LangChain
  - Load documents from a table
  - Add a filter to the loader
  - Customize the connection and authentication
  - Customize document construction by specifying customer content and metadata
  - How to use and customize a `  SpannerDocumentSaver  ` to store and delete documents

## Chat message history for Spanner

Question and answer applications require a history of the things said in the conversation to give the application context to answer further questions from the user. The LangChain `  ChatMessageHistory  ` class lets the application save messages to a database and retrieve them when needed to formulate further answers. A message can be a question, an answer, a statement, a greeting or any other piece of text that the user or application gives during the conversation. `  ChatMessageHistory  ` stores each message and chains messages together for each conversation.

Spanner extends this class with `  SpannerChatMessageHistory  ` .

### Chat message history procedure guide

The [Spanner guide for chat message history](https://github.com/googleapis/langchain-google-spanner-python/blob/main/docs/chat_message_history.ipynb) shows you how to do the following:

  - Install LangChain and authenticate to Google Cloud
  - Initialize a table
  - Initialize the `  SpannerChatMessageHistory  ` class to add and delete messages
  - Use a client to customize the connection and authentication
  - Delete the `  SpannerChatMessageHistory  ` session

## Graph store for Spanner

Graph store retrieves and stores nodes and edges from a graph database. Use graph store to let an application do the following:

  - Add nodes and edges into a graph
  - Perform traversals and analysis on a graph
  - Inspect the schema of a graph

You can also use graph store with graph QA chain to create an application that can chat with a graph.

To use graph store with Spanner Graph, use the [`  SpannerGraphStore  `](/python/docs/reference/langchain-google-spanner/latest#spanner-graph-store-usage) class. to store nodes and edges that are extracted from documents. `  SpannerGraphStore  ` supports the Graph Query Language (GQL).

### Graph store procedure guide

The [Spanner guide for graph store](https://github.com/googleapis/langchain-google-spanner-python/blob/main/docs/graph_store.ipynb) shows you how to do the following:

  - Install the integration package and LangChain
  - Prepare graphs from various data sources
  - Initialize `  SpannerGraphStore  ` with an existing Spanner Graph database
  - Add nodes and edges into Spanner Graph
  - Perform traversals using a [GQL](/spanner/docs/graph/queries-overview) query
  - Visualize the graph query results
  - Clean up the graph

## Graph QA chain for Spanner

Graph QA chain for Spanner uses a Spanner graph to answer questions. The graph QA workflow to answer a question is as follows:

1.  Graph QA uses a LangChain LLM to translate a natural language question to a GQL query.
2.  Spanner Graph uses the graph store interface to run the GQL query.
3.  The GQL query results are sent back to the LLM.
4.  The LLM generates and returns the answer to the question.

### Graph QA procedure guide

The [Spanner guide for graph QA](https://github.com/googleapis/langchain-google-spanner-python/blob/main/docs/graph_qa_chain.ipynb) demonstrates how to use Spanner and graph QA to answer a question by showing you how to do the following:

  - Create a graph from unstructured text blobs using `  LLMGraphTransformer  ` .
  - Store the graph in Spanner Graph using the `  SpannerGraphStore  ` class.
  - Initialize a `  SpannerGraphQAChain  ` instance.
  - Generate an answer to a natural language question using the graph store in Spanner Graph.
