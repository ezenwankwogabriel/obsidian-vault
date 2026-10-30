
### Question: 
If a major failure makes your primary application environment unavailable, how quickly can you restore service, and how much data can you afford to lose?

##### 1\. Starting with RTO and RPO
These concepts determine which disaster recovery architecture you should choose

RTO - Recovery Time Objective (downtime)
When you recover from a downtime
Covers how long an application can remain unavailable before service must be restored
ex: RTO = 1 hour. The biz needs service restored within 1 hour of interruption

RPO - Recovery Point Objective (data loss)
How often you run backups, how back in time can you recover if there was a loss
Covers how much recent data the business afford to lose, measured as a time interval
ex: RPO = 5 mins. The recovery point should be no more than 5 mins behind the interruption(up to 5 mins of recent updates could be lost)

Imagine a banking application fails at 12:00 noon.
- With an RTO of 30 minutes, service should be restored by 12:30 pm.
- With an RPO of 5 minutes, the recovery data should ideally include updates up to at least 11:55 am (5 mins before the interruption, so likely backups every 5 mins). 

##### 2\. The four AWS disaster recovery strategies (decreasing RTO)
a\. Backup and restore
Back up your data and config. After a disaster, restore the data and redeploy the app in the recovery environment. 

	Before Failure: Backup exists; recovery app may not be running
	After Failure: Restore data; provision infrastructure & redirect traffic
	Trade-off: This has the longest recovery time (tradeoff)

	examples: AWS Backup, EBS snapshots, RDS snapshots, S3 cross-region replication, cloudformation

b\. Pilot light
Keep critical data stores & replication for this data alone running in the recovery Region, but leave app servers and other non-core resources unprovisioned / switched off. DB is replicated continously, but EC2 instances are turned off in replicated zone
	Before failure: core infrastructure and  replicated data are ready
	After failure: Provision app servers, deploy or activate required components and scale up
	Trad-off: faster than rebuilding from scratch, but activation is required
	
	AWS examples: RDS read replicas, Aurora Global Database, S3 replication, infrastructure as code.

c\. Warm standby
Maintain a smaller but fully functional copy of your application in another Region (all app setu p is ready, but ran at a small scale,  can be scaled easily in situations of disaster)
	Before Failure: the recovery application is running and can handle some traffic
	After Failure: Redirect traffic and scale the environment to full production capacity
	Trade-off: Faster recovery, but higher ongoing cost than pilot light

	AWS examples: EC2 Auto Scaling, replicated databases, Route 53 health checks or AWS Global Accelerator.

d\. Multi-site / Hot Site 
Run the application in multiple Regions simultaneously, with all participating Regions serving production traffic
	Before Failure: Multiple region serve users
	After Failure: Route traffic away from failed Region to remaining healthy Regions
	Trade-off: Highest complexity and typically highest cost, but potentially near-zero recovery time for regional failures
	
	AWS examples: Route 53, Global Accelerator, cross-Region data replication, and carefully designed multi-Region databases.

***Of the 4 above, only Backup & restore offers backup in its foundations. the other 3 do not guarantee unless backups are configured.***

4\. High Availability is not the same as disaster recovery
a\. High availability keeps the workload operating through common failures, such as an EC2 instance or AZ being unavailable

b\. Disaster recovery restores the workload after a larger event, potentially including loss of an entire AWS Region or destructive data corruption

Ex: running EC2 across multiple AZ with an ALB and Auto Scaling improves availability. But if some accidentally deletes critical prod data and that deletion is replicated, the highly available application may continue operating with the wrong data

This is why DR needs recoverable backpus, versionings, or point-in-time recovery–– not just redundancy

### Disaster Recovery Tips
	Backups:
		Can use EBS snapshots, RDS automated backups / snapshots ...
		Regular pushes to S3 / S3 IA, Glacier, lifecycle policy, cross region replication
		From On-premise: use snowball(hardware), storage gateway (any of the types)

	High Availability
		Use Route53 to migrate DNS over from Region to Region
		RDS Multi-AZ, ElastiCache multi-AZ, EFS, S3
		Site-to-Site VPN as a recovery from Direct Connect

	Replication
		RDS Replication (Cross region), AWS Aurora + Global DBs
	
	Automation
		 CloudFormation / Elastic Beanstalk to re-create a whole new environment
		 Recover / Reboot EC2 instances with CloudWatch if alarm fail
		 AWS Lambda functions for customized automations
	
	Chaos
		Netflix has a "simian-army" randomly terminating EC2, like chaos-monkey to be sure their application is resilient

