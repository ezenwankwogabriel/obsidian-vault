Provides fully managed file storage using popular file systems
	S3 stores  objects as images, ...
	EBS provides block storage for an EC2 instance
	EFS provides shared, elastic file storage (primarily for linux workloads)
	FSx provides managed file systems with specialized capabilities for Windows File shares, high-performance computing, enterprise storage

Imagine your company has 100 Windows computers that all need access to the same shared company folder. Rather than building and maintaining your own Windows file server, you could use Amazon FSx for Windows File Server.

## Four FSx file systems
1\. FSx for Windoes File Server
Provides shared file storage using the SMB (server message block) protocol, with support for Microsoft Active Directory.

Use it for Windows home directories, shared company folders, and Windows applications that expect a traditional file server.

Even though it was built for windows, can be mounted on Linux EC2 instances

Storage Options: SSD, HDD

Memory trick: Windows + NTSF + SMB + Active Directory.

2\. FSx for Lusture (linux + cluster) designed for HPC (high performance computing)
Designed for high-throughput workloads that process large datasets in parallel, including machine learning, scientific simulations, and media processing.

It can integrate with S3, allowing supported workflows to import data from S3 into the file system and export results back to S3.

Seamless integration with S3. can read s3, and write the output

Storage Options: SSD, HDD

Memory trick: Lustre = large-scale, parallel processing.

3\. FSx for NetApp ONTAP
Provides enterprise-grade shared storage with multiple access protocols, including SMB, NFS, and iSCSI. It supports features such as snapshots, cloning, and data replication.

Use it when migrating existing NetApp workloads or when applications need flexible enterprise storage capabilities.

Memory trick: ONTAP = enterprise storage with flexible protocols.

4\. FSx for OpenZFS
Provides high-performance file storage based on OpenZFS, with features including snapshots, compression, and point-in-time clones.

Use it only for workloads migrating from ZFS-based systems or applications that benefit from low-latency NFS file storage.

Memory trick: OpenZFS = fast file access, snapshots, and clones.


## Deployment options
1\. Scratch File System
	Temp storage
	Data not replicated (not persisted if server fails)
	use-case: short-term processing, optimize cost

2\. Persistent File System
	Long term storage
	Data is replicated within same AZ
	Replace failed files within minutes
	use-case: long-term processing and sensitive data