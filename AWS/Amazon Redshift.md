A fully managed cloud data warehouse designed for large-scale analytics using SQL

Concept
- RDS runs d database behind your app
- DynamoDB handles high-scale NoSQL access patterns
- Athena queries data sitting in S3 without managing a warehouse
- Redshift analyzes large volumes of data to help answer business operations

### Data warehouse

A transactional database might answer:
> What is the status of order 123 right now?

A data warehouse might answer:
> What were our top-selling products in each region over the last five years, and how did sales change quarter by quarter?

### OLTP - Online Transaction Processing
Examples: placing orders, updating user profilings, transferring money

### OLAP - Online Analytical Processing
Examples: calculating monthly revenue, ...

Redshift is used for OLAP and not for OLTP as we can see from above as it is a data warehouse

It gives you 10x better performance than any other warehouse, and scales to Petadatas of data

Uses Columnar storage of data & parallel query engine

It has two modes: Provisioned cluster / Serverless Cluster

BI tools such as Amazon Quicksight or Tableau integrate with this

vs Athena: is faster queries/joins/aggregates thanks to indexes. whereas athena is serverless and has all your data in S3 (can make queries directly on S3)

Nodes
These are the compute machines that store and process your data warehouse workloads. They work to execute SQL queries over large datasets
Think of Redshift as a team working on a huge spreadsheet: multiple nodes can process diff portions of data in parallel instead of one
![[Pasted image 20261027135541.png]]
1\. Leader node: used for query planning, result aggregation [plan]
2\. Compute node: for performing the queries, send results to leader [execute

Provisioned vs Serverless: In Provisioned, you choose and manage cluster compute capacity and node configuration. In serveless, AWS manages the underlying compute provisioning.

### Redshift Snapshots & DR (data recovery)
snapshots are point-in-time backups of a Redshift data warehouse cluster, stored internally in S3 that you can use to restore data and recreate a cluster

Snapshots are incremental, (only what has changed is saved)

## Automated vs manual snapshots

| Feature                         | Automated                    | Manual                                      |
| ------------------------------- | ---------------------------- | ------------------------------------------- |
| Created by                      | AWS                          | You or your automation                      |
| Purpose                         | Routine recovery             | Important milestones or long-term retention |
| Default retention               | 1 day                        | Indefinite                                  |
| Deleted when cluster is deleted | Yes, for automated snapshots | No, normally retained                       |
| Can control retention?          | Yes                          | Yes                                         |
Exam trap: If you want to preserve a particular automated snapshot beyond its normal retention period or cluster lifecycle, copy it to a manual snapshot.
### Loading data into Redshift [large inserts are preferred]
- Amazon Kinesis Data Firehose
- S3 using COPY command
- EC2 Instance JDBC driver

## Undersanding what Redshift does
Imagine running a payment platform. Tx history is stored as files in S3, we use Redshift to analyze revenue, payment failures, and customer trends

Two operations matter:
1\. COPY: Load data from S3 to Redshift
2\. UNLOAD: Export data from Redshift to S3

The question is: "what network path does that data take"

Without Enhanced VPC Routing enabled in Redshift, COPY and UPLOAD traffic doesn't follow your VPC route tables/security groups/network ACLs in same way traffic routed through your VPC would. 
Enhanced VPC routing forces COPY and UPLOAD traffic through the VPC. You still need to configure the necessary routes and connectivity though to prevent these from going through the internet
### Redshift Spectrum
Allows you run SQL queries against data stored directly in Amazon S3, without first loading that data to Redshift tables.

### Data Lake vs Data Warehouse
The fundamental difference:

- Data Lake: Stores large volumes of raw data in its original format, whether structured, semi-structured or unstructured.
- Data Warehouse: Stores processed, structured and organized data optimized for analytics and reporting.