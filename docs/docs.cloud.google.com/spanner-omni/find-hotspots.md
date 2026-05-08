> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes how to diagnose hotspots, or queries with high CPU usage, in your Spanner Omni databases. The process for diagnosing hotspots is mostly identical to [Spanner hotspot diagnosis](https://docs.cloud.google.com/spanner/docs/find-hotspots-in-database) , with the following distinctions:

  - The Spanner Omni system insights dashboard is available in Grafana. For more information on how to set up this dashboard, see [set up Grafana](https://docs.cloud.google.com/spanner-omni/deploy-on-vms#setup-grafana) .

  - Instead of going to the Google Cloud console to query `SPANNER_SYS.*` tables, you query your database directly using the Spanner Omni CLI. For example, the following query shows the data splits with the highest CPU usage over the past 5 hours in the database, and ranks them by CPU usage:
    
        spanner sql \
        --database DATABASE_ID --execute \
        "SELECT * \
        FROM SPANNER_SYS.SPLIT_STATS_TOP_MINUTE \
        WHERE INTERVAL_END > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), \
        INTERVAL 5 HOUR) \
        ORDER BY CPU_USAGE_SCORE DESC"
    
    Replace DATABASE\_ID with the database identifier—for example, `MY_DATABASE` .

## Known issues

  - If the total compute capacity of your Spanner Omni deployment is less than 3.5 vCPU per server, the split CPU usage scores in the `SPANNER_SYS.*` tables might be lower than the correct scores on the system insights dashboard. The dashboard shows the correct split CPU usage scores, but you need to query the `SPANNER_SYS.*` tables to identify the splits that correspond to those scores.
