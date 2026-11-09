### EBS Volume
This is a network drive you can attach to your instances while they run. Persists data after their termination
- Can be mounted to one instance at a time, bound to a specific AZ
- It can be detached from an EC2 instance and attached to another one quickly
- Can have two EBS volumes attached to one instance
- Think of these as a "network USB stick"
- you can create and leave it unattached to any instance


## EBS Snapshot
- Make a backup of your EBS volume
- Recommended to detach volume to do a snapshot but not necessary
- Can copy snapshots across AZ or Region

Snapshop Archive: Move a snapshot to an achieve tier that's 75% cheaper, takes within 24-72 hours for restoring the archive

Recycle Bin for EBS Snapshot. for recovery for accidental deletion (1day - 1year retention)

Fast Snapshot Restore: force full initialization of snapshot on first use (very expensive)

EBS volumes are network drives with good limited performance
If need be for a high performance hardware disk, use an EC2 Instant Store

### Instant Store
This is a temporary, physically attached storage on the host machine running your EC2 instance.
- Better I/0 performance
- Caveat: stopping the instance, ec2 instant store lose their storage (ephemeral). But it is preserved on reboot
- Ensure to backup if you choose to go with this for reliability
use-case: it is good for buffer/cache/temp content (not for long term storage)
#### IOPS (Input/Output Per Second)
Measures how many read & write operation a storage device can perform in one second.
Low IOPS: the librarian retrieves 100 books per second (request takes longer to complete)
High IOPS: the librarian retrieves 10,000 books per second (Many more requests handled quickly)

### EBS Volume Types
- General Purpose SSD Volume
	- Cost effective storage, low-latency
	- Balances price & performance for wide variety of workloads
	- System boot, Virtual desktop, Dev & test env, medium-sized db, transactional workloads
		- gp3: (1 GiB - 64 TiB) 
			- Baseline of 3,000 IOPs and throughput of 125MiB/s
			- Can increase IOPs up to 80,000 & throughput up to 2000MiB/s
		- gp2: (1 GiB - 16 TiB)
			- Small gp2 volumes can burst IOPs of 3000, max IOPs is 16000
- Provisioned IOPS (PIOPS) SSD
	- For critical biz app with sustained (highest) IOPS performance
	- Or apps needing more than 80000 IOPS
	- great for db workloads (sensitive to storage perf & consistency)
		- io 1 (4 GiB - 16 TiB)
			- max PIOPS: 64,000 for Nitro EC2 instance & 32,000 for other
		- io 2 (4 GiB - 64 TiB)
			- sub-millisecond latency
			- max piops: 256000 with an iops:gib ratio of 1000:1
- Hard Disk Drives (HDD)
	- Cannot be a boot volume
	- 125gib to 16tib
	- throughput optimized (st 1)
		- big data, data warehouse, log processing. 
		- max throughput: 500mib - max IOPS 500
	- Cold HDD (sc 1)
		- for data that is infrequently accessed
		- where lowest cost is important
		- max throughput is 250mib/s - max iops 250
- st 1 (HDD): Low cost HDD volume designed for frequently accessed, throughput intensive workloads
- sc 1 (HDD): lowest cost HDD volume designed for less frequently accessed workload

*only gp2/gp3 and io 1/io 2 can be used as boot volumes (i.e where root OS is running)* [the SSD candidates].
The EBS volumes are characterized in Size|Throughput|IOPS (I/O Ops per sec)

### Here is a list of important information about EBS Volumes:
- When you create an EBS volume in an Availability Zone, it is automatically replicated within that zone to prevent data loss due to a failure of any single hardware component.
	important: this replication is not a backup. If you have corrupted data, it'd also be reflected in the replicated storage

- An EBS volume can only be attached to one EC2 instance at a time.

- After you create a volume, you can attach it to any EC2 instance in the same Availability Zone

- An EBS volume is off-instance storage that can persist independently from the life of an instance. You can specify not to terminate the EBS volume when you terminate the EC2 instance during instance creation.

- EBS volumes support live configuration changes while in production which means that you can modify the volume type, volume size, and IOPS capacity without service interruptions.

- Amazon EBS encryption uses 256-bit Advanced Encryption Standard algorithms (AES-256)

- EBS Volumes offer 99.999% SLA.

io2 Block Express is the provisioned-IOPS tier built for exactly this: tens of thousands of sustained IOPS at sub-millisecond latency with the 99.999% durability SLA. gp3 tops out at 16,000 IOPS per volume, so it cannot meet the number. st1 is a throughput-oriented HDD and is not even permitted as a boot volume. Instance store delivers high raw IOPS but is **ephemeral** — stopping or terminating the instance destroys the data, which fails the durability requirement outright.


### EBS Multi-Attach - io 1 / io 2 family
allows you to attach same EBS volume to multiple EC2 instances in same AZ
only available for the io 1 and io 2 family of ebs volumes
*much useful in applications that must manage concurrent write operations*
- One limitation is that it cannot not be applied cross AZ
- Also, it allows up to 16 EC2 instances to be applied to at a time
- To work, you must use a file system that's cluster-aware (not XFS, EXT4, ...)

### Encryption
EBS Encryption leverages keys from KMS (AES 256)
- Create an EBS snapshot of the volume
- Encrypt the EBS snapshot (using copy) when you copy a snapshot, you enable encryption
- Create new EBS volume from the snapshot (the volume will also be encrypted)
- Now you can attach the encrypted volume to the original instance
- Encryption has very little impact on memory
