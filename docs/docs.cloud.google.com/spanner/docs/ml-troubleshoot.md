MODEL DDL statements and ML functions in Spanner invoke Vertex AI endpoints and can fail due to various reasons:

<table>
<thead>
<tr class="header">
<th>Error Code</th>
<th>Error Message</th>
<th>Possible cause</th>
<th>Possible solution</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>CANCELLED</td>
<td>Call to Vertex AI endpoint {ENDPOINT} was cancelled due to query cancellation.</td>
<td>Query was cancelled by the client application.</td>
<td>Investigate why your client cancelled the query.</td>
</tr>
<tr class="even">
<td>DEADLINE_EXCEEDED</td>
<td>Vertex AI endpoint {ENDPOINT} exceeded the call deadline.</td>
<td>Query deadline is too short.</td>
<td>Increase query deadline on the client.</td>
</tr>
<tr class="odd">
<td>-</td>
<td>-</td>
<td>Endpoint was too busy.</td>
<td>See Vertex AI monitoring and deploy more nodes.</td>
</tr>
<tr class="even">
<td>FAILED_PRECONDITION</td>
<td>Vertex AI endpoint {ENDPOINT} returned failed precondition error.</td>
<td>Endpoint has no models deployed.</td>
<td>Deploy models to the endpoint.</td>
</tr>
<tr class="odd">
<td>INTERNAL</td>
<td>Unknown error when accessing Vertex AI endpoint {ENDPOINT}.</td>
<td>Unexpected internal error.</td>
<td><a href="/spanner/docs/ml-tutorial#failover_models">Use failover endpoints</a> or open a support ticket.</td>
</tr>
<tr class="even">
<td>INVALID_ARGUMENT</td>
<td>Invalid request to Vertex AI endpoint {ENDPOINT}. Make sure that Vertex AI endpoint and Spanner model schema match.</td>
<td>Spanner model schema and Vertex AI endpoint schema do not match</td>
<td>Update Spanner model's schema.</td>
</tr>
<tr class="odd">
<td>NOT_FOUND</td>
<td>Vertex AI endpoint {ENDPOINT} not found.</td>
<td>Endpoint was deleted.</td>
<td>Update Spanner model's schema.</td>
</tr>
<tr class="even">
<td>PERMISSION_DENIED</td>
<td>Access to Vertex AI endpoint {ENDPOINT} was denied.</td>
<td>Spanner service agent does not have permissions to access the endpoint</td>
<td><a href="/spanner/docs/ml-tutorial#configure_access_for_to_endpoints">Grant service agent role permissions</a></td>
</tr>
<tr class="odd">
<td>-</td>
<td>-</td>
<td>VPC SC error</td>
<td>See Vertex AI error message and follow <a href="/vpc-service-controls/docs/troubleshooting">VPC SC troubleshooting</a></td>
</tr>
<tr class="even">
<td>RESOURCE_EXHAUSTED</td>
<td>Vertex AI endpoint {ENDPOINT} quota has been exceeded.</td>
<td>Too many requests to Vertex AI.</td>
<td><a href="/vertex-ai/docs/quotas">Increase online prediction quota</a></td>
</tr>
<tr class="odd">
<td>UNAVAILABLE</td>
<td>Could not create service agent for project {PROJECT}.</td>
<td>Service Usage API issue.</td>
<td><a href="/spanner/docs/ml-tutorial#configure_access_for_to_endpoints">Create service agent manually</a></td>
</tr>
<tr class="even">
<td>-</td>
<td>Vertex AI endpoint {ENDPOINT} is unavailable.</td>
<td>Too many requests to Vertex AI.</td>
<td>Deploy more nodes.</td>
</tr>
<tr class="odd">
<td>-</td>
<td>-</td>
<td>Vertex AI has a regional issue</td>
<td><a href="/spanner/docs/ml-tutorial#failover_models">Use failover endpoints</a></td>
</tr>
</tbody>
</table>

**Note:** Errors returned by Spanner include the original Vertex AI errors as `  google.rpc.Status  ` [details message](/apis/design/errors#error_details) . You can see [Vertex AI toubleshooting page](/vertex-ai/docs/general/troubleshooting) for more information.

**Note:** Both Spanner and Vertex AI provide monitoring dashboards and write logs to Cloud Logging, which can contain additional details.
