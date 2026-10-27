Think of this as instructions to AWS about where to put your servers in its data centers
On launching multiple EC2 instances, you tell AWS how those instances should be physically arranged to optimise performance or availability

## Types
1. Cluster: Keeps instances close together (in same AZ) to provide low network latency and high network throughput (likely using private network connections)
		used for computing requiring servers to communicate very quickly
2. Spread: Keeps instances on separate underlying hardware. Spreading instances across hardware to reduce the chance of one hardware failure affecting multiple instance. Each instance in separate hardware. Group of hardware in an AZ, having multiple AZ of different hardware, each containing an instance.
		A small number of critical servers, such as domain controllers or essential application nodes, that must avoid sharing the same underlying hardware.
3. Partition: Divide instances into separate partitions. Each having its own set of racks, helping reduce the impact of hardware failure across groups of instances
		used for large distributed systems like Hadoop, Kafka, cassandra running many instances

![[Pasted image 20261014162312.png]]







