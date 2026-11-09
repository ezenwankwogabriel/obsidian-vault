A managed relational database engine compatible with MySQL and PostgreSQL.
It is a part of amazon RDS but has its own distributed storage architecture
### Architecture
Aurora has two main components:

1\. DB instances / Compute: The resources that execute db queries
They provide: `cpu and memory` , `db engine execution`, `query processing`, `connection handling`, `transaction processing`

Aurora has two principal instance roles:

a\. Writer Instance
This is the primary instance responsible for modifying the db data
The writer processes this request and commits the changes to the shared cluster storage

b\. Reader Instance
These are additional DB instances that can execute read-only queries

Instead of sending every read request to the writer, the application can distribute read requests across reader instances (called `Read Scaling`)

Though Readers do not maintain separate independent db. They access the same underlying cluster storage

2\. Cluster storage: The infrastructure that permanently stores db data.
This is where the actual db data lives

Imagine your application has:
- 10 million customers
- 50 million orders
- 5 million products
The db records and associated db structure are stored in the Aurora cluster volume

This separates its storage from the DB instances (query makers / resolvers)

```
Writer instance ─────┐
                     │
Reader instance 1 ───┼──── Shared cluster storage
                     │
Reader instance 2 ───┘
```

If you add another reader, you aren't creating another full copy of the database. You're adding more compute capacity to access the existing data.


![[Pasted image 20261101181435.png]]


### How is the storage protected?
Aurora automatically maintains six copies of data across 3 AZ:
- Two copies in AZ-A.
- Two copies in AZ-B.
- Two copies in AZ-C.
Designed to tolerate loss of up to two copies without affecting write availability, and up to three copies without affecting read availability.

Also storage grows as your db grows

### What happens when something fails?
Imagine your cluster has:
- Writer in AZ-A.
- Reader in AZ-B.
- Reader in AZ-C.
- Shared cluster storage across all three AZs.

![[Pasted image 20261101183657.png]]

Scenario A: Writer instance fails
The writer becomes unavailable, Aurora identifies an available reader, then promotes that reader to become a new writer. The cluster writer endpoint points to the new writer.

Scenario B: An Availability Zone fails
the writer in AZ becomes unavailable, cluster storage still has copies in AZ-B and AZ-C. An available reader can be promoted to writer

Scenario C: You need more read capacity
suppose you need more capacity in case of a sure in SELECT queries. Instead of increasing writer's capacity, you can add reader instances

## How does your app connect to Aurora?
Aurora provides different endpoints for diff purposes. They determine which DB instance receives application's connection.

| Endpoint          | What it connects to           | When to use it                       |
| ----------------- | ----------------------------- | ------------------------------------ |
| Cluster endpoint  | Current writer                | INSERT, UPDATE, DELETE and writes    |
| Reader endpoint   | Available readers             | Read scaling                         |
| Instance endpoint | A particular instance         | Connecting to a specific DB instance |
| Custom endpoint   | A selected group of instances | Workload-specific routing            |

The application sends write transactions to the writer and read-only queries to the reader endpoint.

Note: The reader endpoint balances connections, not necessarily every individual SQL query. Connection pooling and long-lived connections affect how traffic is distributed.

## Where does Aurora Serverless fit into this architecture?
Remember we have two fundamental components: Compute and Storage

Aurora serverless changes how the compute capacity is provisioned & scaled. It doesn't replace the cluster storage architecture

There are two main compute options:

Option A: Provisioned Aurora
You select the DB instance class and capacity
	example: 
		Writer: db.r6g.large
		Reader: db.r6g.large
AWS runs those instances at the selected capacity. If demand increases you can resize instances or add readers

Option B: Aurora Serverless
You specify the minimum and maximum compute capacity in ACUs. Aurora auto adjusts compute capacity according to workload demands
	example:
		Minimum: 2 ACUs (Aurora Capacity Unit)
		Maximum 32 ACUs

## How does Aurora Global fit in?
So far, it's all been having the db across AZs within one Region

What happens if an entire AZ becomes unavailable? Thats where Global DB comes in

## Aurora's key features

| Feature                | What it means for the exam                                                                                                  |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| High availability      | Storage is replicated across 3 AZs, with 6 copies of data. (2 per AZ)                                                       |
| Automatic failover     | If the writer fails, Aurora can promote a reader to become the new writer.                                                  |
| Read scaling           | Add Aurora Replicas to distribute read traffic.                                                                             |
| Storage autoscaling    | Storage grows automatically as data increases; you don't manually provision the full volume upfront.                        |
| Automated backups      | Continuous backups support point-in-time recovery within the configured retention period.                                   |
| Aurora Global Database | Replicates data to secondary AWS Regions for cross-Region disaster recovery and low-latency reads.                          |
| Aurora Serverless      | Automatically adjusts database compute capacity according to demand, depending on the Serverless version and configuration. |

## Backsups
Aurora has its native backup mechanism. You don't need AWS Backup to perform a standard Aurora backup. 
However, it integrates with AWS Backup if you want centralized backup management

1\. Aurora native backups
Aurora automatically backs up your cluster data to AWS-managed storage (S3).

Key characteristics:
- Automated backups are enabled by default.
- Continuous, incremental backups.
- Supports Point-in-Time Recovery (PITR).
- Retention period: **1–35** days. (exceeding 35 days retention, should use AWS Backup)
- Backups don't affect database performance significantly.
- You can restore to a new Aurora cluster at a selected point in time.

2\. Manual snapshots
Aurora also supports manual DB cluster snapshots.

Unlike automated backups:
- You initiate them yourself.
- They remain until you explicitly delete them.
- You can copy them across AWS Regions.
- You can use them to restore a new cluster.