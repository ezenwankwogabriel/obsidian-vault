Kinesis data streams let you collect, retain, and process streaming data yourself

kinesis data firehose delivers streaming data to destinations for you, with minimal operational management

Data stream preserves records for independent consumers and replays them
Firehose delivers them to S3 for durable storage and later analytics

## The scenario that makes the difference obvious

At 10:00 a.m., payments start flowing. At noon, your team develops a new fraud-detection application and wants to process the payment events from 10:00 a.m. onward.

- With Data Streams, the new application can read retained records from the appropriate point in the stream.
    
- With Firehose alone, you cannot simply rewind Firehose and replay its delivery buffer as though it were a retained event stream. If the events were delivered to S3, you could build a separate process to read the stored files, but that is a different workflow.