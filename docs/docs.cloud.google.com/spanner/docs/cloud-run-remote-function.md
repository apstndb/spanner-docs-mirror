**Preview — Remote Functions in Spanner**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

A Spanner remote function lets you implement your function in languages other than SQL. Functions must be hosted in [Cloud Run functions](https://cloud.google.com/functions) or [Cloud Run](https://cloud.google.com/run) . Hosting functions in this manner allows complex business logic to be split out into separate remote functions.

A typical remote function deployment has the following steps:

1.  [Create an HTTPS endpoint](#create-endpoint) in Cloud Run functions or Cloud Run.
    
      - If you are new to remote functions, we recommend using Cloud Run functions.

2.  [Create a remote function](#create-a-remote-function) in Spanner pointing to that HTTPS endpoint.

3.  [Use the remote function](#use-remote-function) in a query.

## Required roles

To ensure that your Spanner agent service account (service- PROJECT\_ID @gcp-sa-spanner.iam.gserviceaccount.com) has the necessary permissions to use Spanner remote functions, ask your administrator to grant your Spanner agent service account (service- PROJECT\_ID @gcp-sa-spanner.iam.gserviceaccount.com) the [Spanner API Service Agent](/iam/docs/roles-permissions/spanner#spanner.serviceAgent) ( `  roles/spanner.serviceAgent  ` ) IAM role on the project.

**Important:** You must grant this role to your Spanner agent service account (service- PROJECT\_ID @gcp-sa-spanner.iam.gserviceaccount.com), *not* to your user account. Failure to grant the role to the correct principal might result in permission errors.

For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

Your administrator might also be able to give your Spanner agent service account (service- PROJECT\_ID @gcp-sa-spanner.iam.gserviceaccount.com) the required permissions through [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Supported types

Remote functions support the following data types as argument or return types:

  - `  ARRAY  ` (of any of the following supported types)
  - `  BOOLEAN  `
  - `  BYTES  `
  - `  DATE  `
  - `  JSON  `
  - `  INTEGER  `
  - `  NUMERIC  `
  - `  STRING  `
  - `  TIMESTAMP  `

## Limitations

  - You cannot create table-valued remote functions.

  - Remote functions are not supported in generated column expressions.

  - You might see repeated requests with the same data to your endpoint, sometimes even after successful responses, due to transient network errors or Spanner internal restarts.

  - Running remote functions in fractional Spanner instances is not supported.

  - Remote functions are not supported for Cloud Run functions behind custom domains.

  - Remote functions are not supported in PostgreSQL dialect.

## Create an endpoint

The business logic must be implemented as a Cloud Run function or Cloud Run. The endpoint must be able to process a batch of rows in a single HTTPS POST request and return the result for the batch as an HTTPS response.

If you have created remote functions for [BigQuery](/bigquery/docs/remote-functions) , you can reuse them for Spanner.

See the [Cloud Run functions tutorial](/functions/docs/tutorials/http) and other [Cloud Run functions documentation](/functions/docs/writing/http) on how to write, deploy, test and maintain a Cloud Run function.

See the [Cloud Run quick start](/run/docs/quickstarts#build-and-deploy-a-web-service) and other [Cloud Run documentation](/run/docs/how-to) on how to write, deploy, test and maintain a Cloud Run service.

We recommend maintaining the default authentication for your Cloud Run function or Cloud Run service. Avoid configuring the service to allow unauthenticated invocation.

### Input format

Spanner sends HTTPS POST requests with JSON bodies in the following format:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Field name</th>
<th>Description</th>
<th>Field type</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       requestId      </code></td>
<td>Request identifier. Unique over multiple requests sent to the endpoint in a GoogleSQL query.</td>
<td>Always provided. String.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       calls      </code></td>
<td>A batch of input data.</td>
<td>Always provided. A JSON array.
<p>Each element is a JSON array that represents a JSON-encoded argument list for a single remote function call.</p></td>
</tr>
</tbody>
</table>

An example of a request:

``` text
// Sample request to a Cloud Run functions to calculate sum of two numbers. This request
// has two calls batched together into a single request.
{
 "requestId": "124ab1c",
 "calls": [
  [1, 2],
  [3, 4]
 ]
}
```

### Output format

Spanner expects the endpoint to return an HTTPS response in the following format; otherwise, Spanner can't consume the response and the query fails calling the remote function.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td>Field name</td>
<td>Description</td>
<td>Value Range</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       replies      </code></td>
<td>A batch of return values.</td>
<td>JSON array.
<p>Each element corresponds to a JSON encoded return value of the external function.</p>
Size of the array must match the size of the JSON array of <code dir="ltr" translate="no">       calls      </code> in the HTTPS request. For example, if the JSON array in <code dir="ltr" translate="no">       calls      </code> has 4 elements, this JSON array needs to have 4 elements as well. Required for a successful response.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       errorMessage      </code></td>
<td>Error message when the HTTPS response code other than 200 is returned. For non-retryable errors, Spanner returns this error message to the user. Required in failed responses. Typically less than 1 KB.</td>
<td>String.</td>
</tr>
</tbody>
</table>

An example of a successful response:

``` text
// Sample response from the Cloud Run functions which has the sum of the two numbers. Note
// that the order of the values within `replies` field matches the `calls` field from
// the request.
{
  "replies": [
    3,  // 1 + 2 = 3
    7   // 3 + 4 = 7
  ]
}
```

An example of a failed response:

``` text
{
  // The error message returned by your Cloud Run functions to indicate an error.
  // In this sample, the error message states that an overflow occurred when summing two numbers.
  "errorMessage": "Overflow detected when calculating sum of two numbers."
}
```

#### HTTPS response code

Your endpoint returns a 200 HTTPS code for a successful response. When Spanner receives any other value, it considers the response a failure, and retries when the HTTPS response code is 408, 429, 500, 503 or 504 until an internal limit is met.

### Sample code

### Cloud Run function

The following sample Python code implements adding all the integer arguments of the remote function. It handles a request with the arguments for batched invocations and returns all the results in a response.

``` text
"""
Python script which uses Flask framework to spin up a HTTP server to take
integers and return their sum. In case of overflow, it returns the error
as part of the response.
"""
import functions_framework

from flask import jsonify

# Max INT64 value encoded as a number in JSON by TO_JSON_STRING. Larger values are encoded as
# strings.
_MAX_LOSSLESS=9007199254740992

@functions_framework.http
def batch_add(request):
  try:
    return_value = []
    request_json = request.get_json()
    calls = request_json['calls']
    for call in calls:
      return_value.append(sum([int(x) if isinstance(x, str) else x for x in call if x is not None]))
    replies = [str(x) if x > _MAX_LOSSLESS or x < -_MAX_LOSSLESS else x for x in return_value]
    return_json = jsonify( { "replies":  replies } )
    return return_json
  except Exception as e:
    return jsonify( { "errorMessage": str(e) } ), 400
```

Assuming that the function is deployed in the project PROJECT\_ID in region `  us-east1  ` as the function name `  remote_add  ` , it can be accessed using the endpoint `  https://us-east1- PROJECT_ID .cloudfunctions.net/remote_add  ` .

### Cloud Run

The following sample Python code implements a web service, which can be built and deployed to Cloud Run for the same functionality.

``` text
"""
Python script which uses Flask framework to spin up a HTTP server to take
integers and return their sum. In case of overflow, it returns the error
as part of the response.
"""
import os

from flask import Flask, request, jsonify

# Max INT64 value encoded as a number in JSON by TO_JSON_STRING. Larger values are encoded as
# strings.
_MAX_LOSSLESS=9007199254740992

app = Flask(__name__)

@app.route("/", methods=['POST'])
def batch_add():
  try:
    return_value = []
    request_json = request.get_json()
    calls = request_json['calls']
    for call in calls:
      return_value.append(sum([int(x) if isinstance(x, str) else x for x in call if x is not None]))
    replies = [str(x) if x > _MAX_LOSSLESS or x < -_MAX_LOSSLESS else x for x in return_value]
    return jsonify( { "replies" :  replies } )
  except Exception as e:
    return jsonify( { "errorMessage": str(e) } ), 400

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=int(os.environ.get("PORT", 8080)))
```

To build and deploy the code, see [Quickstart: Build and deploy a Python (Flask) web app to Cloud Run](/run/docs/quickstarts/build-and-deploy/deploy-python-service) .

Assuming that the Cloud Run service is deployed in the project PROJECT\_ID in region `  us-east1  ` as the service name `  remote_add  ` , it can be accessed using the endpoint `  https://remote_add-<project_id_hash>-ue.a.run.app  ` .

## Create a remote function

To create a remote function:

### SQL

Run the following `  CREATE FUNCTION  ` statement in Spanner:

``` text
CREATE FUNCTION REMOTE_FUNCTION_NAME(x INT64, y INT64) RETURNS INT64 NOT DETERMINISTIC LANGUAGE REMOTE OPTIONS (
  endpoint = `ENDPOINT_URL`,
  max_batching_rows = MAX_BATCHING_ROWS
);
```

Replace the following:

  - `  REMOTE_FUNCTION_NAME  ` : the name of your remote function. For example, `  sum_func  ` .
  - `  ENDPOINT_URL  ` : the Cloud Run functions or Cloud Run endpoint created in the prior step.
  - `  MAX_BATCHING_ROWS  ` (optional): the maximum number of rows to be sent as part of a request. If not specified, Spanner determines the batch size automatically.

## Use remote function in a query

To call the remote function from the previous step within a query, use the following example:

``` text
SELECT REMOTE_FUNCTION_NAME(1, 2); -- 1 + 2 = 3
```

## Pricing

  - Standard [Spanner pricing](https://cloud.google.com/spanner/pricing) applies.

  - Spanner bills for bytes sent to and received from Cloud Run functions or Cloud Run.

  - This feature may incur costs from Cloud Run functions and Cloud Run. Review the [Cloud Run functions](https://cloud.google.com/functions/pricing) and [Cloud Run](https://cloud.google.com/run/pricing) pricing pages for details.
