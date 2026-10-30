A centrally managed service for automating, organizing, and restoring backups across AWS.

Each storage service has its own native backup and recovery features. AWS Backup lets you manage supported resources through a centralized service instead of configuring every backup workflow independently.

### Backup plan

Defines when backups run, which backup vault receives them, how long recovery points are retained, and whether copies are created in another Region or account.

Example: Back up production databases daily and retain recovery points for 35 days.

### Backup vault

A logical container that stores and organizes recovery points. A vault has an encryption configuration and can have an access policy and Vault Lock configuration.

Example: Keep production database backups in a restricted vault.

### Recovery point

A particular backup that you can use to recover a supported resource. Depending on the service, it may represent a snapshot, a database backup, or another supported recovery format.

Example: Restore a database to the state captured by yesterday's recovery point.

### Lifecycle policy

Controls how long a recovery point is retained and, where supported, when it transitions to a colder storage tier or expires.

Example: Retain daily backups for 35 days and monthly backups for seven years.

