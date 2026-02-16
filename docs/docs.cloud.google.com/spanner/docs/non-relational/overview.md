Spanner is a highly reliable, fully managed, database system. While Spanner has [evolved](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/46103.pdf) to become a relational database management system, it has its [roots](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf) as a non-relational, key-value storage system, and retains the fundamental characteristics of such a system. As such, you can use Spanner as a non-relational (NoSQL) database and migrate from other non-relational databases to Spanner. This document helps you understand whether Spanner is the right fit for your non-relational workloads.

NoSQL databases came into use during a time when conventional relational databases lacked features to support emerging applications that required high scalability, availability, and compute elasticity. They did so by sacrificing several features often critical for data management such as transactions, consistency, and ad-hoc queries. Spanner was built to support both the demanding nature of highly available applications and functionalities provided by conventional relational databases, so that customers can take advantage of both sets of features.

With Spanner, you can start off with straightforward, non-relational storage needs and scale your application as needed.

## How Spanner meets NoSQL database criteria

Spanner meets the following key criteria for your NoSQL workloads.

### Scale and performance

NoSQL databases gained popularity due to their ability to scale reads and writes horizontally. With Spanner, you don't need to worry about scale or performance being a concern. Key-value styled Spanner databases can scale horizontally to support hundreds of millions of read or write requests per second and petabytes of data. Spanner's compute capacity scales with the workload, maintaining a consistent, low latency profile even as your application scales by several orders of magnitude.

### NoSQL API

Conventional relational databases are typically accessed using SQL, which comes with a learning curve for developers unfamiliar with relational databases. Clients for these databases also typically rely on persistent connections, and require connection pooling infrastructure to be deployed in order to scale. In contrast, the Spanner API is built on top of a gRPC/HTTP2 request and response model that automatically handles connection failures. Spanner provides straightforward, language-native, and efficient NoSQL read-write APIs which don't require SQL knowledge. In addition, Spanner clients don't require any connection pooling in order to scale.

### Fully managed

A big draw of NoSQL databases has been that they are perceived to be easier to manage. As a fully managed service, Spanner doesn't place any operational burden on customers. Spanner performs zero-downtime software and hardware updates behind the scenes while maintaining backward compatibility. The Spanner API and semantics are the same as if the operations were being performed on a single machine database and don't require knowledge of Spanner's internal architecture. Spanner runs on deployments ranging from 1/10th of a node to tens of thousands of nodes, scaling automatically and responsively with a [managed autoscaler](/spanner/docs/managed-autoscaler) .

### Semi-structured data

Spanner supports flexible data types like JSON and BYTES that are used to store semi-structured or unstructured data. Like other NoSQL databases, you can use these data types to avoid specifying all your storage schema upfront.

### Access control

Like other NoSQL databases, Spanner supports [IAM based access control](/spanner/docs/iam) . Administrators can configure and administer access control policies without storing usernames and passwords in the database.

## How Spanner differs from conventional NoSQL databases

Spanner provides the following advantages over conventional NoSQL databases.

### Transactions

As applications grow in complexity, they often need to perform multi-row and multi-table transactional operations on the database. With Spanner, you don't need to migrate to a transactional datastore as your database grows because Spanner has full support for read-write transactions. As an ACID compliant database, Spanner maintains transactional consistency of your database at all times, regardless of scale.

### Data modeling

Schema design in NoSQL databases can be unnatural due to the need to fit all data into one table, and to forcibly denormalize data due to the inability to perform joins. With Spanner, you can specify your schema without resorting to a single table or denormalization. To optimize access patterns that touch multiple tables, you can use [table interleaving](/spanner/docs/schema-and-data-model#create-interleaved-tables) . You can also perform joins across tables.

### Ad hoc queries

Even if you mainly use the NoSQL API, it is often still useful to run ad hoc queries for debugging or analytics purposes. Spanner adheres to the SQL standard query language. You can use [request prioritization](/spanner/docs/reference/rest/v1/RequestOptions) to isolate ad hoc low-priority traffic from online traffic or use Spanner [Data Boost](/spanner/docs/databoost/databoost-overview) to run analytical queries on compute resources completely isolated from online database traffic.

### Strongly consistent secondary indexes

Applications often require secondary indexes to support low latency lookups. Typical NoSQL databases offer eventually consistent secondary indexes or place constraints on how large an index can grow. This can complicate application logic which needs to work around these constraints. Spanner offers strongly consistent secondary indexes at scale without any size constraints. This lets you focus on your application logic and not worry about consistency issues.

## Is Spanner right for your NoSQL database needs?

Spanner is a flexible data storage system that supports both SQL and NoSQL use cases. While some applications start off with straightforward storage needs, as they grow in complexity, they need more features typically not offered by NoSQL databases, such as transactions, consistent secondary indexes, and a flexible query language. With Spanner, you aren't constrained by these limitations and can grow your application as needed.

Most non-relational workloads are a great fit for Spanner. Not only does Spanner offer a straightforward NoSQL read-write API, but it's also backed by high availability, high reliability, low latency, compute elasticity, and extreme scalability. Spanner lets you consolidate diverse workloads onto a single flexible platform.
