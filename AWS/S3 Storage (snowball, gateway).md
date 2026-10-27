1\. Snowball (offline data transfer)
	This is part of AWS's family of physical data-transfer services.
	
	Helps you move large amounts of data into/out of AWS using a physical device instead of relying entirely on an internet connection.
	
	Picture a large company in lagos with 200TB of video footage stored in local servers and want to move it to Amazon S3. Doing so over internet would take a long time. Snowball is used here, copy data into it locally, then return the device to AWS, and have AWS import the data into its destination

2\. Snowball Edge (edge computing)
	This is an important distinction. Isn't merely a hard drive in a shipping container. It has onboard storage and compute capabilities, allows supported workload to run at remote locations with limited/no internet connectivity.
	
	an example is an oil facility with poor connectivity could collect sensor data, process it locally, and transfer results to aws later

3\. DataSync
	Transfers data online over a network. Choose if you have sufficient network and want managed, auto data movement  btw supported storage systems & aws
	Move large amount of data to and from
		on-premise/other cloud to AWS
		AWS - AWS (diff storage services)
	Perserves File permission & metadata (NFS POSIX, SMB)

4\. AWS Data Transfer Terminal
	Provides a physical facility where customers can bring storage devices and transfer large datasets securely into AWS. It's an alternative to investigate when online transfer isn't practical. Availability and access depend on the supported locations and booking arrangements.


## 1. Compare the data-transfer services

|Service|What makes it different|Typical exam scenario|
|---|---|---|
|AWS DataSync|Managed data transfer over a network|Efficiently migrate NFS/SMB data to S3, or transfer data between supported storage locations|
|AWS Snowball Edge|Physical device with storage and, for supported configurations, compute capabilities|Large offline data transfers or supported edge workloads where network connectivity is limited|
|AWS Data Transfer Terminal|Physical facility for customers to bring storage devices and transfer data into AWS|Large datasets need physical transfer, and a supported terminal is available|
|AWS Storage Gateway|Connects on-premises applications to AWS-backed storage|Continue accessing files, block volumes, or virtual tapes through familiar interfaces|

## Hybrid Cloud
AWS is pushing for "Hybrid Cloud": 
	Part of the infrastructure stays on the cloud
	Part of it is on-premise
Reasons:
	Long cloud migration
	Security requirements
	Compliance ""
	IT Strategy

### *AWS Storage Gateway* (a ongoing hybrid cloud-sync service)
This helps a company connect its on-premises apps and storage system to AWS Cloud storage

*Storage gateway is designed for ongoing hybrid-cloud storage access, if question asks for a full migrate of existing data, it wouldn't be a storage gateway*

*Key:* Your company keeps working with familiar local storage, while AWS provides cloud storage behind it

This is the bridge btw on-premise data and cloud data. Could be for disaster recovery, backup, tiered storage, on premise-cache & low-latency files

Imagine a hospital in Lagos has applications that expect files or disks to be available inside its own data center. The hospital wants to store data in AWS without immediately rewriting those applications or changing every workflow.

## Gateway Types:

1\. Amazon S3 File Gateway

Your app sees a **shared** file system, while the files are stored as objects in S3. The gateway caches data locally for faster access

NFS/SMB: File Gateway exposes s3 objects as files through NFS or SMB file shares
Supports SMB authentication and authorization through Active Directory integration
Data remains in S3: files are stored in S3, gateway provides local access and caching

Use-Case: An on-premise app saves files to a shared folder, and the company wants those files stored in S3.

2\. Tape Gateway

Replace physical backup tapes with virtual tapes stored in AWS. Existing backup software can continue using a tape-based workflow.

Use-Case: Company wants to move its tape backups to S3, archive them to Glacier storage classes without replacing its backup application

3\. Volume Gateway

Presents iSCSI block storage, not NFS or SMB file shares

Is suitable when an on-premise app expects a disk volume. Its cached and stored mode determines where primary data resides.

Presents storage volumes to on-premise servers as block devices. Supports two config, a cached volume and a stored volume

Cached volumes: cloud holds the main dataset. low latency access to frequently accessed files
Stored volumes: whole dataset is on premise (on the site), scheduled backup to S3

Use-Case: An existing app expects a disk volume, and the company wants cloud-backed storage or off-site backup

4\. Amazon FSx File Gateway

Provides on-premises access to Amazon FSx for Windoes File Server shares.

Provides low-latency access from on-premises environments to supported Amazon FSx for Windows file server shares in aws

Use-Case: Employee needs familiar Windows file shares hosted in AWS while accessing them from an on-premises office

See: obsidian://open?vault=obsidian-vault&file=AWS%2FAmazon%20FSx

Block Storage: Amazon EBS, EC2 Instance Store
File Storage: EFS, FSx
Object Storage: Amazon S3, Amazon Glacier

Storage Comparison:
	EBS volumes: Network storage for one EC2 instance at a time
	Instance Storage: Physical storage for EC2 instances (high ioFS)
	EFS: Network file system for linux instances
	Storage Gateway: S3 & FSx Gateway, Volume Gateway (cache/stored), Tape Gateway
	Transfer Family: (S)FTP(S) interface on top of AWS S3/EFS
	DataSync: Schedule data sync from on-premises to AWS/ AWS-AWS
	Snowcone/Snowball/Snowmobile: to move large amount of data to the cloud, physically
	

## AWS Transfer Family
A file transfer service for file transfer in/out of Amazon S3/EFS using FTP protocol

A Machine Learning company is working on a set of datasets that are hosted on S3 buckets. The company decided to release those datasets to the public to be useful for others in their research, but they don’t want to configure the S3 bucket to be public. And those datasets should be exposed over the FTP protocol. What can they do to do the requirement efficiently and with the least effort?