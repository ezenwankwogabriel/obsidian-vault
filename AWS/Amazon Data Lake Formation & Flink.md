Helps you create Data Lakes, typically using S3 for storage

Exam cue: Lake Formation = centrally manage permissions & access to data in a data lake

Data Lake: A central place to have all your data for analytics purpose in its original / transformed form.

Amazon S3 is commonly used as the storage foundation for a data lake

This is a layer on top of Glue, but you don't interact with Glue directly

It gives access control, row and column level security

use-case: Imagine a company stores millions of customer, sales & fin records in S3. Diff teams need access to diff datasets, but the company must prevent unauthorized access to sensitive information

Lake formation helps admin define and enforce fine-grained permissions on the data lake: access to db, tables, olumns, ...

So instead of setting up security in DataSources (S3, RDS, Aurora) or in analytic tools like (Athena, Quicksight), you can handle Access Control from AWS Lake Formation

It injects all the data, and stores the Data Lake in S3 which can manage security and any other thing we want

Think of the services this way:

|Service|Main job|
|---|---|
|Amazon S3|Stores the data|
|AWS Lake Formation|Manages data lake setup and fine-grained permissions|
|AWS Glue Data Catalog|Stores metadata describing datasets, tables, and schemas|
|Amazon Athena|Queries data in S3 using SQL|
|AWS Glue ETL|Extracts, transforms, and loads data|

# Apache Flink (a support for apache Flink in AWS)

A framework designed to process streaming data in real-time.

It can analyse events as they arrive rather than waiting for a large batch of data to accumulate

It allows you build applications that continuously process incoming data rather than waiting for data to accumulate in a database or data warehouse

Previously called Amazon kinesis Data Analytics for Apache Flink, now Amazon Managed Service For Apache Flink

By this, you can run any Apache Flink application on a managed cluster on AWS

use-case:
Imagine an e-commerce platform receiving thousands of transactions every second.

You want to:
- Detect suspicious transactions immediately.
- Calculate sales totals every minute.
- Monitor transaction volumes.
- Identify unusual purchasing patterns.

Traditional batch processing might collect data and process it every hour.

Flink processes events continuously as they arrive.

NB: Flink doesn't read from Amazon Data Firehose, but can read from kinesis data stream (a live feed)

##### Flink vs Kinesis Data Streams vs Firehose

| Service                          | Primary responsibility                                                      |
| -------------------------------- | --------------------------------------------------------------------------- |
| Kinesis Data Streams             | Collects and stores streaming records for consumers                         |
| Managed Service for Apache Flink | Processes and analyzes streaming data                                       |
| Kinesis Data Firehose            | Delivers streaming data to destinations such as S3, Redshift and OpenSearch |
|                                  |                                                                             |
```
Firehose can perform similar transformations Flink does using Lambda. and supports format conversion

Flink is designed for more sophisticated, continous processing, such as stateful computation, ...
```

##### Flink vs AWS Glue
Another possible exam comparison:

|Feature|Apache Flink|AWS Glue|
|---|---|---|
|Processing model|Continuous stream processing|Primarily batch ETL; also supports streaming ETL|
|Latency|Low-latency continuous processing|Depends on job type and execution|
|Main use|Real-time analytics and event processing|Data integration and transformation|
|Typical input|Kinesis, Kafka|S3, databases, streaming sources|
|Example|Detect fraud as transactions happen|Transform daily sales data into Parquet|

- Flink is for stream processing, not simply storing streaming data.
- It can consume from Kinesis Data Streams and Apache Kafka.
- It supports stateful processing and event-time windows.
- Checkpoints help recover application state after failures.
- It can send processed data to downstream services.
- Managed Service for Apache Flink is managed by AWS, so you don't have to provision and operate the Flink cluster yourself.