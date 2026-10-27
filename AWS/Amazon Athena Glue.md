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