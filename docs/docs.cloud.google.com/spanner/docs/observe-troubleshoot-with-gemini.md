This document describes how you can use Gemini assistance to help you use system insights to optimize and troubleshoot your Spanner resources.

**Note:** Query insights aren't supported by Gemini in Spanner.

To improve system performance for Spanner using Gemini Cloud Assist, follow these steps:

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  To open the **Overview** page of an instance, click the instance name.

3.  To open Gemini, click spark **Gemini** .

4.  In the Gemini pane, enter a prompt that describes the information you're interested in. The prompt must request information about system insights—for example:
    
      - "What was the peak database CPU utilization in the last 24 hours?"
      - "What is the peak transaction latency for this database Instance around 2pm today?"
      - "What is the database load trend for this instance in the last week?"
      - "What is the peak cpu utilization by priority only for system operations for database instance in the last hour?"
      - "What is the p99 request latency for this database Instance in the last 24 hours?"
    
    The duration for the prompt request must be within the past seven days. The duration can be specified in the following ways:
    
      - Append a duration to the prompt. For example, to get information about the past day, add "in the last 24 hours" or "in the last day" to the end of the prompt.
      - In the Google Cloud console by doing the following:
        1.  In Spanner, click **System insights** in the left pane.
        2.  Click **1 hour** , **6 hours** , **1 day** , **7 days** . Click **Custom** to specify a custom duration that's within the past seven days.
      - Don't specify a duration in the prompt or the Google Cloud console to use the default duration of the last one hour.
    
    If the specified duration extends further in the past than the last seven days, then an error is returned.

5.  To send the prompt, click send send **Send** . Gemini returns a response to your prompt.

## What's next

  - [Write SQL with Gemini assistance](/spanner/docs/write-sql-gemini) .
  - Learn about how to [optimize Spanner performance](/spanner/docs/performance) .
  - Learn how to [monitor instances with system insights](/spanner/docs/monitoring-console) .
