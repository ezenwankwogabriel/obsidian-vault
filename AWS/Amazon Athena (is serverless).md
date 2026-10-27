This is a serverless, interactive query service for analysing data stored in Amazon S3 using SQL

Key: If you data is already in S3 and you wish to query it without setting up a db/managing servers, think "ATHENA"

You load data to S3, then use Athena to query/Analyze the data directly on the dbwithout moving it

It supports CSV, JSON, ORC, Avro and Parquet

Commonly used with Amazon Quicksight for reporting/dashboards. from the name sight/insight –– dashboard/reporting

```
quicksight is a business intelligence (BI) service. turns data stored in db/warehouse/files into interactive dashboards, charts, ... business insights
```

use-cases: biz intelligence / analytics / reporting for any type of logs from your aws service...

## performance improvements using Athena
A\. Use columna formats (less scan)
Store analytical data in formats such as Apache Parquet or ORC, rather than relying on large CSV/JSON files

If a dataset has 50 columns but your query needs only 3, a columnar format lets Athena read the relevant columns rather than the entire row-oriented dataset

B\. Partition the data
Organize files into folders by useful attributes, such as year, month, or region

Example: s3://analytics/download/year=2026/month=10/

A query filtering on the partition keys can skip unrelated folders

C\. Filter your queries
Select only columns & rows you need. A partifion filter can avoid scanning other partitions, while columnar fomarts and predicate pushdown can reduce the data reads

Example: merely adding a WHERE clause in your query dramatically reduces the scanned bytes

Exam clue: If the question says Athena query costs are too high because too much S3 data is scanned, look for partitioning, Parquet/ORC, and query optimization.

**One more thing, "use larger files (> 128 MB) to minimize overhead". If you had many many small files, athena isn't going to perform best (maybe use glue to gather the resources before passing to athena)**

### Athena Federated Queries
Lets you use SQL in Athena to query data stored in different stores, including sources outside Amazon S3, without first moving all the data to a data warehouse

Use Data Source connectors that run on AWS lambda to run Federated Queries (cloudwatch logs, dynamodb, rds)

### AWS Glue Data Catalog
Your S3 bucket might contain thousands of files, Athena needs to know how to interpret those files as tables

There goes Glue Data Catalog, which stores metadata such as
- Table names & column names
- Data Types
- S3 locations
- Partition information
- Structure needed to interpret the underlying data
For example, your S3 bucket contains CSV files with columns `country`, `image_id`, and `download_time`. You define a table in the Glue Data Catalog so Athena knows how to query those files.

NB: the catalog contains metadata; it doesn't normally contain copies of your actual S3 records

