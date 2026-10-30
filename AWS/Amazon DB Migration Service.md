## AWS Database Migration Service (DMS)

DMS moves db data from source to a target, and can continuously replicate changes while minimizing downtime. 

It supports homogenous and heterogenous migrations

use-cases:
1\. Database migration: move data from on-premises db to an AWS target

2\. Continuous replication: keep a target synchronized with ongoing source changes, useful for minimizing migration downtime or maintaining a replica

3\. Analytics and data lakes: replicate supported data into s3, redshift, and other supported targets for analytics & reporting

4\. Disaster recovery & data distribution: keep data synced btw supported data stores. DMS replication alone, however, does not provide a complete disaster recovery strategy

#### Migration types: 
1\. Full Load (one time copy): 
DMS copies the existing data from source to target in full
After completion, later source changes are not continuously replicated by a full-load-only task

2\. CDC only (changes only)
DMS captures changes occuring after the configured CDC starting point.
Example: the target has already been populated, and you want to apply subsequent inserts, updates, and deletes.

3\. Full load + CDC (initial copy + ongoing changes)
DMS copies existing data while capturing changes made during the load, then applies the captured changes and continues ongoing replication.

Example: migrate a production database to Aurora while keeping the source online during the migration.

#### Supported sources and targets

| Source                 | Example target            | Typical use                        |
| ---------------------- | ------------------------- | ---------------------------------- |
| On-premises MySQL      | Amazon Aurora MySQL       | Database migration                 |
| On-premises PostgreSQL | Amazon Aurora PostgreSQL  | Migration with CDC                 |
| Oracle                 | Amazon RDS for PostgreSQL | Heterogeneous migration            |
| Amazon RDS database    | Another RDS database      | Consolidation or migration         |
| Relational database    | Amazon S3                 | Data lake and downstream analytics |
| Relational database    | Amazon Redshift           | Analytics and data warehousing     |
| Supported source       | Amazon DynamoDB           | Data migration to a NoSQL target   |
| Supported source       | Amazon OpenSearch Service | Search and analytics use cases     |