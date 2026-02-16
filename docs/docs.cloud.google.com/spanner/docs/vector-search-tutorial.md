**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This tutorial demonstrates how to build a generative AI application using Spanner and [Vertex AI](/vertex-ai/docs/start/introduction-unified-platform) .

This application lets you perform semantic similarity search, enabling you to find products that match a natural language query. It achieves this by using embeddings, which are numerical representations of text that capture the meaning and context of words. You'll use a Vertex AI model to generate these embeddings, and then store and search the embeddings in Spanner. This approach is particularly useful for use cases like product search, where users might describe what they want in natural language instead of specific keywords.

The following topics help you learn how to:

1.  [Create an Google Cloud project](#before-you-begin)
2.  [Create a Spanner instance](#create-instance)
3.  [Create a database](#create-database)
4.  [Create an embedding model](#create-embedding-model)
5.  [Load data into Spanner](#load-data)
6.  [Generate embeddings for data](#generate-embeddings)
7.  [Perform KNN vector similarity search](#perform-vector-search)
8.  [Scale vector search with a vector index](#scale-vector-search)
9.  [Clean up resources](#clean-up)

To learn about Spanner pricing details, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

To try out a codelab, see [Getting started with Spanner vector search](https://codelabs.developers.google.com/codelabs/spanner-getting-started-vector-search) .

**Note:** To explore Spanner using a 90-day free trial instance, see [Create a Spanner free trial instance](/spanner/docs/free-trial-quickstart) .

## Before you begin

You must create a Google Cloud project that is connected to a billing account.

1.  Sign in to your Google Cloud account. If you're new to Google Cloud, [create an account](https://console.cloud.google.com/freetrial) to evaluate how our products perform in real-world scenarios. New customers also get $300 in free credits to run, test, and deploy workloads.

2.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

3.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

4.  Enable the required API.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

5.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

6.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

7.  Enable the required API.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

8.  The Spanner API should be auto-enabled. If not, enable it manually:

9.  The Vertex AI API should be auto-enabled. If not, enable it manually:

<!-- end list -->

4.  To get the permissions that you need to create instances and databases, ask your administrator to grant you the [Cloud Spanner Admin](/iam/docs/roles-permissions/spanner#spanner.admin) ( `  roles/spanner.admin  ` ) IAM role on your project.

<!-- end list -->

5.  To get the permissions that you need to query Spanner graphs if you're not granted the Cloud Spanner Admin role, ask your administrator to grant you the [Cloud Spanner Database Reader](/iam/docs/roles-permissions/spanner#spanner.databaseReader) ( `  roles/spanner.databaseReader  ` ) IAM role on your project.

## Create an instance

When you first use Spanner, you must create an [instance](/spanner/docs/instances) , which is an allocation of resources that are used by Spanner databases. This section shows you how to create an instance using the Google Cloud console.

1.  In the Google Cloud console, go to the **Spanner** page.

2.  Select or create a Google Cloud project if you haven't done so already.

3.  Do one of the following:
    
    1.  If you haven't created a Spanner instance before, on the **Welcome to Spanner** page, click **Create a provisioned instance** .
    2.  If you've created a Spanner instance, on the **Instances** page, click **Create instance** .

4.  On the **Select an edition** page, select **Enterprise Plus** or **Enterprise** .
    
    Spanner vector search is available only in the Enterprise edition or Enterprise Plus edition. To compare the different editions, click **Compare editions** . For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

5.  Click **Continue** .

6.  In **Instance name** , enter an instance name, for example, `  test-instance  ` .

7.  In **Instance ID** keep or change the instance ID. Your instance ID defaults to your instance name, but you can change it. Your instance name and instance ID can be the same or they can be different.

8.  Click **Continue** .

9.  In **Choose a configuration** , do the following:
    
    1.  Keep **Regional** selected.
    2.  In **Select a configuration** , select a region. The region you select is where your instances are stored and replicated.
    3.  Click **Continue** .

10. In **Configure compute capacity** , do the following:
    
    1.  In **Select unit** , select **Processing units (PUs)** .
    2.  In **Choose a scaling mode** , keep **Manual allocation** selected and in **Quantity** keep 1000 processing units.

11. Click **Create** . The Google Cloud console displays the **Overview** page for the instance you created.

## Create a database

After your instance starts running, you can create your database. You define your schema in the database .

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the instance you created, for example, `  test-instance  ` .

3.  In **Overview** , under the name of your instance, click **Create database** .

4.  In **Database name** , enter a database name. For example, `  example-db  ` .

5.  In **Select database dialect** , choose Google Standard SQL.
    
    Spanner vector search isn't available in the PostgreSQL dialect.

6.  Copy and paste the following schema into the **DDL Templates** editor tab. The schema defines a `  Products  ` table.
    
    ``` text
      CREATE TABLE products (
        categoryId INT64 NOT NULL,
        productId INT64 NOT NULL,
        productName STRING(MAX) NOT NULL,
        productDescription STRING(MAX) NOT NULL,
        productDescriptionEmbedding ARRAY<FLOAT32>,
        createTime TIMESTAMP NOT NULL OPTIONS (
        allow_commit_timestamp = true
        ), inventoryCount INT64 NOT NULL,
        priceInCents INT64,
      ) PRIMARY KEY(categoryId, productId);
    ```

7.  Don't make any changes in **Show encryption options** .

8.  Click **Create** . Google Cloud console displays the **Overview** page for the database you created.

## Create an embedding model

When you use the [`  CREATE MODEL  `](/spanner/docs/reference/standard-sql/data-definition-language#create-model) DDL statement in Spanner, you are registering a reference to the Vertex AI model's endpoint from your database. After you register the model, you can use the [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function to access the model in your queries.

The following example demonstrates how to register a Vertex AI [text embedding model](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings) , which is then used to perform similarity search to find similar products in a database.

**Note:** The following schema uses the `  us-central1  ` Vertex AI endpoint. Change this in the `  EmbeddingsModel  ` definition if you chose a different Spanner region when creating the instance.

1.  In the database **Overview** page, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Enter:
    
    ``` text
    CREATE MODEL EmbeddingsModel INPUT(
    content STRING(MAX),
    ) OUTPUT(
    embeddings STRUCT<values ARRAY<FLOAT32>>,
    ) REMOTE OPTIONS (
    endpoint = '//aiplatform.googleapis.com/projects/PROJECT_ID/locations/us-central1/publishers/google/models/TEXT_EMBEDDING_MODEL'
    );
    ```
    
    Replace the following:
    
      - PROJECT\_ID : a permanent identifier that is unique for your Google Cloud project.
      - TEXT\_EMBEDDING\_MODEL : the name of the text embedding model. For a list of the Vertex AI text embedding models, see [Supported models](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) .

4.  Click **Run** to create the model.
    
    After the model is added successfully, you'll see it displayed in the **Explorer** pane.

## Load data

To load the Cymbal sample data into the `  products  ` table, do the following:

1.  In a new tab in **Spanner Studio** , copy and paste the following insert statement:
    
    ``` text
    INSERT INTO products (categoryId, productId, productName, productDescription, createTime, inventoryCount, priceInCents)
    VALUES (1, 1, "Cymbal Helios Helmet", "Safety meets style with the Cymbal children's bike helmet. Its lightweight design, superior ventilation, and adjustable fit ensure comfort and protection on every ride. Stay bright and keep your child safe under the sun with Cymbal Helios!", PENDING_COMMIT_TIMESTAMP(), 100, 10999),
          (1, 2, "Cymbal Sprout", "Let their cycling journey begin with the Cymbal Sprout, the ideal balance bike for beginning riders ages 2-4 years. Its lightweight frame, low seat height, and puncture-proof tires promote stability and confidence as little ones learn to balance and steer. Watch them sprout into cycling enthusiasts with Cymbal Sprout!", PENDING_COMMIT_TIMESTAMP(), 10, 13999),
          (1, 3, "Cymbal Spark Jr.", "Light, vibrant, and ready for adventure, the Spark Jr. is the perfect first bike for young riders (ages 5-8). Its sturdy frame, easy-to-use brakes, and puncture-resistant tires inspire confidence and endless playtime. Let the spark of cycling ignite with Cymbal!", PENDING_COMMIT_TIMESTAMP(), 34, 13900),
          (1, 4, "Cymbal Summit", "Conquering trails is a breeze with the Summit mountain bike. Its lightweight aluminum frame, responsive suspension, and powerful disc brakes provide exceptional control and comfort for experienced bikers navigating rocky climbs or shredding downhill. Reach new heights with Cymbal Summit!", PENDING_COMMIT_TIMESTAMP(), 0, 79999),
          (1, 5, "Cymbal Breeze", "Cruise in style and embrace effortless pedaling with the Breeze electric bike. Its whisper-quiet motor and long-lasting battery let you conquer hills and distances with ease. Enjoy scenic rides, commutes, or errands with a boost of confidence from Cymbal Breeze!", PENDING_COMMIT_TIMESTAMP(), 72, 129999),
          (1, 6, "Cymbal Trailblazer Backpack", "Carry all your essentials in style with the Trailblazer backpack. Its water-resistant material, multiple compartments, and comfortable straps keep your gear organized and accessible, allowing you to focus on the adventure. Blaze new trails with Cymbal Trailblazer!", PENDING_COMMIT_TIMESTAMP(), 24, 7999),
          (1, 7, "Cymbal Phoenix Lights", "See and be seen with the Phoenix bike lights. Powerful LEDs and multiple light modes ensure superior visibility, enhancing your safety and enjoyment during day or night rides. Light up your journey with Cymbal Phoenix!", PENDING_COMMIT_TIMESTAMP(), 87, 3999),
          (1, 8, "Cymbal Windstar Pump", "Flat tires are no match for the Windstar pump. Its compact design, lightweight construction, and high-pressure capacity make inflating tires quick and effortless. Get back on the road in no time with Cymbal Windstar!", PENDING_COMMIT_TIMESTAMP(), 36, 24999),
          (1, 9,"Cymbal Odyssey Multi-Tool","Be prepared for anything with the Odyssey multi-tool. This handy gadget features essential tools like screwdrivers, hex wrenches, and tire levers, keeping you ready for minor repairs and adjustments on the go. Conquer your journey with Cymbal Odyssey!", PENDING_COMMIT_TIMESTAMP(), 52, 999),
          (1, 10,"Cymbal Nomad Water Bottle","Stay hydrated on every ride with the Nomad water bottle. Its sleek design, BPA-free construction, and secure lock lid make it the perfect companion for staying refreshed and motivated throughout your adventures. Hydrate and explore with Cymbal Nomad!", PENDING_COMMIT_TIMESTAMP(), 42, 1299);
    ```

2.  Click **Run** to insert the data.

## Generate vector embeddings

After you register a model and load data into Spanner, you can generate vector embeddings with the product descriptions from your data. Vector embeddings transform text data into a numerical value that captures the meaning and context of words. This transformation is crucial for performing a semantic search.

In this step, you'll populate the `  productDescriptionEmbedding  ` column by generating embeddings from the `  productDescription  ` column using `  ML.PREDICT  ` . This lets you perform a vector similarity search in the next step.

1.  In a new tab in **Spanner Studio** , copy and paste the following update statement:
    
    ``` text
    UPDATE products p1
    SET productDescriptionEmbedding =
      (SELECT embeddings.values
        FROM ML.PREDICT(MODEL EmbeddingsModel,
          (SELECT p1.productDescription as content)
        )
      )
    WHERE categoryId=1;
    ```

2.  Click **Run** to generate the embeddings.

## Perform vector similarity search

In the following example, you provide a natural language search request using a SQL query. The SQL query performs a vector similarity search using the vector embeddings you generated previously. The query performs the search by doing the following:

  - Uses `  ML.PREDICT  ` to generate an embedding for the given search query ("I'd like to buy a starter bike for my 3 year old child").
  - Calculates the [`  COSINE_DISTANCE  `](/spanner/docs/find-k-nearest-neighbors) between this query embedding and the `  productDescriptionEmbedding  ` of each product in the products table to find similar results in your Cymbal store.
  - Filters the results to only include products with an `  inventoryCount  ` greater than 0.
  - Orders the results by the calculated distance and returns the top five closest matches, along with the `  productName  ` , `  productDescription  ` , and `  inventoryCount  ` .

<!-- end list -->

1.  In a new tab in **Spanner Studio** , copy and paste the following query:
    
    ``` text
    SELECT productName, productDescription, inventoryCount,
      COSINE_DISTANCE(
        productDescriptionEmbedding,
        (
          SELECT embeddings.values
          FROM
            ML.PREDICT(
              MODEL EmbeddingsModel,
              (SELECT "I'd like to buy a starter bike for my 3 year old child" AS content))
        )) AS distance
    FROM products
    WHERE inventoryCount > 0
    ORDER BY distance
    LIMIT 5;
    ```

2.  Click **Run** to return the products that best match your search text.
    
    Example output:
    
    ``` text
    /*-----------------+--------------------+----------------+--------------------*
    | productName      | productDescription | inventoryCount | distance           |
    +------------------+--------------------+----------------+--------------------+
    | Cymbal Sprout    | Let their cycling  | 10             | 0.3094387191860244 |
    |                  | journey begin with |                |                    |
    |                  | the Cymbal Sprout, |                |                    |
    |                  | the ideal balance  |                |                    |
    |                  | bike for beginning |                |                    |
    |                  | riders ages 2-4    |                |                    |
    |                  | years...           |                |                    |
    | Cymbal Spark Jr  | Light, vibrant,    | 34             | 0.3412342902117166 |
    |                  | and ready for      |                |                    |
    |                  | adventure, the     |                |                    |
    |                  | Spark Jr. is the   |                |                    |
    |                  | perfect first bike |                |                    |
    |                  | for young riders   |                |                    |
    |                  | (ages 5-8)...      |                |                    |
    | Cymbal Helios    | Safety meets style | 100            | 0.4197863319656684 |
    | Helmet           | with the Cymbal    |                |                    |
    |                  | children's bike    |                |                    |
    |                  | helmet...          |                |                    |
    | Cymbal Breeze    | Cruise in style and| 72             | 0.485231776523978  |
    |                  | embrace effortless |                |                    |
    |                  | pedaling with the  |                |                    |
    |                  | Breeze electric    |                |                    |
    |                  | bike...            |                |                    |
    | Cymbal Phoenix   | See and be seen    | 87             | 0.525101413779242  |
    | Lights           | with the Phoenix   |                |                    |
    |                  | bike lights...     |                |                    |
    *------------------+--------------------+----------------+--------------------*/
    ```

## Scale vector search to use approximate nearest neighbors

The previous vector search example uses exact, [K-nearest neighbors (KNN) vector search](/spanner/docs/find-k-nearest-neighbors) . The KNN vector distance functions (cosine distance, euclidean distance, and dot product) are helpful when you can query a specific subset of your Spanner data. Because KNN search calculates the exact distance between a query vector and all vectors in the database, it is efficient when you can partition the data. If your query needs to compare the query vector to all vectors in your database without any specific filters, and you can't divide the query into independent subqueries, you might experience performance bottlenecks if you use KNN. Approximate nearest neighbors (ANN) vector search becomes useful in these situations. For more information, see [Find approximate nearest neighbors](/spanner/docs/find-approximate-nearest-neighbors) .

If your workloads aren't partitionable, and you have a large amount of data, you can use ANN vector search to increase query performance for larger datasets.

To scale and use ANN vector search in Spanner, do the following:

  - [Create a vector index](#create-vector-index)
  - [Modify your query to use an ANN distance function](#use-ANN-function)

### Create a vector index

Spanner accelerates ANN vector searches by using a specialized vector index that leverages Google Research's [Scalable Nearest Neighbor (ScaNN)](https://github.com/google-research/google-research/tree/master/scann) .

To create a vector index in your dataset, you need to modify the `  productDescriptionEmbeddings  ` column to define a `  vector_length  ` annotation. The `  vector_length  ` annotation indicates the dimension of each vector. The following DDL statements drops the `  productDescriptionEmbedding  ` column, and re-creates it with the `  vector_length  ` . The maximum length (dimension) of the vector varies depending on the [embedding model](/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings#supported-models) you've chosen.

1.  In a new tab in **Spanner Studio** , copy and paste the following DDL statement to re-create the `  productDescriptionEmbedding  ` column:
    
    ``` text
    ALTER TABLE products DROP COLUMN productDescriptionEmbedding;
    ALTER TABLE products
      ADD COLUMN productDescriptionEmbedding ARRAY<FLOAT32>(vector_length=>VECTOR_LENGTH_VALUE);
    ```
    
    Replace `  VECTOR_LENGTH_VALUE  ` with the maximum output dimensions of the embedding model you've chosen.

2.  Click **Run** .

3.  Copy and paste the following insert statement to re-generate the vector embeddings:
    
    ``` text
    UPDATE products p1
    SET productDescriptionEmbedding =
    (SELECT embeddings.values from ML.PREDICT(MODEL EmbeddingsModel,
    (SELECT p1.productDescription as content)))
    WHERE categoryId=1;
    ```

4.  Click **Run** .

5.  Copy and paste the following DDL statement to create the vector index:
    
    ``` text
    CREATE VECTOR INDEX ProductDescriptionEmbeddingIndex
        ON products(productDescriptionEmbedding)
        WHERE productDescriptionEmbedding IS NOT NULL
    OPTIONS (
    distance_type = 'COSINE'
    );
    ```

6.  Click **Run** .

**Note:** In a production environment, instead of dropping the original embedding column first, we recommend that you add a new column with the new `  vector_length  ` definition. Then, re-generate the embeddings in the new column. Finally, modify your applications to use the new column before dropping the original column.

### Use ANN vector distance function

To use ANN vector search in Spanner, modify the following in your SQL query:

  - Generate the prompt embedding separately, instead of within the SQL query.
  - Copy the results of the embeddings into the query.
  - Use the `  FORCE_INDEX  ` hint to reference the new vector index: `  @{force_index=ProductDescriptionEmbeddingIndex}  `
  - Use the `  APPROX_COSINE_DISTANCE  ` vector distance function instead of `  COSINE_DISTANCE  ` . The `  JSON '{"num_leaves_to_search": num_leaves }'  ` option is required.

<!-- end list -->

1.  In a new tab in **Spanner Studio** , copy and paste the following query to generate the prompt embedding and perform vector search:
    
    ``` text
    -- Generate the prompt embedding
    WITH embedding AS (
      SELECT embeddings.values
      FROM ML.PREDICT(
        MODEL EmbeddingsModel,
          (SELECT "I'd like to buy a starter bike for my 3 year old child" as content)
      )
    )
    -- Use embedding to find the most similar entries in the database
    SELECT productName, productDescription, inventoryCount,
      (APPROX_COSINE_DISTANCE(productDescriptionEmbedding,
       embedding.values,
      options => JSON '{"num_leaves_to_search": 10}')) as distance
    FROM products @{force_index=ProductDescriptionEmbeddingIndex}, embedding
    WHERE productDescriptionEmbedding IS NOT NULL AND inventoryCount > 0
    ORDER BY distance
    LIMIT 5;
    ```

2.  Click **Run** .
    
    Example output:
    
    ``` text
    /*-----------------+--------------------+----------------+--------------------*
    | productName      | productDescription | inventoryCount | distance           |
    +------------------+--------------------+----------------+--------------------+
    | Cymbal Sprout    | Let their cycling  | 10             | 0.30935457151661594|
    |                  | journey begin with |                |                    |
    |                  | the Cymbal Sprout, |                |                    |
    |                  | the ideal balance  |                |                    |
    |                  | bike for beginning |                |                    |
    |                  | riders ages 2-4    |                |                    |
    |                  | years...           |                |                    |
    | Cymbal Spark Jr  | Light, vibrant,    | 34             | 0.34116496551593656|
    |                  | and ready for      |                |                    |
    |                  | adventure, the     |                |                    |
    |                  | Spark Jr. is the   |                |                    |
    |                  | perfect first bike |                |                    |
    |                  | for young riders   |                |                    |
    |                  | (ages 5-8)...      |                |                    |
    | Cymbal Helios    | Safety meets style | 100            | 0.4198014303921187 |
    | Helmet           | with the Cymbal    |                |                    |
    |                  | children's bike    |                |                    |
    |                  | helmet...          |                |                    |
    | Cymbal Breeze    | Cruise in style and| 72             | 0.4850674854267337 |
    |                  | embrace effortless |                |                    |
    |                  | pedaling with the  |                |                    |
    |                  | Breeze electric    |                |                    |
    |                  | bike...            |                |                    |
    | Cymbal Phoenix   | See and be seen    | 87             | 0.525101413779242  |
    | Lights           | with the Phoenix   |                |                    |
    |                  | bike lights...     |                |                    |
    *------------------+--------------------+----------------+--------------------*/
    ```
    
    The Cymbal Sprout, with its `  APPROX_COSINE_DISTANCE  ` of 0.30935457151661594, has the highest degree of similarity to the original query.
    
    For more information about interpreting the relationship between vector functions and similarity, see [Choose among vector distance functions to measure vector embeddings similarity](/spanner/docs/choose-vector-distance-function) .

## Clean up

This section shows you how to use the Google Cloud console to clean up your resources. To avoid additional charges to your Cloud Billing account, delete the database and the instance that you created during setup. Deleting an instance deletes all databases created in the instance.

### Delete the database

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the name of the instance that has the database that you want to delete, for example, **test-instance** .

3.  Click the name of the database that you want to delete, for example, **example-db** .

4.  On the **Database overview** page, click delete **Delete database** .

5.  Confirm that you want to delete the database by entering the database name and clicking **Delete** .

### Delete the instance

**Warning:** Deleting an instance permanently removes the instance and all its databases. You can't undo this later. Also, if you're using a free trial instance, you can't create another free trial instance after you delete your first free trial instance. You can create one free trial instance per project lifecycle.

1.  In Google Cloud console, go to the **Spanner Instances** page.

2.  Click the name of the instance that you want to delete, for example, **test-instance** .

3.  Click **Delete instance** .

4.  Confirm that you want to delete the instance by entering the instance name and clicking **Delete** .

## What's next?

  - Learn more about Spanner's [k-nearest neighbor (KNN) feature](/spanner/docs/find-k-nearest-neighbors) .
  - Learn more about Spanner's [approximate nearest neighbor (ANN) feature](/spanner/docs/find-approximate-nearest-neighbors) .
  - Learn more about how to [perform online predictions with SQL using Vertex AI](/spanner/docs/ml) .
