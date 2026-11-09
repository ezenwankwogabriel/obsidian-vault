AWS storage services fall into three categories

1\. Block Storage: Stores data in addressable blocks. An operating system sees it like a disk or drive
examples: EBS, EC2 instance store

2\. File storage: Stores data in files and directories, accessible through shared file system
examples: EFS, FSx

3\. Object storage: Stores data as objects in buckets, accessed through APIs.

## Block vs file vs object: practical example
Suppose you're building an application that stores customer information, uploaded documents and application files.
- PostgreSQL needs a disk to store database files → EBS (or managed RDS storage).
- Multiple EC2 instances need to access shared files → EFS.
- Customer profile pictures and videos → S3.
These are not interchangeable storage mechanisms.

## EC2 block storage: EBS vs Instance Store
The two most important storage types for EC2.

a\. Amazon EBS (elastic block store): provides persistent block storage volume you attach to EC2 instances. Think of it as a virtual hard drive

You launch an EC2 instance and attach a 100 GB EBS volume.
- The EC2 instance runs your application.
- The EBS volume stores your database files.
- You stop the EC2 instance.
- The EBS volume remains.
- You start the instance again and the data is still available.

Important: Creating an EBS volume does not automatically make it usable as a filesystem. The OS must initialize and mount it unless you're using it as a preconfigured root volume.

b\. EC2 Instance Store: Is a temporary block storage physically attached to the host computer running your EC2 instance

Example:
1. Launch an EC2 instance type with local NVMe storage.
2. The instance gets access to that local storage.
3. Your database writes data to it.
4. You stop or terminate the instance.
5. The data is lost.

why would anyone use this? Because some applications already have their own replication and recovery mechanisms.
Examples
- Cassandra distributed db
- temporary data processing
- caches
- high performance scratch space

If one fails, the app an rebuild its local data from another replica

### EBS vs Instance Store

| Characteristic       | EBS                            | Instance Store                              |
| -------------------- | ------------------------------ | ------------------------------------------- |
| Storage type         | Network-attached block storage | Local block storage                         |
| Persistence          | Persistent                     | Ephemeral                                   |
| Survives reboot      | Yes                            | Yes, normally                               |
| Survives stop/start  | Yes                            | No                                          |
| Survives termination | Depends on DeleteOnTermination | No                                          |
| Detachable           | Yes                            | No                                          |
| AZ scope             | One AZ                         | Tied to host/instance                       |
| Performance          | Depends on volume type         | Very high local performance                 |
| Typical use          | OS, databases, persistent data | Cache, temporary data, replicated workloads |

## EBS volume types 
AWS provides diff EBS volume types for diff workload requirements

- SSD: optimized for IOPS and low latency
- HDD: optimized for throughput and large sequential workload

#### SSD Volumes
gp3/gp3: general purpose ssd.
- Boot volumes
- Dev env
- General application workload
- Moderate db workloads
io2 Block Express: Provisioned IOPS ssd
- mission-critical db
- high iops requirements
- low-latency tx workload
- apps requiring predictable i/o performance

#### HDD volumes
st1 — Throughput Optimized HDD
- Big data.
- Data warehouses.
- Log processing.
- Large sequential reads and writes.

sc1 — Cold HDD
- Infrequently accessed data.
- Lowest-cost EBS HDD option.
- Large sequential workloads with low performance requirements

### Understanding IOPS vs throughput
This is a major exam concept.

IOPS = Input/Output Operations Per Second.

It measures how many read/write operations storage can perform per second.
Imagine a database executing 10,000 small random reads per second. High IOPS is important.

Throughput = Amount of data transferred per second.

Usually measured in MB/s or GB/s.
Imagine reading a 10 GB video file sequentially. High throughput matters.

### The four EBS types to remember

| Volume | Media | Main purpose                |
| ------ | ----- | --------------------------- |
| gp3    | SSD   | General purpose             |
| io2    | SSD   | High IOPS                   |
| st1    | HDD   | High sequential throughput  |
| sc1    | HDD   | Low-cost, infrequent access |

## Instance store and database replication
Consider a distributed NoSQL database such as Cassandra.

You configure a replication factor of three.

A record is stored on three database nodes:
- Node A: local instance store.
- Node B: local instance store.
- Node C: local instance store.
If Node A's EC2 instance is stopped, its local data is lost. Nodes B and C still have copies.

A replacement node can retrieve data from the remaining replicas.

This is the reasoning behind the original exam question.

But be careful: replication does not make instance store persistent. It makes the database architecture resilient to the loss of an individual node.

The database must have appropriate replication, consistency and repair mechanisms.

## Instance store vs EBS io2: which is faster?
This is where you must read the question carefully.

|Requirement|Strong candidate|
|---|---|
|Highest local I/O performance and lowest local latency|Instance store NVMe|
|High provisioned IOPS with persistent storage|EBS io2|
|High sequential throughput using HDD|EBS st1|
|General-purpose persistent SSD|EBS gp3|
If the question says:
The database replicates its data automatically and I/O throughput is the highest priority.

Instance store is a strong intended answer, especially if the options include NVMe instance store.

If it says:
The database needs high IOPS, but data must persist independently of EC2 instance lifecycle.

Then EBS io2 is a better choice.

This is the distinction you should apply rather than assuming instance store is always the answer to a high-performance database question.