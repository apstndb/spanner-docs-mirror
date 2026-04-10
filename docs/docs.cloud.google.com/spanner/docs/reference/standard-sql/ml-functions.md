GoogleSQL for Spanner supports the following machine learning (ML) functions.

## Function list

| Name                                                                                                                        | Summary                                                                      |
| --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| [`         AI.CLASSIFY        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/ml-functions#aiclassify) | Classifies a natural language input into one of specified categories.        |
| [`         AI.IF        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/ml-functions#aiif)             | Evaluates a natural language condition.                                      |
| [`         AI.SCORE        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/ml-functions#aiscore)       | Rates a natural language input and assigns it a score.                       |
| [`         ML.PREDICT        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/ml-functions#mlpredict)   | Applies ML computations defined by a model to each row of an input relation. |

## `     AI.CLASSIFY    `

    AI.CLASSIFY(prompt, categories)

**Description**

Classifies the input prompt value into categories that you provide.

The following are common use cases:

  - **Retail** : Classify reviews by sentiment or classify products by categories.
  - **Text analysis** : Classify support tickets or emails by topic.

**Definitions**

  - `  prompt  ` : A `  STRING  ` value that specifies the input to classify. For example, `  'apple'  ` .

  - `  categories  ` : The categories by which to classify the input. You can specify categories with or without descriptions:
    
      - With descriptions: Use an `  ARRAY<STRUCT<STRING, STRING>>  ` value where each struct contains the category name, followed by a description of the category. The array can only contain string literals. For example, you can use colors to classify sentiment:
        
            [('green', 'positive'), ('yellow', 'neutral'), ('red', 'negative')]
        
        You can optionally name the fields of the struct for your own readability, but the field names aren't used by the function:
        
            [STRUCT('green' AS label, 'positive' AS description),
            STRUCT('yellow' AS label, 'neutral' AS description),
            STRUCT('red' AS label, 'negative' AS description)]
    
      - Without descriptions: Use an `  ARRAY<STRING>  ` value. The array can only contain string literals. This works well when your categories are self-explanatory. For example, you can use the following categories to classify sentiment:
        
        `  ['positive', 'neutral', 'negative']  `
    
    To handle input that doesn't closely match a category, consider including an `  'Other'  ` category.

**Details**

`  AI.CLASSIFY  ` returns a `  STRING  ` value containing the provided category that best fits the input.

The prompt value is evaluated using [Vertex AI Gemini LLM](https://docs.cloud.google.com/vertex-ai/generative-ai/docs) in the same project as the database. Enable the API before calling this function. [pricing](https://cloud.google.com/vertex-ai/pricing) applies.

If the call to is unsuccessful for any reason, such as exceeding quota or model unavailability, then the function produces an error. In safe mode, the function returns `  NULL  ` instead.

**Return Type**

`  STRING  `

**Examples**

The following query categorizes news articles into high-level categories:

    -- Classify text by topic
    SELECT
      title,
      body,
      AI.CLASSIFY(
        body,
        ['tech', 'sport', 'business', 'politics', 'entertainment', 'other']) AS category
    FROM
      news
    LIMIT 100;

The following query classifies movie reviews of *The English Patient* by sentiment according to a custom color scheme. For example, a review that is positive is classified as 'green'.

    -- Classify reviews by sentiment
    SELECT
      AI.CLASSIFY(
        ('Classify the review by sentiment: ', review),
        categories =>
             [('green', 'The review is positive.'),
              ('yellow', 'The review is neutral.'),
              ('red', 'The review is negative.')]) AS ai_review_rating,
      reviewer_rating AS human_provided_rating,
      review,
    FROM
      reviews
    WHERE
      title = 'The English Patient'

## `     AI.IF    `

    AI.IF(prompt)

**Description**

Evaluates a condition described in natural language and returns a `  BOOL  ` .

You can use the `  AI.IF  ` function to filter and join data based on conditions described in natural language or multimodal input. The following are common use cases:

  - **Sentiment analysis** : Find customer reviews with negative sentiment.
  - **Topic analysis** : Identify news articles related to a specific subject.
  - **Image analysis** : Select images that contain a specific item.
  - **Security** : Identify suspicious emails.

**Definitions**

  - `  prompt  ` : A `  STRING  ` value that specified the prompt to send to the model. For example, `  'Is Seattle a US city?'  ` .

**Details**

The prompt value is evaluated using [Vertex AI Gemini LLM](https://docs.cloud.google.com/vertex-ai/generative-ai/docs) in the same project as the database. Enable the API before calling this function. [pricing](https://cloud.google.com/vertex-ai/pricing) applies.

If the call to is unsuccessful for any reason, such as exceeding quota or model unavailability, then the function produces an error. In safe mode, the function returns `  NULL  ` instead.

**Return Type**

`  BOOL  `

**Examples**

The following query uses the `  AI.IF  ` function to filter news stories to those that cover a natural disaster:

    -- Filter text by topic
    SELECT
      title, body
    FROM
      news
    WHERE
      AI.IF(CONCAT(
        'The following news story is about a natural disaster: ', body));

The following query first filters by the equality operator, and then filters using the `  AI.IF  ` function:

    -- Combine filters
    SELECT
      title, body
    FROM
       news
    WHERE
      AI.IF(CONCAT('This news is related to Google: ', body))
      AND category = 'tech'; -- Non-AI filters are evaluated first

The following query joins tables based on whether the news is about a product:

    -- Join tables based on semantic understanding.
    SELECT
      news.title, news.body, products.product_name
    FROM
      news
    JOIN products
      ON
        AI.IF(
          CONCAT(
            'Determine if the following news story is about product ',
            products.product_name,
            ': ',
            news.body))
    WHERE
      category = 'tech';

## `     AI.SCORE    `

    AI.SCORE(prompt)

**Description**

Rates inputs based on a scoring system that you describe and returns a `  FLOAT64  ` value.

Use the `  AI.SCORE  ` function with the `  ORDER BY  ` clause to rank items. The following are common use cases:

  - **Retail** : Find the top 5 most negative customer reviews about a product.
  - **Hiring** : Find the top 10 resumes that appear most qualified for a job post.
  - **Customer success** : Find the top 20 best customer support interactions.

**Definitions**

  - `  prompt  ` : A `  STRING  ` value that specifies the input to score. For example, `  CONCAT('Rate this review: ', review_col, ' Use a scale from 1-10.')  ` .

**Details**

`  AI.SCORE  ` returns a `  FLOAT64  ` indicating the score assigned to the input. There is no fixed default range for the score. For best results, provide a scoring range in the prompt.

The prompt value is evaluated using [Vertex AI Gemini LLM](https://docs.cloud.google.com/vertex-ai/generative-ai/docs) in the same project as the database. Enable the API before calling this function. [pricing](https://cloud.google.com/vertex-ai/pricing) applies.

If the call to is unsuccessful for any reason, such as exceeding quota or model unavailability, then the function produces an error. In safe mode, the function returns `  NULL  ` instead.

**Return Type**

`  FLOAT64  `

**Examples**

The following query uses the `  AI.SCORE  ` function to assign ratings based on movie reviews of *The English Patient* , alongside the ratings that the human reviewers gave. It returns the top 10 highest AI rated reviews.

    -- Rate reviews
    SELECT
      AI.SCORE(CONCAT(
        """
        On a scale from 1 to 10, rate how much the reviewer liked the movie.
        Review:
        """, review)) AS ai_rating,
      reviewer_rating AS human_rating,
      review
    FROM
      reviews
    WHERE
      title = 'The English Patient'
    ORDER BY ai_rating DESC
    LIMIT 10;

The following query builds on the previous example by using the `  AI.IF  ` function to filter the results to show reviews that mention at least one of the film's main characters:

    -- Rate and filter reviews
    SELECT
      AI.SCORE(
        CONCAT(
          'On a scale from 1 to 10, rate how much the reviewer liked the movie. Review:',
          review)) AS ai_rating,
      reviewer_rating AS human_rating,
      review
    FROM
      reviews
    WHERE
      title = 'The English Patient'
      AND AI.IF(
        CONCAT("This review mentions at least one of the film's main characters: ", review))
    ORDER BY ai_rating DESC
    LIMIT 10;

## `     ML.PREDICT    `

    ML.PREDICT(input_model, input_relation[, model_parameters])
    
    input_model:
      MODEL model_name
    
    input_relation:
      { input_table | input_subquery }
    
    input_table:
      TABLE table_name
    
    model_parameters:
      STRUCT(parameter_value AS parameter_name[, ...])

**Description**

`  ML.PREDICT  ` is a table-valued function that helps to access registered machine learning (ML) models and use them to generate ML predictions. This function applies ML computations defined by a model to each row of an input relation, and returns the results of those predictions. Additionally, you can use `  ML.PREDICT  ` to perform vector search. When you use `  ML.PREDICT  ` for vector search, it converts your natural language query text into an embedding.

**Note:** Make sure that Spanner has access to the referenced Vertex AI endpoint as described in [Model endpoint access control](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create_model_permissions) .

**Supported Argument Types**

  - `  input_model  ` : The model to use for predictions. Replace `  model_name  ` with the name of the model. To create a model, see [`  CREATE_MODEL  `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create_model) .
  - `  input_relation  ` : A table or subquery upon which to apply ML computations. The set of columns of the input relation must include all input columns of the input model; otherwise, the input won't have enough data to generate predictions and the query won't compile. Additionally, the set can also include arbitrary pass-through columns that will be included in the output. The order of the columns in the input relation doesn't matter. The columns of the input relation and model must be coercible.
  - `  input_table  ` : The table containing the input data for predictions, for example, a set of features. Replace `  table_name  ` with the name of the table.
  - `  input_subquery  ` : The subquery that's used to generate the prediction input data.
  - `  model_parameters  ` : A `  STRUCT  ` value that contains parameters supported by `  model_name  ` . These parameters are passed to the model inference.

**Return Type**

A table with the following columns:

  - Model outputs
  - Pass-through columns from the input relation

**Note:** If a column of the input relation has the same name as one of the output columns, the value of the output column is returned.

**Examples**

The examples in this section reference a model called `  DiamondAppraise  ` and an input table called `  Diamonds  ` with the following columns:

  - `  DiamondAppraise  ` model:
    
    | Input columns                        | Output columns                           |
    | ------------------------------------ | ---------------------------------------- |
    | `          value FLOAT64         `   | `          value FLOAT64         `       |
    | `          carat FLOAT64         `   | `          lower_bound FLOAT64         ` |
    | `          cut STRING         `      | `          upper_bound FLOAT64         ` |
    | `          color STRING(1)         ` |                                          |
    

  - `  Diamonds  ` table:
    
    | Columns                            |
    | ---------------------------------- |
    | `          Id INT64         `      |
    | `          Carat FLOAT64         ` |
    | `          Cut STRING         `    |
    | `          Color STRING         `  |
    

The following query predicts the value of a diamond based on the diamond's carat, cut, and color.

    SELECT id, color, value
    FROM ML.PREDICT(MODEL DiamondAppraise, TABLE Diamonds);
    
    +----+-------+-------+
    | id | color | value |
    +----+-------+-------+
    | 1  | I     | 280   |
    | 2  | G     | 447   |
    +----+-------+-------+

You can include model-specific parameters. For example, in the following query, the `  maxOutputTokens  ` parameter specifies that `  output  ` , the model inference, can contain 10 or fewer tokens. This query succeeds because the model `  TextBison  ` contains a parameter called `  maxOutputTokens  ` .

    SELECT prompt, output
    FROM ML.PREDICT(
      MODEL TextBison,
      (SELECT "Is 13 prime?" as prompt), STRUCT(10 AS maxOutputTokens));
    
    +----------------+---------------------+
    | prompt         | output             |
    +----------------+---------------------+
    | "Is 13 prime?" | "Yes, 13 is prime." |
    +----------------+---------------------+

The following example generates an embedding for a natural language query. The example then uses that embedding to find the most similar entries in a database that are indexed by vector embeddings.

    -- Generate the embedding from a natural language prompt
    WITH embedding AS (
      SELECT embeddings.values
      FROM ML.PREDICT(
        MODEL DiamondAppraise,
          (SELECT "What is the most valuable diamond?" as prompt)
      )
    )
    -- Use embedding to find the most similar entries in the database
    SELECT id, color, value,
      (APPROX_COSINE_DISTANCE(valueEmbedding,
      embedding.values,
      options => JSON '{"num_leaves_to_search": 10}')) as distance
    FROM products @{force_index=valueEmbeddingIndex}, embedding
    WHERE valueEmbedding IS NOT NULL
    ORDER BY distance
    LIMIT 5;

You can use `  ML.PREDICT  ` in any DQL/DML statements, such as `  INSERT  ` or `  UPDATE  ` . For example:

    INSERT INTO AppraisedDiamond (id, color, carat, value)
    SELECT
      1 AS id,
      color,
      carat,
      value
    FROM
      ML.PREDICT(MODEL DiamondAppraise,
      (
        SELECT
          @carat AS carat,
          @cut AS cut,
          @color AS color
      ));
