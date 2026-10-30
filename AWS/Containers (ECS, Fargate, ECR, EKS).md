# ECS (Elastic Container Service)
This is an AWS service for running and managing containers

Given a Node.js api using Docker, instead of manually starting containers on servers, ECS manages where your container runs, how many copies run, and how they are replaced when they fail.

When  you launch a container on aws, you're launching an ECS Task on ECS Cluster

# Concepts
1\. Cluster: A logical grouping where ECS manages your workload using EC2 capacity or Fargate

2\. Task Definition: The blueprint for running your application: image, CPU, memory, ...

3\. Task: A running instance of a task definition. Can contain one or more closely related containers

4\. Service: keeps the desired number of task instances running & replaces failed task. Can be integrated with a load balancer

## Launch Types

1\. EC2 launch type (Fargate & Self-managed instances)
ECS places your containers on EC2 instances that you provide and manage
- You choose instance type, and manage the EC2 capacity
- You benefit from Reserved Instances/Savings Plans for EC2 usage
- Suitable when you need host-level control, specialized config / predictable workloads

2\. Fargate only launch type
AWS manages the underlying compute
- ECS runs tasks on "serverless" compute without you provisioning or maintaining EC2 instances.
- Suitable when you want to focus on containers rather than servers

3\. Fargate and managed instances
ECS will manage patching and scaling on your behalf while giving you configurability about the types of instances. Great for advanced workloads

## ECS Anywhere
This lets you register & manage supported external servers, such as on-premise/virtual machines, as part of ECS environment.

This is an extension of ECS to external infrastructure (likely outside of AWS, existing on-premises server)

## Launch types(compute) vs capacity providers(capacity)

Launch type specify the compute options used to run a task (EC2 / Fargate)

Capacity provider connects ECS to a capacity source and helps manage how capacity is supplied and scaled. examples include
	- EC2 auto scaling group capacity provider
	- Fargate capacity providers. types are:
		Fargate: for standard serverless container compute
		Fargate spot: tasks can be interrupted when AWS needs capacity. uses spare AWS capacity at a discount. (think spot storage)

## IAM Roles for ECS

EC2 Instance Profile (for EC2 launch type only) (used by the EC2 host)
	The role is attached to the EC2 instance through an instance profile, gives the host permissions needed to operated as an ECS container instance
	
	Examples:
		Register with and comm with an ECS
		Retrieve information / perform action required by host software
		Access AWS services that the host itself needs

ECS Task Roles (app code inside the task)
	Provides AWS credentials to application code running inside the ECS task
	
	Examples:
		Upload to S3
		Read messages from SQS
		Publish events to EventBridge
		Read permitted secrets from secret manager

ECS task execution role
	Pull a private ECR image; send container logs to cloudwatch
	used by ECS/Fargate infrastructure during taask startup/operation

The EC2 instance hosts the container, but your app should use its task roles to assess S3. 

## Data Volumes [EFS] (for persistance)
1\. We use EFS as it is a Network File System (NFS)
Mount EFS file systems onto ECS tasks
Then tasks running in any AZ will share same data in the EFS file system

Altimate combo is: Fargate + EFS = Serverless

use case: persistent multi-AZ shared storage for your containers

2\. *Also you could mount **S3 file** systems onto ECS tasks. This works for Fargate & ECS Managed Instances (EC2 launch type is not supported)*
*this comes handy when you need shared file access to data already in S3*

## ECS Service Auto Scaling
ECS increases/descreases desired number of ECS tasks
Uses AWS Application Auto Scaling for this and can be benched on:
	ECS Service Average CPU utilization
	ECS Service Average Memory utilization (ram)
	ALB request count per target (metric coming from the ALB)

### Policies for Auto Scaling

Target Tracking –– maintain a metric near a target, such as average CPU utilization around 50% (a cloudwatch metric)
example: maintain CPU at 50%

Step scaling –– Add / remove capacity in step based on CloudWatch alarm threshold
example: add two tasks when CPU exceeds 70%

Scheduled scaling –– Adjust desired capacity at predictable time
example: scale up every weekend at 8am


---
Imagine you're running Node.js using EC2 launch type

You have two EC2 instances, each can run four of your containers

Then traffic increases, Your service needs eight tasks, but your two EC2 instance only have enough capacity for four

	ECS Service auto scaling
		How many applications should be running?
		i.e: it increases the desired task count from four to eight when CPU utilization rises. It doens't guarantee that the underlying EC2 instances have room for those tasks
	
	For EC2 Instance:
		EC2 Auto Scaling Group (ASG)
			How many EC2 instances should exist
			i.e: it increases the instance count from two to four
			it launches/terminates EC2 instances according to its scaling config
		
		ECS capacity provider
			How should ECS obtain and manage the compute capacity needed for task
			EC2 capacity provider can link ECS to an EC2 Auto Scaling group. With managed scaling enabled, ECS can use capacity demand to help scale that group up or down

Think of the capacity provider as the bridge between ECS and the EC2 Auto Scaling group.
It connects ECS's capacity requirements to that mechanism
![[Pasted image 20261020091939.png]]

	For Fargate:
With Fargate, there is no EC2 Auto Scaling group to be managed. we have Fargate Capacity Provider

You can still configure ECS service auto scaling to change the task count, AWS manages the compute capacity for Fargate tasks


# ECR (Elastic Container Registry)
This is used to store and manage Docker images on AWS (similar to dockerhub)
Can store images privately or publicly (https://gallery.ecr.aws)
It supports image vulnerability scanning, versioning, image tags, image lifecycle, ...


# EKS (Elastic Kubernetes Service) pods
A way to launch managed Kubernetes. cluster on AWS
An alternative to ECS, similar goal but different API
ECS isn't opensource unlike Kubernetes
EKS supports EC2 and Fargate also (just like ECS)
It is cloud-agnostic (so if you wish to use different clouds, use this)

Node Types:
	Managed Node Groups
		Creates and manages node (ec2 instances) for you
		supports on-demand and spot instances
	
	Self-Managed Nodes
		Create nodes yourself, and register to EKS cluster
		Can use prebuilt AMI
		supports on-demand and spot instances
	
	AWsFargate
			No maintenance required; no nodes managed.
			just run containers on top of EKS

Data Volumes in EKS:
	You'd need to specify StorageClass manifest on your EKS cluster
	Leverages a Container Storage Interface compliant driver
