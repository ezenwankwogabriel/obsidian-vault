# SQS (Simple Queue Service)

Produces –– Send Messages ––> SQS Queue –– Pull messages ––> Consumers

It is used to decouple applications.

From the concept of queuing, we have producers, sending messages to a queue, and consumers that listen for new messages, pulling on the queue (sqs)

In AWS, we have *Auto Scaling Group (ASG)*
This is done automatically for SQS

Cause the consumers would be running on ec2 instances likely, would need some way to auto scale depending on need

This is done according to a metric from CloudWatch called *Queue Length*.
Queue Length is the approximate number of messages in the queue.

Set up an alarm for the queue length, and once it reaches or exceeds a particular length, you want to trigger increase/decrease the capacity of the scaling group

# Security
1\. Encryption
- In-flight encryption using HTTPS API
- At rest encryption using KMS keys
- Client-side encrpytion if the client wants to perform encryption/decryption itself
2\. Access Control: IAM policies to regulate access to SQS api
3\. SQS access policies (similar to s3 bucket policies)
- for cross-account access to sqs queues
- for allowing other services (sns, s3, ...) write to an sqs queue

# Message visibility Timeout

Once a message is pulled by a consumer, it becomes immediately invisible to other consumers

It is invisible for a "message visibility timeout" of 30 seconds This means that during the 30 seconds, the message has to be processed

After the timeout, and the message hasn't been deleted by the consumer, the message is put back into the queue, so another consumer would see this message

Hence, it would be received twice by a consumer. If the consumer knows it'd take more than 30 seconds to completely consume the read message, it calls "*ChangeMessageVisibility*" API to increase the timeout (hours, seconds, ...) so it is invisible for the set time (this is just to prevent the consumer from reading same message twice)

# Long Polling
Waiting for a message to arrive if nothing is in the poll at the moment

With LongPolling, consumers decrease the number of API calls made to SQS while increasing the efficiency and reducing latency of your app

The time can be set between 1-20 seconds preferably. prefer this to short polling

Can be enabled using the *WaitTimeSeconds* API

# FIFO Queues
This is the ordering of messages in the queue(first in first out)

- There's limited throughput: 300 msg/s without batching, 3000 msg/s with batching
- Can do "exactly once" send (removing duplicates using Deduplication ID)
- Ordering is by "Message Group ID"

# Auto Scaling Group
Look at the Queue Length, set an alarm, this alarm using cloudwatch alarm to triggers a scaling action (scale up / down)

If the load is too big, some transactions may be lost on inserting to the db (thousands or millions of requests to the db at same time). 

To solve this, we use SQS as a buffer to database writes. Instead of writing to d database, it writes to the SQS Queue (EnQueue) (we don't loose any, all durably stored)
Then we Dequeue the messages and write to the db. only when a message is inserted to the db, do we delete it from the SQS queue.

This works for async requests majorly (this improves decoupling and reliability)

# SNS (notification)
What if you want to send "one" message to "many" receivers

example: a buying service, notifies (email, fraud, shipping, sqs queue)

A Pub / Sub. creating an SNS Topic

- The event producer only sends message to one SNS topic
- As many "event receivers" (subscriptions) as we want to listen to the SNS topic notifications
- Each subscribefr to the topic gets the message
- Up to 12million plus subs per topic
- Can create up to 100,000 topics

Amazon SNS supports email as one of its subscription protocols
For example:
1. CloudWatch Alarm detects that RDS CPU utilization exceeds 85%.
2. CloudWatch publishes an alarm notification to an SNS topic.
3. SNS sends an email to the subscribed Operations team.

# SNS + SQS: Fan Out
- Push once in SNS, receive in all SQS queues that are subscribers
- Fully decoupled, no data loss
- SQS allows for: data persistence, delayed processing and retries of work
- Ability to add more SQS subscribers over time
- Ensure SQS queue access policy allows SNS to write


---
## SQS: Think of a waiting line


### What happens when an order arrives?

Suppose the web tier receives order 101.

1. The web application sends order 101 to SQS.
2. SQS stores the message durably.
3. An EC2 worker polls SQS and receives order 101.
4. SQS temporarily hides that message from other consumers.
5. The worker processes the payment.
6. After successful processing, the worker deletes the message from SQS.

The key is step 6.

Receiving a message does not delete it. The consumer must explicitly delete it after successful processing.

### What if the EC2 instance crashes?

Suppose EC2 Worker 1 receives order 101 but crashes before completing the payment.

The message isn't deleted.

After its visibility timeout expires, SQS makes the message available again.

Another healthy worker can receive and process it.

This is why SQS protects against losing messages when a worker fails


# SNS: Think of broadcasting a notification

SNS (Simple Notification Service) is fundamentally different.

SNS is a publish/subscribe messaging service.

Instead of keeping messages in a waiting line for workers to retrieve, SNS distributes published messages to its subscribers.

### Concrete example: Order notification

Imagine a customer places an order.

Your application needs to notify three different systems:

1. Payment service.
2. Inventory service.
3. Email notification service.

You publish one event to SNS:

Each subscriber receives its own copy of the event.

For example:

- Payment service processes the payment.
- Inventory service reduces stock.
- Email service sends the confirmation.

The services don't need to know about each other.

### The key difference

With SQS, consumers pull messages from a queue.

With SNS, SNS pushes messages to subscribers.

---


# Kinesis Data Streams (real-time data stream)
A service to collect and store streaming data in "real-time" 
	analytics 
	can pass to SNS+SQS 
	and can be independent (directly connecting to consumers))
	allows replaying data

This is because we want to have consumers in real time to read from the kinesis data stream

- Data retained up to 365 days (can replay data by consumers)
- data can't be deleted from kinesis until expiry
- data up to 10mib
- Use "Kinesis Producer Library (KPL)" to write an optimized producer app
- Use "Kinesis Client Libary (KCL)" to write an optimized consumer app

Modes 
1\. Provisioned mode
	Choose  number of shards. 
		each having 1MB/S in (1000 records per seconds)
		and 2mb/s out
2\. On-Demand mode
- no need to provision / manage the capacity
- Default capacity provisioned (4 mb/s in our 4000 records per second)

# Amazon Data Firehose
This is a service to send data from sources into target destinations

Records up to 1 MB, can optionally be transformed using Lambda
This can be transformed/batched into S3, Redshift, Opensearch

This is a "Near Real Time" service with buffering capability based on size/time

Supports CSV, JSON, Parquet, Avro, Raw Text, Binary data


KDS is a streaming data collection service, youd write your producer & consumer code, it is real time, and has provisioned and on-demaind mode. Data can be stored up to one year, and you have replay capability

ADF is a way to load streaming data into other services S3/Redshift/Opensearch/3rd party.. Is fully managed, and near real-time, no data storage

# Amazon MQ
When migrating existing apps (on MQTT, AMQP, ...)to the cloud , you use Amazon MQ
This is a managed message broker service for RabbitMQ and ActiveMQ

This doesn't scale as much as SQS/SNS
Runs on servers, can run in Multi-AZ with failover
Has both Queue and Topic features



