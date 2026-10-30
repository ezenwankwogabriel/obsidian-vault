A managed RDS compatible with MySQL and PostgreSQL.

## Aurora's key features

| Feature                | What it means for the exam                                                                                                  |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| High availability      | Storage is replicated across 3 AZs, with 6 copies of data.                                                                  |
| Automatic failover     | If the writer fails, Aurora can promote a reader to become the new writer.                                                  |
| Read scaling           | Add Aurora Replicas to distribute read traffic.                                                                             |
| Storage autoscaling    | Storage grows automatically as data increases; you don't manually provision the full volume upfront.                        |
| Automated backups      | Continuous backups support point-in-time recovery within the configured retention period.                                   |
| Aurora Global Database | Replicates data to secondary AWS Regions for cross-Region disaster recovery and low-latency reads.                          |
| Aurora Serverless      | Automatically adjusts database compute capacity according to demand, depending on the Serverless version and configuration. |

## Aurora endpoints

Cluster (writer) endpoint: Routes connections to the current writer (for INSERT, UPDATE, DELETE)

Reader endpoint: Distributes connections across available Aurora Replicas for read scaling

Instance endpoints: Connects to a specific DB instance. For when you need to target a particular reader / writer, but must account for failovers and role changes

Custom Endpoint: routes connections to a defined subset of instances. For directing reporting queries to larger / smaller replicas according to your design.
The custom endpoint provides load-balanced database connections based on criteria other than the read-only or read-write capability of the DB instances. For example, you might define a custom endpoint to connect to instances that use a particular AWS instance class or a particular DB parameter group. Then you might tell particular groups of users about this custom endpoint. For example, you might direct internal users to low-capacity instances for report generation or ad hoc (one-time) querying, and direct production traffic to high-capacity instances.


**Amazon Aurora** typically involves a cluster of DB instances instead of a single instance. Each connection is handled by a specific DB instance. When you connect to an Aurora cluster, the host name and port that you specify point to an intermediate handler called an _endpoint_. Aurora uses the endpoint mechanism to abstract these connections. Thus, you don't have to hardcode all the hostnames or write your own logic for load-balancing and rerouting connections when some DB instances aren't available.

