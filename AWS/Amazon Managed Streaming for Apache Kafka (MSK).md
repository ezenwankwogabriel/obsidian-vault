A managed service that lets you build applications using Apache Kafka to stream events and data btw systems in real time

Imagine an e-commerce application generates events whenever someone places an order, makes a payment, or updates inventory.

Instead of connecting every service directly to every other service, the application publishes events to Kafka. Multiple consumers can independently read those events.

MSK allows you to create, delete and update clusters
Also it creates & manages kafka brokers nodes and Zookeepers node for you

There's an option for MSK serverless

# Kafka and SNS
SNS: primarily a message distribution service: publish a message, and SNS delivers to subscribers

Kafka is a distributed event log: producers write events to a retained stream, and consumers can read those events independently, keep track of their progress and replay them later

Both can distribute events to multiple servies. Diff is what happens to those events after they're published, and how consumers process them.

use-case:
Imagine your fintech app processes a payment. When payment succeeds, backend emits event `PaymentCompleted(P123, #50k)`
Three services react to this (fraud detection, analytics, notification)

With SNS, you can publish the event to a topic and subscibe all three services. So why kafka?

Because imagine analytics service goes offline for 6 hours, fraude detection needs a replay of yesterday's transactions. Kafka's retained event log and consumer offsets become very useful.


*crucial diff*: kafka retains the event stream independently of the consumers' progress. Consumers can replay events while those events remain available.
SNS is designed around delivering messages to subscribers, and itself doesn't retain event logs.


### kinesis data stream vs amazon msk
kafka and kinesis data stream solves a similar problem. collecting, retaining, and distributing streams of events for real-time processing

Main diff is that kafka is open-source that AWS offers as a managed service through Amazon MSK, while Kinesis Data Stream (kds) is AWS's own managed streaming service

kds has 1mb message size limit. same as msk but can configure higher (ex: 10mb)
kds has data streams with shards, kafka topics with partitions
scaling: shard spliting and merging for kds. plaintext / tls in-flight encryption for msk
kms encryption at rest for kds. same as msk
msk can keep data for as long as you want but kds is 1 year