---
# AWS Elastic Disaster Recovery (DRS)
A disaster recovery service that helps you recover servers running on-premises or in other cloud envs on AWS when a disaster occurs

*key*: continuously replicate your source servers to AWS, then launch recovery instances when needed

Designed to reduce downtime & data loss without requiring you to maintain a fully running copy of every prod server in aWS

The main steps are:
1. Install the AWS Replication Agent on each supported source server.
2. AWS DRS continuously replicates *disk changes* to AWS.
3. Replicated data is maintained in a staging area, generally using EBS-backed storage.
4. If the source environment fails, initiate a recovery job.
5. DRS launches recovery instances using the replicated data and your recovery settings.

---
# Database Migration Service (DMS)
This is a managed service for migrating databases and replicating db changes btw data stores

This is used when a company wants to move a database to aWS, migrate btw db engines or keep db synchronized during a migration

To do this, you create an EC2 instance to perform the migration tasks.

If the DB engine is different, you'd need a Schema Conversion Tool (SCT) for it to be migrated, else SCT is not needed

---
# AWS RDS and Aurora Migrations
Key is to distinguish migrating a db, upgrading its engine, or moving it to another aWS Regions and switch btw MySQL and Aurora. 

Scenario A –– RDS MySQL to RDS MySql
This is same db engine. Common choice includes:
	a snapshot restore
	native MySQL replication
	AWS DMS when you need to minimize downtime

Scenario B –– RDS mysql to aurora mysql-Compatible
Same general db engine family, but different db platform. Options include:
	snapshot restore
	using Auroras migration feature
	replicating changes before cutover

scenario C -- MySQL to PostgreSQL
A heterogeneous migration. You generally need schema conversion and data migration, with AWS DMS often used to move data and replicate changes

scenario D –– External MySQL to Aurora MySQL
	Option 1
		use percona XtraBackup to create a file backup in Amazon S3
		Create an Aurora MySQL DB from Amazon S3
	Option 2
		Create an Aurora MySQL Db
		Use mysqldumb utility to migrate (slower than s3 method)

---
# On Premise strategy with AWS
You have the ability to download Amazon Linux 2 AMI as a VM (in .iso format)
Then can load this iso image into the software used to create VM
allowing you run Amazon linux in your own premise VM

### VM import / export
There's a feature called VM import / export. This allows you to:
	migrate existing applications VM into EC2. 
	Create a Disaster recovery strategy for your on-premise VMs
	 Export back the VMs from EC2 to on-premise

### AWS Application Discovery Service
This would help gather information about your on-premise servers to plan a migration
And you can track these information using AWS Migration Hub

AWS Database Migration Service
Helps you replicate on-premise => AWS, AWS => AWS, AWS => on-premise

### Application Migration Service (MGN)
For incremental replication of on-premises live servers to AWS. This is a rehost (lift and shift) service. 
### Application Migration Service vs. DRS vs. DMS

These three services are easy to confuse on the exam.

| Requirement in the question                                                    | Service                                                        |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| Move physical or virtual servers to EC2 with minimal changes                   | AWS Transform MGN (formerly AWS Application Migration Service) |
| Recover servers after a disaster, including failback to the source environment | AWS Elastic Disaster Recovery (DRS)                            |
| Migrate database records between supported database engines                    | AWS Database Migration Service (DMS)                           |

The distinction between MGN and DRS is especially subtle because both use continuous block-level replication technology. MGN is focused on server migration and cutover; DRS focuses on disaster recovery, including recovery and failback capabilities.

One current naming detail for your study notes: AWS renamed Application Migration Service to AWS Transform MGN in June 2026. Older SAA-C03 materials and practice questions may still use the former name.

---
# AWS Backup
This is a fully managed AWS service allowing you to centrally auto manage & automate backups across AWS services.

There's a Backup Vault Lock
This enforces a WORM (write once read many) state for all your backups that are stored in the vault. Once written, is proteded, and cannot be modified or deleted for a specified retention period

---
# VMware Cloud on AWS
VMware Cloud on AWS lets organisations run their VMware-based workloads on AWS infrastructure without having to immediately redesign or rewrite their applications.

Think of a company that already runs hundreds of virtual machines in its own data centre using VMware vSphere. It wants to move those workloads to AWS but keep its familiar VMware environment and management tools.

Instead of migrating every application to native EC2 architectures immediately, the company can use VMware Cloud on AWS to run supported VMware workloads on AWS infrastructure.