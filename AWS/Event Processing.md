SQS + Lambda:
	Events in SQS, Lambda polls message from the SQS queue
	In case of issues, it puts back the message in SQS queue and tries to retry it (this could cause a loop)
	In case of this loop, you can setup a Dead Letter queue (DLQ) after about 5 tries

SQS FIFO + Lambda:
	Messages are processed in order.
	Since this is processed in order, if one fails, it retries, and blocks others from processing
	Setup a DLQ to send this message off the queue for message continuity

SNS + Lambda:
	Messages are sent async to Lambda
	In case it couldn't process, it retries, but internally (not polling from a queue). 
	It retries 3 times, and if not successful, message is either discarded, or sent to a DLQ but within the Lambda environment not the queue/SNS

Fan out pattern: 
How you delivery data to multiple SQS queue.
Imagine you have an e-commerce app that receives a new order. That single order event needs to trigger three separate tasks (3 SQS queue).

Ideally, you'd manually send to individual queues.

With the fan out, you publish only one event, and AWS distributes it to the intended customers. You send a message to an SNS topic, which sends (fans out) to other SQS queues. The sqs queue listens as subscribers to the topic.

---
# S3 Event Notifications
Reacting to specific events on S3. send event to SNS, SQS, Lambda functions

# EventBridge
From S3 -> Amazon EventBridge. And with rules, you can send that to about 18 AWS services as destination
The event bridge gives advaned filtering options wiht JSON rules (metadata, object size, name)
Can archive, replay, ...
Can intercept any API call using cloudtrail

---
# High Performance Computing (HPC)

This means using powerful computers, often working together, to solve computationally demanding problems quickly.

The cloud is the perfect place to perform HPC. Can create high number of resources, speed up time to results by adding more resources, and pay for only systems you've used 

Imagine you're asked to solve a problem that would take one computer 100 hours.

Instead of making one computer do all the work, you divide the problem among many computers that work in parallel.

Examples: weather forecasting, aircraft simulations, scientific research, financial modelling, and engineering design.

### What makes an HPC system fast?
Think of HPC as a team of workers. Four things determine how effective they work

1\. Compute - EC2: The workers doing the calculations. Choose suitable EC2 based on CPU, memory and GPU requirements

2\. Networking - EFA: How quickly the workers communicate. Helps supported instances exchange data with low latency and high throughput

3\. Storage - FSx for Lustre: How quickly workers access large shared datasets
	Best fit for HPC workloads requiring high-throughput parallel files

4\. Job Management - AWS Batch, ParallelCluster, or PCS
	Organizes jobs, allocates resources, and helps get work running on the cluster

### Two types of HPC workloads
1\. Tightly coupled workloads: (sync)
	Picture a team solving a big problem, each work needs information from the others
		ex: simulating how air flows around an aircraft
		main problem: workers must communicate very quickly
	Think: EFA + cluster placement group

2\. Loosely coupled workloads
	Picture team processing 10k independent photos. Each can process diff photo individually
		ex: running independent financial simulations
		main problem: efficiently scheduling lots of jobs & scaling capacity
	Think AWS Batch; Spot instances may reduce cost if jobs can tolerate interruption

### EFA & Cluster Placement groups
###### EFA - Elastic Fabric Adapter
	This is a network interface that helps supported EC2 instances communicate efficiently for HPC workloads, particularly those using MPI
	*key*: EFA improves communication between workers

###### Cluster placement group
	places supported EC2 instances close together in same AZ to support low-latency, high throughput networking
	*key*: a placement group infuences where the workers are placed
	
	don't confuse this with spread placement which prioritizes separating instances across hardware to reduce correlated failures

###### Exam clue: 
If the question mentions frequent communication between nodes and network latency, consider EFA. 
If it specifically mentions keeping instances physically close together, consider a cluster placement group.
###### Compute and Networking
EC2 Enhanced Network 
This gives EC2 instances higher network performance with lower CPU overhead and lower latency. Uses SR-IOV tech to let VM access network hardware more efficiently than traditional virtualized networking
	Ways to get this up:
		1. Elastic Network Adapter (ENA): up to 100Gbps
		2\. Intel 825599VF: up to 10Gbps -- Legacy
Elastic Fabric Adapter
	this is an improved ENA for HPC, only works for Linux
	It is great when you have inter-node communications, tightly coupled workloads
	This is because it leverages a Message Passing Interface (MPI) standard which bypasses the underlying Linux OS to provide low-latency, reliable transport for the EC2 network

- ENI = the network interface attached to your EC2 instance.(ntw properties such as IP, mac, subnet, security group)
- ENA = a high-performance network adapter for ordinary IP networking.
	- this provides enhanced IP networking (TCP/IP, UDP, ...) and high network throughput
- EFA = a specialised adapter for extremely fast communication between compute nodes, especially for HPC and distributed AI/ML.

ENA - Elastic Network Adapter: This provides enhanced networking for supported EC2 instances. Designed to delivery high network bandwidth & packet-processing performance for many workloads.

Think ENA as the high performance network interface used by ordinary cloud applications as well as for many demanding applications

EFA - Elastic Fabric Adapter: Is a network interface designed for applications that need very fast communication btw compute nodes. Adds support for OS-bypass communication through HPC communication libraries 

This is important because ideally, a ntw comm can involve significant operating-system network overhead. In a tightly coupled simulation that overhead can become a bottlenect, EFA helps with that.
### Automation and Orchestration
AWS Batch:
This supports multi-node parallel jobs, enabling you run single jobs that span multiple EC2 instances
Easily schedule jobs and launch EC2 instances 

### AWS ParallelCluster
This is an open-source cluster management tool to deploy High Performance computing clusters on AWS.

It helps provision the infra required to run computational jobs across multiple EC2 instances, rather than building & configuring every server manually

ParallelCluster manages the HPC cluster; it in itself isn't a compute engine / fully managed batch job execution service

You configure this using Text files to automate the creation of VPC, subnet, ...
You can use an EFA on this also for improved performance

---
# EC2 Instance High Availability

