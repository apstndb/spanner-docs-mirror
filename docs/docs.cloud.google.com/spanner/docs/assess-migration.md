Assessing your source database and how its usage maps to Spanner requires evaluating your business, technical, operational, and financial needs. We recommend covering the following key areas for your assessment:

  - **Business goals** : Define the specific business problems Spanner solves, such as scalability, availability, and consistency. Establish measurable success criteria, such as reduced latency, increased transaction volume, and cost reduction.

<!-- end list -->

  - **Cost analysis** : Calculate the potential total cost of using Spanner (compute, storage, and network) and compare it to your current database costs. Factor in one-time migration costs and ongoing operational expenses. For more information, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

<!-- end list -->

  - **Schema compatibility** : Analyze the existing source database schema for possible incompatibilities with Spanner such as data types, constraints, indexes, or stored procedures. Plan for schema modifications and data transformations to appropriately map your source database schema to Spanner. For more information, see [Schema design best practices](/spanner/docs/schema-design) .

  - **Data consistency and transactions** : Understand Spanner's external consistency model and its differences from your source database transaction model. Evaluate the impact on your application logic. For more information, see [Spanner: TrueTime and external consistency](/spanner/docs/true-time-external-consistency) .

  - **Data locality and regional configurations** : Determine optimal Spanner deployment topology such as regional, dual-region, or multi-region deployments based on user locations, latency requirements, and cost considerations. For more information, see [Instances configurations](/spanner/docs/instance-configurations#configuration) .

  - **Application code compatibility** : Inventory all database interactions with your application code. Identify areas that require modification because of differences in SQL dialect, client libraries, and transaction management.

  - **Performance and scalability requirements** : Define current and projected workloads such as read and write ratios, transaction rates, and data volume. Determine acceptable latency and throughput. For more information on Spanner's performance, see [Performance overview](/spanner/docs/performance#typical-workloads) .

  - **Migration strategy and downtime** : Develop a detailed migration plan, including data extraction, transformation, loading, and validation. If downtime isn't a concern, you can perform a one-time bulk load and cutover. Otherwise, consider minimizing downtime. Define a rollback plan.

  - **Operational consideration** : Plan for changes in database administration, monitoring, and disaster recovery. Assess the learning curve for the team. Integrate Spanner with existing operational tools and processes For more information, see [Disaster recovery overview](/spanner/docs/backup/disaster-recovery-overview) .

  - **Security** : Review Spanner's security features such as [authentication](/spanner/docs/authentication) , [authorization](/spanner/docs/iam) , and [encryption](/spanner/docs/encryption-in-transit) . Ensure compliance with relevant regulations.

## Source specific guides

  - MySQL: [Migrate from MySQL to Spanner](/spanner/docs/migrating-mysql-to-spanner) .
