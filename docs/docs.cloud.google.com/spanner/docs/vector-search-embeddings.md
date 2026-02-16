[Vertex AI Vector Search](/vertex-ai/docs/vector-search/overview) allows users to search for semantically similar items using vector embeddings. Using the [Spanner To Vertex AI Vector Search Workflow](https://github.com/cloudspannerecosystem/spanner-ai/tree/main/vertex-vector-search/workflows#readme) , you can integrate your Spanner database with Vector Search to perform a vector similarity search on your Spanner data.

The following diagram shows the end-to-end application workflow of how you can enable and use Vector Search on your Spanner data:

The general workflow is as follows:

1.  **Generate and store vector embeddings.**
    
    You can generate vector embeddings of your data, then store and manage them in Spanner with your operational data. You can generate embeddings with Spanner's `  ML.PREDICT  ` SQL function to [access the Vertex AI text embedding model](/spanner/docs/ml-tutorial-embeddings#generate-store-embeddings) or [use other embedding models deployed to Vertex AI](/spanner/docs/ml-tutorial) .

2.  **Sync embeddings to Vector Search.**
    
    Use the [Spanner To Vertex AI Vector Search Workflow](https://github.com/cloudspannerecosystem/spanner-ai/tree/main/vertex-vector-search/workflows#readme) , which is deployed using [Workflows](/workflows/docs/overview) to export and upload embeddings into a Vector Search index. You can use Cloud Scheduler to periodically schedule this workflow to keep your Vector Search index up to date with the latest changes to your embeddings in Spanner.

3.  **Perform vector similarity search using your Vector Search index.**
    
    Query the Vector Search index to search and find results for semantically similar items. You can query using a [public endpoint](/vertex-ai/docs/vector-search/query-index-public-endpoint) or through [VPC peering](/vertex-ai/docs/vector-search/query-index-vpc) .

## Example use case

An illustrative use case for Vector Search is an online retailer who has an inventory of hundreds of thousands of items. In this scenario, you are a developer for an online retailer, and you would like to use vector similarity search on your product catalog in Spanner to help your customers find relevant products based on their search queries.

Follow step 1 and step 2 presented in the general workflow to generate vector embeddings for your product catalog, and sync these embeddings to Vector Search.

Now imagine a customer browsing your application performs a search such as "best, quick-drying sports shorts that I can wear in the water". When your application receives this query, you need to generate a request embedding for this search request using the Spanner [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) SQL function. Make sure to use the same embedding model used to generate the embeddings for your product catalog.

Next, query the Vector Search index for product IDs whose corresponding embeddings are similar to the request embedding generated from your customer's search request. The search index might recommend product IDs for semantically similar items such as wakeboarding shorts, surfing apparel, and swimming trunks.

After Vector Search returns these similar product IDs, you can query Spanner for the products' descriptions, inventory count, price, and other metadata that are relevant, and display them to your customer.

You can also use [generative AI](/vertex-ai/generative-ai/docs/overview) to process the returned results from Spanner before displaying them to your customer. For example, you might use Google's large generative AI models to generate a concise summary of the recommended products. For more information, see this tutorial on how to [use Generative AI to get personalized recommendations in an ecommerce application](/spanner/docs/ml-tutorial-generative-ai) .

## What's next

  - Learn how to [generate embeddings](/spanner/docs/ml-tutorial-embeddings) using Spanner.
  - Learn more about [AI's multitool: Vector embeddings](https://cloud.google.com/blog/topics/developers-practitioners/meet-ais-multitool-vector-embeddings)
  - Learn more about machine learning and embeddings in our [crash course on embeddings](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture) .
  - Learn more about the Spanner To Vertex AI Vector Search Workflow, see the [GitHub repository](https://github.com/cloudspannerecosystem/spanner-ai/tree/main/vertex-vector-search/workflows) .
  - Learn more about the [open source spanner-analytics package](https://github.com/cloudspannerecosystem/spanner-analytics/) that facilitates common data-analytic operations in Python and includes integrations with Jupyter Notebooks.
