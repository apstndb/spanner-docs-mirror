**PostgreSQL interface note:** Spanner Vertex AI integration is not supported in PostgreSQL-dialect databases.

## Objective

In this tutorial, you learn how to:

  - Use Google provided Vertex AI [Generative AI](/vertex-ai/generative-ai/docs/overview) models in a Spanner database.
  - Use [Generative AI](/vertex-ai/generative-ai/docs/overview) to provide personalized product recommendations in a sample ecommerce application.

## Costs

This tutorial uses billable components of Google Cloud, including:

  - Spanner
  - Vertex AI

For more information about Spanner costs, see the [Spanner pricing](/spanner/pricing) page.

For more information about Vertex AI costs, see the [Vertex AI pricing](/vertex-ai/pricing) page.

## Create the ecommerce website schema

For this tutorial, we use the following schema and data:

``` text
CREATE TABLE Products (
  id INT64,
  name STRING(MAX),
  description STRING(MAX),
  category_id INT64,
) PRIMARY KEY(id);

CREATE TABLE Categories (
  id INT64,
  name STRING(MAX)
) PRIMARY KEY(id);

CREATE TABLE Users (
  id INT64,
  age INT64,
  likes STRING(MAX)
) PRIMARY KEY(id);

INSERT INTO Categories (id, name) VALUES
    (1, "Toys"),
    (2, "Tools");

INSERT INTO Products (id, name, description, category_id) VALUES
    (1, "Plush Bear", "Really fluffy. Safe for infants.", 1),
    (2, "Bike", "Bike for teenagers.", 1),
    (3, "Drill", "Cordless.", 2);

INSERT INTO Users (id, age, likes) VALUES
    (1, 30, "DIY"),
    (2, 14, "Toys");
```

## Register a Generative AI model in a Spanner schema

