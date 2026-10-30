A fully managed, serverless NoSQL DB designed for fast, predictable performance at scale. optimized for key-value access

Think when an app needs very low-latency reads and writes, massive scale, high availability, and **minimal db administration**

ex: shopping carts, gaming leaderboards, user sessions, IoT data and other metadata

DynamoDB is a key-value & document database, not a traditional db like PSql / MySQL

Supports millions of requests per second, trillions of rows, 100s of TB of storage
Fast & consistent in performance (single digit millisecond)

It is made of Tables, every table has a primary key. Comes in:
	Simple primary key: partition key only => CustomerID = C001
	Composit primary key: Primary key + sort key => CustomerId & OrderDate
Each table can have an infinite number of items (=rows)
Each item has attributes (can be added over time - can be null)

Max size is 400KB

## DynamoDB Accelerator (DAX)
A fully managed highly available seamless in-memory cache for DynamoDB
Helps solve read congestion by caching. giving you microsecond latency for cached data
compatible with existing dynamodb apis
Default TTL of 5mins

Use this for individual object cache, query & scan cache
Use "ElastiCache" for store aggregation result

## Stream Processing
DynamoDB streams: This captures changes made to items in DynamoDB tables, such as insertion/updates, delete. Other services can react to them
	24hour retention
	Limited # of consumers
	Process using AWS lambda Triggers or Dynamodb stream kinesis adapter

Kinesis Data Streams (newer)
	1 year retention
	High # of consumers
	Process using AWS Lambda, kinesis data analytics, kinesis data firehose, ...

## Global Tables
These are DynamoDB tables that'd be replicated across different regions. There's a two-way replication for the table
This makes it accessible with low latency in multiple regions

To use this first enable DynamoDB streams (as it's the underlying architecture behind)