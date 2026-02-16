**Preview — Spanner integration with LlamaIndex**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

You can build large language model (LLM) applications that use graph retrieval-augmented generation (GraphRAG) with [LlamaIndex](https://www.llamaindex.ai/) and Spanner Graph.

Spanner Graph integrates with LlamaIndex through its property graph store capabilities to let you use the following to create data retrieval workflows:

  - **[Property graph store](#property-graph-store)** : Lets you represent data as a graph by storing nodes and edges in a graph database. You can use the graph database to query for complex relationships in your data.

  - **[Graph retrievers](#graph-retrievers)** : Lets you use an LLM to translate a user's natural language question into a query for the graph store. This enables applications to answer questions using the structured relationships in the graph data.

## What is LlamaIndex?

LlamaIndex is a data framework for building LLM applications that helps you streamline the development of retrieval-augmented generation (RAG) and other context-aware systems. By providing tools to connect LLMs with your data, LlamaIndex helps with data ingestion, indexing, and querying. You can use LlamaIndex with LLMs to build applications that deliver accurate and relevant responses.

For more information about the LlamaIndex framework, see the [LlamaIndex product documentation](https://docs.llamaindex.ai/) .

## Property graph store for Spanner

A property graph store can be used in an application to do the following:

  - Extract entities and relationships from documents and store them as a graph.

  - Perform complex traversals and analysis on a graph structure.

  - Query a graph using the Graph Query Language (GQL) to provide specific context to an LLM.

To work with a property graph store in Spanner Graph, use the `  SpannerPropertyGraphStore  ` class.

### Property graph store tutorial

To learn how to use the property graph store with Spanner, see the [property graph store tutorial for Spanner](https://github.com/googleapis/llama-index-spanner-python/blob/main/docs/property_graph_store.ipynb) . This tutorial helps you learn how to do the following:

  - Install the `  llama-index-spanner  ` package and LlamaIndex

  - Initialize the `  SpannerPropertyGraphStore  ` class and use it to connect to your Spanner database.

  - Add nodes and edges to your Spanner Graph that contain data extracted from documents using a LlamaIndex knowledge graph extractor.

  - Retrieve structured information by querying the graph using GQL.

  - Visualize the results of your graph queries.

## Graph retrievers for Spanner

Graph retrievers in LlamaIndex are components that use an LLM to translate a user's natural language question into a query for the graph store. Applications use the generated query to answer questions by using the structured relationships in the graph data. Graph retrievers use the following workflow to generate an answer from a natural language query:

1.  Prompt an LLM to translate the natural language question into a GQL query.

2.  Run the GQL query against the graph store using Spanner Graph and the `  SpannerPropertyGraphStore  ` class.

3.  Send the structured data that's returned by the query to the LLM using Spanner Graph.

4.  Generate a human-readable answer using the LLM.

### Use LlamaIndex retriever classes

The following LlamaIndex graph retriever classes can be used with Spanner Graph to generate human-readable answers to LLM prompts:

#### `     SpannerGraphTextToGQLRetriever    ` class

The `  SpannerGraphTextToGQLRetriever  ` class translates natural language into GQL queries for data extraction from the graph.

#### `     SpannerGraphCustomRetriever    ` class

The `  SpannerGraphCustomRetriever  ` class implements a hybrid retrieval approach. `  SpannerGraphCustomRetriever  ` handles specific and conceptual questions using the following steps:

1.  Perform the following searches simultaneously:
    
      - A graph search that translates the natural language question into a GQL query that uses the graph to find answers.
    
      - A Vector Search or semantic search to find conceptually related information.

2.  Combine the results from the graph search and the vector search.

3.  Evaluate and re-rank the combined results using the LLM. The LLM selects the most relevant and context-aware information to answer the original question.

### Graph retrievers tutorial

To learn how to use graph retrievers with Spanner to answer questions, see the [graph retrievers tutorial for Spanner](https://github.com/googleapis/llama-index-spanner-python/blob/main/docs/graph_retriever.ipynb) . This tutorial shows you how to:

  - Create a graph from unstructured text blobs.

  - Store the graph in Spanner using the `  SpannerPropertyGraphStore  ` class

  - Initialize a `  SpannerGraphTextToGQLRetriever  ` class and a `  SpannerGraphCustomRetriever  ` instance using your graph store and an LLM.

  - Generate an answer to a natural language question using the graph data that's stored in Spanner.

## What's next

  - To learn how to use Spanner with other Google Cloud products to build generative AI applications, see [Spanner AI overview](/spanner/docs/spanner-ai-overview) .

  - To learn about vector search in Spanner, see [Use vector search with Spanner Graph](/spanner/docs/graph/perform-vector-similarity-search) .

  - To learn how to use Spanner to store vector embeddings, see [Get Spanner text embeddings](/spanner/docs/ml-tutorial-embeddings) .

  - To learn more about machine learning with Spanner, see the [Vertex AI integration overview](/spanner/docs/ml) .
