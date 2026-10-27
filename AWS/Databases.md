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
