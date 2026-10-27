Helps you create Data Lakes, typically using S3 for storage

Exam cue: Lake Formation = centrally manage permissions & access to data in a data lake

Data Lake: A central place to have all your data for analytics purpose

This is a layer on top of Glue, but you don't interact with Glue directly

It gives access control, row and column level security

use-case: Imagine a company stores millions of customer, sales & fin records in S3. Diff teams need access to diff datasets, but the company must prevent unauthorized access to sensitive information

Lake formation helps admin define and enforce fine-grained permissions on the data lake: access to db, tables, olumns, ...

So instead of setting up security in DataSources (S3, RDS, Aurora) or in analytic tools like (Athena, Quicksight), you can handle Access Control from AWS Lake Formation

It injects all the data, and stores the Data Lake in S3 which can manage security and any other thing we want


# Apache Flink (a support for apache Flink in AWS)

A framework designed to process streaming data in real-time.

It can analyse events as they arrive rather than waiting for a large batch of data to accumulate

By this, you can run any Apache Flink application on a managed cluster on AWS

NB: Flink doesn't read from Amazon Data Firehose, but can read from kinesis data stream (a live feed)

use-case: Imagine a payment company processing thousands of tx every second. It wants to detect suspicious spending patterns immediately.
Flink can process each transaction as it arrives, maintains information about previous tx, and flag unusual activities within seconds