In this tutorial, we use the Vertex AI [text-bison](/vertex-ai/docs/generative-ai/model-reference/text) model to provide personalized product recommendations to end customers. To register this model in a Spanner database, [execute](/spanner/docs/schema-updates) the following DDL [statement](/spanner/docs/reference/standard-sql/data-definition-language#create_model) :

``` text
CREATE MODEL TextBison
INPUT (prompt STRING(MAX))
OUTPUT (content STRING(MAX))
REMOTE
OPTIONS (
  endpoint = '//aiplatform.googleapis.com/projects/PROJECT/locations/LOCATION/publishers/google/models/text-bison'
);
```

Replace the following:

  - `  PROJECT  ` : the project ID
  - `  LOCATION  ` : the region where you are using Vertex AI

Schema discovery and validation isn't available for Generative AI models. Therefore, you must provide `  INPUT  ` and `  OUTPUT  ` clauses that match the model's schema. You can find the full schema of the `  text-bison  ` model on the Vertex AI [Model API reference](/vertex-ai/docs/generative-ai/model-reference/text) page.

As long as both the database and endpoints are in the same project, Spanner should grant appropriate permissions automatically. Otherwise, review the [model endpoint access control](/spanner/docs/reference/standard-sql/data-definition-language#model_endpoint_access_control) section of the `  CREATE MODEL  ` reference page.

To verify the model was registered correctly, query it with the [ML.PREDICT](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function. The model expects a single `  STRING  ` column named `  prompt  ` . You can use a [Spanner subquery](/spanner/docs/reference/standard-sql/subqueries) to generate the `  prompt  ` column. The `  TextBison  ` model requires you to specify a `  maxOutputTokens  ` model parameter. Other parameters are optional. The Vertex AI `  text-bison  ` model doesn't support batching, so you must use the `  @{remote_udf_max_rows_per_rpc=1}  ` parameter to set the batch size to 1.

``` text
SELECT content
FROM ML.PREDICT(
  MODEL TextBison,
  (SELECT "Is 13 prime?" AS prompt),
  STRUCT(256 AS maxOutputTokens, 0.2 AS temperature, 40 as topK, 0.95 AS topP)
) @{remote_udf_max_rows_per_rpc=1};

+--------------------+
| content            |
+--------------------+
| "Yes, 13 is prime" |
+--------------------+
```

## Use the `     TextBison    ` Model to answer customer questions

Generative AI text models can solve a wide array of problems. For example, a user on an ecommerce website might be browsing for products that are safe for infants. With a single query, we can pass their question to the `  TextBison  ` model. All we need to do is provide relevant context for the question by fetching product details from the database.

NOTE: Some model answers were edited for brevity.

``` text
SELECT product_id, product_name, content
FROM ML.PREDICT(
  MODEL TextBison,
  (SELECT
    product.id as product_id,
    product.name as product_name,
    CONCAT("Is this product safe for infants?", "\n",
        "Product Name: ", product.name, "\n",
        "Category Name: ", category.name, "\n",
        "Product Description:", product.description) AS prompt
   FROM
     Products AS product JOIN Categories AS category
       ON product.category_id = category.id),
  STRUCT(100 AS maxOutputTokens)
) @{remote_udf_max_rows_per_rpc=1};

-- The model correctly recommends a Plush Bear as safe for infants.
-- Other products are not safe and the model provides justification why.
+------------+-----------------+--------------------------------------------------------------------------------------------------+
| product_id | product_name    | content                                                                                          |
+------------+-----------------+--------------------------------------------------------------------------------------------------+
|          1 | "Plush Bear"    | "Yes, this product is infant safe. [...] "                                                       |
|            |                 | "The product description says that the product is safe for infants. [...]"                       |
+------------+-----------------+--------------------------------------------------------------------------------------------------+
|          2 | "Bike"          | "No, this product is not infant safe. [...] "                                                    |
|            |                 | "It is not safe for infants because it is too big and heavy for them to use. [...]"              |
+------------+-----------------+--------------------------------------------------------------------------------------------------+
|          3 | "Drill"         | "No, this product is not infant safe. [...]"                                                     |
|            |                 | " If an infant were to grab the drill, they could pull it on themselves and cause injury. [...]" |
+------------+-----------------+--------------------------------------------------------------------------------------------------+
```

You can replace the question literal with a query parameter, such as `  @UserQuestion  ` , if you want to directly populate the parameter with a customer question. This gives the customer an AI-powered online shopping experience.

## Provide personalized product recommendations to customers

In addition to product details, we can also add information about the customer to the `  prompt  ` . This lets the model take user preferences into consideration so that it can provide fully personalized product recommendations.

``` text
SELECT product_id, product_name, content
FROM ML.PREDICT(
  MODEL TextBison,
  (SELECT
    product.id as product_id,
    product.name as product_name,
    CONCAT(
        "Answer with YES or NO only: Is this a good fit for me?",
        "My age:", CAST(user.age AS STRING), "\n",
        "I like:", user.likes,  "\n",
        "Product name: ", product.name, "\n",
        "Category mame: ", category.name, "\n",
        "Product description:", product.description) AS prompt,
   FROM
     Products AS product
       JOIN Categories AS category ON product.category_id = category.id
       JOIN Users AS user ON user.id = 1),
  STRUCT(256 AS maxOutputTokens)
) @{remote_udf_max_rows_per_rpc=1};

-- The model correctly guessed that the user might be interested in a Drill
-- as they are interested in DIY.
+------------+-----------------+-------------+
| product_id | product_name    | content     |
+------------+-----------------+-------------+
|          1 | "Plush Bear"    | "NO"        |
+------------+-----------------+-------------+
|          2 | "Bike"          | "NO"        |
+------------+-----------------+-------------+
|          3 | "Drill"         | "YES"       |
+------------+-----------------+-------------+
```

To look for a gift for their child, the user can create a profile for their teenager and see a different list of recommendations:

``` text
SELECT product_id, product_name, content
FROM ML.PREDICT(
  MODEL TextBison,
  (SELECT
    product.id as product_id,
    product.name as product_name,
    CONCAT(
        "Answer with YES or NO only: Is this a good fit for me?",
        "\nMy's age:", CAST(user.age AS STRING),
        "\nI like:", user.likes,
        "\nProduct Name: ", product.name,
        "\nCategory Name: ", category.name,
        "\nProduct Description:", product.description) AS prompt,
   FROM
     Products AS product
       JOIN Categories AS category ON product.category_id = category.id
       JOIN Users AS user ON user.id = 2),
  STRUCT(40 AS maxOutputTokens)
) @{remote_udf_max_rows_per_rpc=1};

-- The model correctly guesses that a teenager is interested in a Bike,
-- but not a plush bear for infants or spicy peppers.
+------------+-----------------+---------+
| product_id | product_name    | content |
+------------+-----------------+---------+
|          1 | "Plush Bear"    | "NO"    |
+------------+-----------------+---------+
|          2 | "Bike"          | "YES"   |
+------------+-----------------+---------+
|          3 | "Spicy peppers" | "NO"    |
+------------+-----------------+---------+
```

You can add purchase history or other relevant details to the prompt to give the customer a more customized experience.

Spanner Vertex AI integration helps you assemble complex prompts containing live data and use them to build AI-enabled applications.
