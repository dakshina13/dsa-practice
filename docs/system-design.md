## DynamoDB
### Features
- Flexible Data Model
- Indexing and Querying: Primary key (Partition and Sort Keys)
- Global Secondary Indexes (GSI): Allows you to query by attributes other than your primary partition key.
- Local Secondary Indexes (LSI): Enables sorting by attributes other than the primary sort key.
- Supports Atomic Transactions
- Scaling and Availability: It utilizes consistent hashing for data partitioning across physical nodes and supports asynchronous replication across different availability zones.
- Consistency Options: While eventual consistency is the default, DynamoDB allows users to enable strongly consistent reads when data accuracy is critical.
- DynamoDB Accelerator (DAX): A built-in, in-memory caching layer that provides microsecond response times for heavy read workloads.
- DynamoDB Streams: A feature for Change Data Capture (CDC) that records modifications to the table (inserts, updates, deletes), allowing you to trigger downstream processes like syncing data with other databases (e.g., Elasticsearch).

### When not to use
- Complex Query Patterns: If your application relies heavily on complex joins across many tables or requires extensive subqueries, DynamoDB may not be the optimal fit.
- Transactions Across Multiple Tables: While DynamoDB supports transactions, there is a limit on the number of items that can be included in a single atomic operation.
- Overly Complex Data Modeling: If you find yourself needing to create a high volume of Global Secondary Indexes (GSI) or Local Secondary Indexes (LSI) to support your query patterns, it may be an indication that DynamoDB is being used inappropriately for your data structure.