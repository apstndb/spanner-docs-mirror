> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document explains how to manually trigger a major compaction in your Spanner Omni database. This process is mostly identical to [triggering a major compaction in a Spanner database](https://docs.cloud.google.com/spanner/docs/manual-data-compaction) , with the following distinction:

  - To trigger a major compaction for a Spanner Omni database, use the following command in the Spanner Omni CLI:
    
        spanner admin alpha compact database DATABASE_ID
    
    Replace DATABASE\_ID with the database identifier—for example, `MY_DATABASE` .
