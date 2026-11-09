Choosing the right database

RDBMS (SQL, OLTP): RDB, Aurora
NoSQL: no joins, no SQL (DynamoDB, ElastiCache, Neptune, DocumentDB,(for MongoDB), keyspaces(for apache cassandra))
Object Store: S3 / Glacier (backup/archive)
Data Warehouse: (Sql analytics/BI): Redshift (Olap), Athena, EMR
Search: OpenSearcH(json)
Graph: Amazon Neptune
Ledger: Amazon Quantum ledger database
Time SEries: Amazon Timestream

## RDS
- Managed PostgreSQL/MySQL/Oracle/SQL Server/DB2/MariaDB/Custom
- Provisioned RDS instance size and EBS Volume Type/size
- Auto Scaling capacity for storage
- Support for Read Replicas and Multi AZ
- SEcurity through IAM, Security Groups, Kms, SSL in transit
- Automated Backup with Point in time restore feature (35 days)
- RDS Custom option for access to & customize underlying instance (Oracle & SQL server)

- usecase: Store relational dataset, perform SQL queries & transactions

# Aurora
Compatible API for Postgres/MySQL, sepration of storage and compute
Storage: data is stored in 6 replicas across 3 AZ (highly available, auto scaling)
Compute: Cluster of DB instance across multiple AZ
Since you have a cluster, you need users to know where to write and read from, hence a "writer" & "reader" Custom Endpoints
- Aurora Serverless: for unpredictable workload, no capacity planning
- Aurora Global: up to 16 DB read instances in each region, <1 sec storage replication
- Aurora ML: perform ML using SageMaker & comprehend on Aurora
- Aurora DB cloning: cloning new cluster from existing one, faster than restoring a snapshot. use-case, if you need a staging db from your prod db

# ElastiCache
A managed Redis/Memcached service
In memory data store, sub millisecond latency
Support for clustering(redis), and multi AZ, read replicas (sharding)

DynamoDB
AWS proprietary tech, managed serverless NoSQL db
provisioned capacity or on-demand capacity modes
Can replace ElastiCache as key/value store
Highly Available, mutli-az by default
DAX cluster for read caching

Neptune
A fully managed graph database
A popular graph dataset  is a social network
	users have friends
	posts have comments
	comments have likes from users
	users share and like posts
highly available across 3 AZ, with up to 15 read replicas
use-cases for neptune streams: 
	send notification when certain changes are made
	maintain 

Keyspaces (for Apache Cassandra)
Apache Cassandra is an open-source NoSQL distributed db

keyspace is a managed Apache Cassandra database service
Is serverless, scalable, highly available, fully managed by AWS
Tables are replicated 3 times across multiple AZ
use-case: store IoT device info, time series data

Timestream
A time series db (a bunch of points that have a time series in it)
use-case: 



---
# dynamoDB
A fully managed NoSQL db that stores application data as items in tables

# dynamoDB Streams
A stream of records represent chagnes made to items in a dynamoDB table. it's responsible for making item-level changes available for downstream processing.
On any db change, DynamoDB streams can emit a record describing the change

# kinesis client library (KCL) - the stream consumer domain
This is a library that helps you build apps that consume and process records from a stream.
It is neither the stream nor the DB.

# dynamoDB as KCL's metadata store
dynamoDB can serve as a store for business data and a store for processing cordination data.
The KCL metadata table is typically a separate DynamoDB table

# dynamoDB streams kinesis adapter
kcl was designed around Amazon kinesis data stream consumer model. This adapter is a bridge allowing KCL-based applications to consume DynamoDB streams records. 
It does not turn dynamoDB streams into a kinesis Data stream nor replace the source table

## What to remember for SAA-C03

| If the question asks…                                                        | Think…                                                                                                                            |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| How can I react to item inserts, updates, and deletes?                       | DynamoDB Streams                                                                                                                  |
| How can a Lambda function process DynamoDB changes without managing workers? | Lambda event source mapping for DynamoDB Streams                                                                                  |
| How can I build a KCL-based consumer for DynamoDB Streams?                   | DynamoDB Streams Kinesis Adapter                                                                                                  |
| How does KCL coordinate workers and track progress?                          | DynamoDB metadata table, leases, and checkpoints                                                                                  |
| How do I retain change records longer than 24 hours?                         | Consider a different streaming/archival design, such as forwarding records to Kinesis Data Streams or another durable destination |

This is a common exam distinction:
- Push model: EventBridge invokes Lambda when an event occurs.
- Poll model: Lambda event source mapping polls a stream or queue and invokes Lambda.