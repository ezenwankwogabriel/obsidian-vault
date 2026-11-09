Relational Database Service (SQL)
Postgres, MySQL, MariaDB, Oracle, Microsoft SQL server, IBM DB2, Aurora (AWS Proprietary DB)

Why RDS vs DB on EC2
- Automated provisioning, OS patching
- Continuous backup and restore to specific timestamp (Point in Time Restore)
- Monitoring dashboard
- Read replicas for improved read performance
- Multi AZ setup for Disaster Recovery (DR)
- Scaling. Backed by EBS
- But *You cannot SSH into the instance*

## Read Replicas (async replication (eventual consistency))
These help you scale your reads. Can create up to 15 Read replicas
within same AZ, Cross AZ, and Cross Region
Replication is ASYNC (so reads are eventually consistent)

There's a network cost in AWS when data goes from one AZ to another
For RDB Read Replicas within same region (different AZ), you don't pay that fee

RDS Multi AZ (Disaster Recovery) (sync replication)
This keeps a standby db in another AZ so your db can recover from a failure
Picture a Primary db (main worker handling db operations) and a standby db (your backup worker, ready to take over if primary fails)
There's synchronous replication between the two, the secondary doesn't serve application queries

*Can you setup Read replica as a multi AZ?* **Yes**.

### How to make a db go from single to multi az
this is a zero downtime operation
just click on "modify" for the db: 
	a snapshot is taken
	a new DB is restored from the snapshot to a new AZ
	sync is established between the db

## RDS Custom
A managed db service giving you more control over the underlying db env, including the OS and db config.

Think this a middle ground for RDS and managing db on EC2

The custom gives you access to the underlying db and OS for:
	configuring settings
	install patches
	enable native features
	access ec2 instance using ssh / ssm session manager

To perform customization, de-activate Automatic Mode (better to take a snapshot for easy restoration)

```

Standard Amazon RDS
AWS manages most infrastructure
- AWS manages the OS.
- AWS manages database installation and routine maintenance.
- Limited access to underlying environment.


Amazon RDS Custom
More customization
- Access to the underlying EC2 instance and OS.
- Can modify database and OS configurations.
- AWS still provides managed database capabilities.


Database on EC2
Maximum control
- You manage the OS.
- You install and configure the database.
- You manage backups, patching, recovery and maintenance.
```

### RDS vs RDS Custom
RDS: entire db and OS is managed by AWS
RDS custom: full admin access to the underlying OS and db

## Amazon Aurora
	A proprietary tech from AWS (not open source)
	supports postgres and mysql
	it claims 5x performance improvement over MysQL on RDS, over 3x performance of Postgres on RDS
	Storage grows from 10GB up to 256TB automatically
	Can have up to 15 replicas & the replication process is faster than MySQL
	Failover in Aurora is instantaneous. 

High Availability and Read Scaling
	Aurora stores 6 copies of your data across 3 AZ
		4 copies out of 6 needed for writes
		3 copies out of 6 needed for reads
		relies on 100s of volumes
		Gives 1 master (write) and up 10 15 read replicas

#### Aurora keeps your db highly available within same Region (diff AZ)
#### Aurora Global DB:
keeps your db available across multiple region, especially for recovery and global distributed low-latency reads. 

Think of it as one primary database in one Region, with secondary read-only database clusters in other Regions.
#### Multi AZ protects against instance or AZ failures within a Region

Aurora DB Cluster
	writes only to master (single master)
	can have auto scaling on the read replicas. as we have many read replicas, it'd be hard to keep track of the read urls. for this, we have a "*Reader Endpoint*"

	Reader Enpoint is a connection load balancing and it connects auto to all read replicas. so with this single endpoint, we have a distributed connection to all read replicas

To restore MYSQL Aurora cluster from S3
	Create backup of your on-premises db using Percona XtraBackup
	Store the backup file on Amazon S3
	Restore the backup file onto a new Aurora cluster running MySQL

Aurara Db Cloning: allows you clone a db from an existing one. This is faster than snapshot & restore.
It uses same data volume as the original DB cluster
When updates are made to the new DB cluster, additional storage is allocated and data is copied to be separated

## RDS & Aurora Security
	At rest encryption:
		Encryption of db master & replication using aws kms must be defined at launch time
		If master isn't encrypted, the read replicas cannot be enrypted 
		To encrypt an existing un-encrypted db, take a snapshot, and restore as an encrypted db
	In-flight encryption:
		Clients must use the AWS TLS root certificates for in-flight encryption (encryption in transit)
	IAM Authentication: IAM auth lets an app connect to a supported RDS database using temporary IAM credentials instead of a permanent db password. Auth Tokens are valid for 15 mins. The token is used to authenticate a new connection; an established connection is open beyond the tokens expiry
	Security Groups: Control network access to your RDS/Aurora DB (allow/block specific ip/groups)
	No SSH except for RDS Custom
	Can enable audit log, and sent to CloudWatch logs for longer retention

## RDS Proxy 
This is a managed db connection pooler that sits btw your app and db. It helps applications handle database connections more efficiently during db failovers.

Imagine you have a serverless application using AWS Lambda. Every time traffic increases, Lambda can scale out and invoke many functions concurrently.

*key concept*: connection multiplexing
Suppose 1,000 Lambda invocations connect to your application at once.
The proxy may serve many client connections using a smaller pool of database connections, provided those connections can safely be reused. The exact number depends on workload, configuration, and connection behaviour.

Without a proxy:
- Each function may open its own database connection.
- Connections can accumulate and exhaust the database's connection limit.
- During a database failover, clients may experience connection errors while reconnecting.
With RDS Proxy:
- Connection pooling: it maintains a pool of database connections and reuses them across application requests.
- Better scalability: it reduces the number of connections that need to reach the database directly.
- Faster recovery: it can preserve application-facing connections and route connections to a healthy database instance during supported failovers.
- Managed authentication and credentials: it integrates with AWS Secrets Manager and IAM database authentication where supported.
Allows app to pool and share DB connections established with the db and not having to create one for every app. 
Minimizes open connections (and timeouts), and improves db efficiency by reducing stress on db resource (cpu/ram)
This is fully serverless, autoscaling, highly available (multi-az)
Reduced RDS & Aurora failover time by 66%
If you think of a way to enforce IAM authentication for dbs, think RDS Proxy.
It is never accessed publicly (must be from VPC)


AWS Aurora Global db vs DynamoDB Global Table
- Aurora Global Database: A relational database replicated across Regions.
    
- DynamoDB global tables: A NoSQL table replicated across Regions.
    


# RDS Event Subscription
Lets AWS notify you when specific events happen to your RDS db resources. You choose the events you care about, and RDS sends notification through SNS.

Events you can subscribe to include:
- Availability
- Failure
- Backup
- Configuration changes
- Maintenance
- Storage

NB: This doesn't cover database-level operations like "UPDATE, DELETE, INSERT"
