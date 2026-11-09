AWS Fargate is a serverless compute engine for running containers without managing the underlying servers.

You provide your container image, configure CPU, memory, networking, and permissions, and AWS manages the underlying compute infrastructure.

The two main services you need to associate with Fargate are:

- Amazon ECS: A container orchestration service that manages tasks and services.
- Amazon EKS: A managed Kubernetes service that runs Kubernetes workloads.

Fargate can provide the compute capacity for tasks in ECS or pods in EKS.

## 1. Start with a practical example

Suppose you build a Node.js API using Docker.

You already have a Docker image:

`my-payment-api:v1`

You want to run it in production. Traditionally, you might launch EC2 instances, install container software, manage capacity, patch the operating system, and schedule your containers onto those instances.

With ECS on Fargate, the workflow is simpler:

Docker image

Your Node.js application

Amazon ECS

Task definition specifies the image, CPU, memory, and configuration

AWS Fargate

Runs your container without you managing EC2 hosts

Running application

Receives requests through your configured networking and load balancer

Notice the division of responsibility: ECS orchestrates the workload; Fargate provides the serverless compute to run it.

## 2. ECS on Fargate vs. ECS on EC2

This is one of the most important comparisons for the exam.

### ECS using Fargate

Amazon ECS

Schedules and manages tasks

Fargate-managed compute

No EC2 instances for you to manage

### ECS using EC2

Amazon ECS

Schedules and manages tasks

Your EC2 container instances

You manage instance capacity, operating systems, and host maintenance

| Consideration                                        | Fargate                          | EC2 launch type                                  |
| ---------------------------------------------------- | -------------------------------- | ------------------------------------------------ |
| Manage EC2 hosts?                                    | No                               | Yes                                              |
| Choose underlying instance type?                     | No                               | Yes                                              |
| Control the host OS and install host-level software? | Very limited                     | Yes                                              |
| Pay for allocated compute while tasks run?           | Yes                              | Pay for EC2 capacity, even if underutilized      |
| Can multiple tasks share an EC2 host?                | You don't manage host placement  | Yes                                              |
| Best fit                                             | Reduce infrastructure management | More host-level control or specialized workloads |

Fargate can simplify operations, but it is not always cheaper. EC2 can be more cost-effective for steady, high utilization when you manage capacity well.

## 3. How scaling works

Suppose your application normally handles 100 requests per second but occasionally jumps to 5,000.

With ECS on Fargate, you can configure ECS Service Auto Scaling to increase the number of running tasks when a metric such as CPU utilization or request count rises.

For example:

- Normal traffic: 2 tasks.
- Higher traffic: 10 tasks.
- Traffic falls: scale back to 2 tasks.

Fargate supplies compute for the tasks ECS launches. You still need to configure scaling policies, networking, load balancing, and application capacity appropriately. Fargate does not automatically scale every application without configuration.

## 4. Networking and IAM: don't miss these

Networking

Fargate tasks in ECS use the `awsvpc` network mode. Each task receives its own elastic network interface and private IP address within your VPC subnet. Security groups can be assigned at the task level.

A task in a private subnet can access private resources such as RDS if routing and security rules permit it. Outbound internet access may require a NAT Gateway, while supported AWS services can often be accessed through VPC endpoints.

IAM roles

For ECS on Fargate, distinguish:
- Task execution role: Used by the ECS/Fargate infrastructure for tasks such as pulling images from ECR and sending logs to CloudWatch.
- Task role: Provides AWS permissions to the application code inside your container, such as permission to read an S3 object.

This is a common exam trap: the application should not receive the execution role's permissions merely because it is running in the same task.

## 5. Fargate limitations worth remembering

Fargate isn't a general-purpose EC2 replacement.

- You cannot manage the underlying host operating system.

- Certain ECS task-definition options are unsupported, including privileged containers and GPU configuration for Fargate tasks.

- CPU and memory must use supported combinations in the task definition.

- Persistent application data should generally live in an appropriate external storage service rather than relying on ephemeral task storage.

- EKS Fargate requires Fargate profiles to determine which Pods run on Fargate; not every Kubernetes workload or feature is supported.

Memory trick: ECS decides how containers run; Fargate provides the compute without you managing servers. Choose EC2 instead when you need more control over the underlying hosts.


---

ECS and EKS are container orchestration services.
Fargate is a serverless compute engine for running computers.

They aren't competing products

![[Pasted image 20261109131536.png]]

### Amazon ECS - Elastic Container Service
This is AWS's own container orchestration service.
Helps deploy, manage, scale, and maintain Docker containers

- You package app into Docker image
- You push the image to ECR (container registry)
- Create an ECS Cluster:
	- ex: production-cluster
- Create a Task Definition (docker image, CPU and memory, env, ports, roles, ...)
```
  Task Definition: ecommerce-api

	Container:
	  Image: ecommerce-api:v1
	  CPU: 0.5 vCPU
	  Memory: 1 GB
	  Port: 3000
```
- Create an ECS Service (replaced on crash)
```
Desired tasks: 3

Task 1: Running
Task 2: Running
Task 3: Running
```
- Configure a load balancer: can attach ALB to this
```
                Internet
                   |
                   v
                  ALB
                   |
          +--------+--------+
          |        |        |
        Task 1   Task 2   Task 3
          |        |        |
        Node.js  Node.js  Node.js
```

You can run containers in EC2 or by using Fargate

Fargate doesn't mean you app scales automatically. You still configure the scaling policy, typically using an Application Auto Scaling. And configure ALB if you need load balancing.
AWS only manages the underlying fargate compute

### EKS - Elastic Kubernetes Service
AWS's managed kubernetes service. Kubernetes is an open source platform.
Use this if your:
- org already uses kubernetes
- engineering team has kubernetes expertise