This is Amazons fully managed NFS (network file system) that can be mounted on many EC2
it works with EC2 instances in multi-AZ within one Region
Think it as a shared folder in the cloud that several servers can read from and write to
Highly available, scalable, expensive (3x gp2), pay per use

use-cases: content-management, web serving, data sharing, wordpress
to control access to EFS, use security group
- only compatible with linux based AMI (not windows). for windows-based EC2, use Amazon FSx for Windows File Server
- encryption at rest using KMS
- You don't have to plan capacity in advance, the file system would scale automatically
### 3 concepts to EFS
#### EFS Scaling
Unlike trad disk where you might need to increase allocated size, EFS auto grows & shrinks as you add / remove files. No need to pre-provision storage.

EFS is elastic, hence no need to pre-provision storage size in advance.

Though scaling storage is diff from scaling perf. For performance we need to understand two things below
#### Performance mode: how quickly EFS respond?
This is about the responsiveness of individual file operations & the scale of the parallel operations
##### General Purpose
- lowest latency –– operations respond faster
- best for most app, including websites, CMS, and normal file sharing
- this is the default performance mode
##### Max I/O
- designed for higher parallel workloads needing very high aggregate I/O
- Has higher latency
- Is a legacy option: doesn't support Elastic throughput / One Zone file system

*AWS recommends General Purpose for all file systems because of its lower latency*

Example: Imagine 1k workers requesting files from a warehouse
- General Purpose: Optimises how quickly individual requests are answered
- Max I/O: was designed for workloads with very large numbers of parallel requests, accepting slower individual responses

Performance mode = how EFS handles request

##### Throughput mode –– how much data can EFS transfer
This determines the data transfer capacity available to the EFS file system. there are three modes
- Elastic throughput (unpredictable)
	- EFS auto adjusts throughput to match workload
- Provisioned throughput (known, sustained workload)
	- you specify the throughput your app needs, independently of the amount of data stored
- bursting throughput (storage size + burst cretis)
	- when throughput depends on the amount of data stored in EFS Standard, and the FS can exceed its baseline using burst credit.
	- an example is a small dev FS having occasional periods of heavy activity
	- : 1 TB = 50mib/s + burst of up to 100mib/s

### EFS Storage Class (Standard, IA, Archive)
EFS has different storage classes depending on how frequently you access your files.
The goal is simple: frequently used files cost more to store, while rarely used files can be moved to cheaper storage.

- EFS Standard: for frequently accessed files. offers low latency (files used by a busy website) [multi zone]
- EFS Infrequent Access (IA): Cheaper storage storing files accessed few times per quarter (old reports) [one zone]
- EFS Archive: files accessed only a few times per year or less. the cheapest of the three (historical records kept for compliance)

To move files between tiers, you implement a *lifecycle policy*
