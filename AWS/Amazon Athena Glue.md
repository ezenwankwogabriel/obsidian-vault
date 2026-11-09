AWS Glue is a serverless data integration service that discovers, prepares, transforms, and moves data so it can be used for analytics.

*data integration and ETL (Extract, Transform, Load)*

The easiest way to understand Glue is to imagine a company whose data is scattered across different places and stored in different formats.

For example, your payment platform stores:
- Payment transactions in Amazon S3 as JSON files.
- Customer records in an RDS PostgreSQL database.
- Application logs in other S3 folders.
- Daily reports in CSV format.
You want to analyze all this data using Amazon Athena or Amazon Redshift (a data warehouse). The problem is that the data is scattered, its structure may be inconsistent, and the analytics service needs to know how to interpret it.

Glue helps prepare and organize that data for analysis.

### Parts of Glue
1\. Glue Data Catalog –– the data dictionary
Stores metadata about your data: table names, column names, data types, partitions, and locations
doens't store actual records, Your file remains in S3, or your data remains in its original database

2\.  Glue Crawler –– the data inspector
Scans supported data sources, infers their schemes and can create/update table definitions in the Data Catalog

3\. Glue ETL Job –– the data worker 
Reads source data, applies transformation, and writes the result to a destination.
Jobs can be authored visually or with code and run on managed processing engines as Apache Spark

## things to know
> Glue Job Bookmarks: prevent re-processing old data
> Glue DataBrew: clean and normalize data using pre-built transformation
> Glue Studio: new GUI to create, run and monitor ETL jobs in Glue
> Glue streaming ETL: runs ETL jobs as streaming jobs instead of batch jobs

AWS Glue can read directly from S3 and other data sources. Doesn't require cloudwatch to trigger it.

use-case:
Imagine you're building an analytic platform for e-commerce
You data comes from three places

- Customer orders in RDS PostgreSQL
- Application logs in S3
- Customer activity stored in DynamoDB
You intend to combine these sources, clean the data and produce reports using Amazon Athena

```
       DATA SOURCES
       ────────────
       RDS PostgreSQL
       Amazon S3
       DynamoDB
            |
            v
	    AWS Glue
       ─────────
       Extract data
       Clean data
       Transform data
       Join datasets
            |
            v
       Amazon S3
       (Processed data)
            |
            v
       Amazon Athena
            |
            v
       Reports / Analytics
```

For example, Glue could:

1. Read orders from PostgreSQL.
    
2. Read customer activity records from S3.
    
3. Join orders with customer activity.
    
4. Remove duplicate records.
    
5. Convert the output to Parquet format.
    
6. Write the processed data into an S3 data lake.
    

Athena can then query the resulting Parquet files using SQL.For example, Glue could:
1. Read orders from PostgreSQL.
2. Read customer activity records from S3.
3. Join orders with customer activity.
4. Remove duplicate records.
5. Convert the output to Parquet format.
6. Write the processed data into an S3 data lake.
Athena can then query the resulting Parquet files using SQL.

Glue can read directly from S3, RDS, Redshift, DynamoDB, Aurora, On-premise (postgres & sql), mongodb, kafka