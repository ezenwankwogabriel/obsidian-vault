Elastic Compute Cloud => Infrastructure as a Service
 - Can rent virtual machines (ec2)
 - store data on virtual drives (ebs)
 - distribute load across machines (elb)
 - scaling services using an auto-scaling group (ASG)

### Sizing and config options
> OS: Linux, Windows, or Mac OS
> How much compute power & core (cpu)
> how much RAM
> how much storage space
> 	Network-attached (ebs & efs)
> 	hardware (ec2 instance store) 
> network card: speed of the card, Public IP
> firewall rules
> bootstrap script: *EC2 user data* (runs with root user)

Bootstrapping is launching commands when the machine starts. For tasks you which to automate

*On restarting EC2 instances, the public IP changes, but the private IP stays same*

### EC2 Types
- General Purpose: web servers or code repositories. balance btw compute, memory and networking
- Compute optimized: compute intensive tasks. batch processing, media transcoding, High performance web servers, high performance computing (hpc), scientific modeling & machine learning, gaming servers [all compute intensive]
- Memory optimized: fast performance for large processing data in memory. dbs, web cache, in memory db, processing big unstructured data
- Storage optimized: for accessing lots of data set on local storage. dbs, cache, data warehousing, distributed file systems
- Accelerated computing
- HPC optimized

### Firewalls (security groups)
control how traffic is allowed into or out of EC2 instances
- they contain only *allow* rules
- the rules can be referenced by IP or by security group
By default all inbound traffic is blocked & all outbound traffic is allowed


## Ports to know
22 = SSH (Secure Shell) - log into a linux instance
21 = FTP (File transfer protocol) - upload file to file share
22 = SFTP Secure file transfer protocol (file upload using SSH)
80 = HTTP Hyper text transfer protocol (access unsecure websites)
443 = HTTPS (access secured website)
3389 = RDP Remote desktop Protocol (log into a windows instance securely (22 for linux))


NB: When you see a timeout accessing an ec2 instance, always think of the security group and make sure they are correct
### SSH Summary Table
ssh -i <link-to-.pem-file> ec2-user@public-ip
give permissions: `chmod 0400 <link-to.pen-file>


### SSH using EC2 Instance
You can ssh using the line above
but do not use `aws configure` inside the ec2 instance
This would grant all users access to the ec2 instance and retrieve the values from the instance
Instead, use IAM roles. give the permission to iAM roles, then continue with action in the instance to view/list users

# EC2 instances purchasing options
- On demand EC2 instances - short workload, pay by second, predictable pricing (default i've used)
	- Pay for what you use
		- Linux / Windows > per second, after the first minute billing
		- other OS > billing per hour
	- Highest cost but no upfront payment
	- No long term commitment
	- Recommended for short-term, un-interrupted workload, where you can't predict how systems will behave
- Reserved (1 & 3 years)
	- reserved instances - long workload
	- convertible reserved instances - flexible instances for long workload
	- 72% discount compared to on-demand
	- you get to reserve specific instance attribute (instance type, region, tenancy, os)
	- reservation period
		- no upfront
		- partial upfront
		- all upfront
	- Recommended for steady-state apps (like db)
- Savings Plan (1 & 3 year): commit to an amount of usage, long workload
	- Get discount based on long term usage
	- commit to a certain type of usage ($10/hour for 1-3 years)
	- usage beyond ec2 savings plan is billed at on-demand price
	- locked to a specific instance family. Flexible OS
- Spot instances: short workload, cheap, can lose instances
	- Can get up to 99% compared to ondemand
	- can loose these instances if your max price is less than the current spot price
	- most cost efficient instance in AWS
	- best if you have a workload that's resilient to failure
	-  (batch job, data analytics, image processing, ...)
	- not suited for critical jobs or databases
- Dedicated Hosts: book an entire physical server, control instance placements
	- a physical server with ec2 instance capacity fully dedicated
	- addresses compliance requirement and use existing server bound software licenses (per-socket, per-core, pe -- vm software licenses)
	- the most expensive AWS option
	- could be on-demand or reserved (1/3 years)
	- for complicated licensing model (bring your own license (byol)) or companies with strong regularory / compliance needs
- Dedicated Instances: no other customer will share your hardware
	- Instances run on hardware thats dedicated to you, but you may share the hardwared with other instances in same account
- Capacity reservations: reserve capacity in specific AZ for any duration
	- you always have access to EC2 capacity when needed
	- no time commitment (create/cancel anytime), no billing discounts
	- you're charged at on-demand rate whether you run instances or not
	- suitable for short-term uniterrupted workflow for a particular AZ

## EC2 purchasing options — quick recognition guide

| If the question says...                                  | Think...                  |
| -------------------------------------------------------- | ------------------------- |
| No commitment, pay as you use                            | On-Demand                 |
| Spare capacity, cheapest, can tolerate interruption      | Spot Instances            |
| Predictable workload, 1 or 3-year commitment             | Reserved Instances        |
| Discount across EC2, Fargate and Lambda                  | Compute Savings Plan      |
| Specific EC2 family and Region, flexible instance size   | EC2 Instance Savings Plan |
| Highest RI discount, fixed configuration                 | Standard RI               |
| Change instance family or configuration after commitment | Convertible RI            |
| Guaranteed EC2 capacity in a specific AZ                 | Capacity Reservation      |
| Entire physical server, socket/core licensing            | Dedicated Host            |
| Hardware dedicated to your account                       | Dedicated Instance        |

Dedicated Host => You rent a physical server
Dedicated Instance => Your EC2 instance runs on hardware dedicated to you, but aws manages the physical server
![[Pasted image 20261014130201.png]]

| Feature                                                         | Dedicated Host                    | Dedicated Instance                                 |
| --------------------------------------------------------------- | --------------------------------- | -------------------------------------------------- |
| Physical server                                                 | Dedicated to you                  | Hardware dedicated to you                          |
| Control over placement                                          | Yes                               | No                                                 |
| Visibility into physical sockets and cores                      | Yes                               | No                                                 |
| Bring your own software licences tied to physical sockets/cores | Better suited                     | Generally not suited to host-level licensing needs |
| Management                                                      | More control, more responsibility | Simpler; AWS manages placement                     |
| Cost model                                                      | Host-based billing                | Instance-based billing                             |
Both provide hardware isolation from instances belonging to other customers. A Dedicated Host does not mean you manage the data centre or the physical machine yourself; AWS still operates the infrastructure.

Imagine you own a transport company.

- Dedicated Host: You rent an entire bus. You decide how to arrange the seats and which passengers (EC2 instances) go where.
    
- Dedicated Instance: You book a private ride. You get exclusive use of your ride, but the transport company decides which vehicle to assign.

HOST = Hardware control
Think physical server, socket/core visibility, instance placement, and host-based licensing.

INSTANCE = Instance isolation
Think dedicated hardware without managing physical host placement.

Final exam tip: If a question mentions physical sockets, cores, host-level licences, or control over instance placement, think Dedicated Host. If it simply mentions hardware isolation without host-level control, think Dedicated Instance.


### Spot instance requests
- can get up to a discount of 90% compared to on-demand
- define max-spot price and get the instance while current spot price < max
### How to terminate spot instances
the flow from the start is the sport request which is:
> max price, desired no of instances > launch specification > request type (one-type | persistent) > valid from | valid until
To cancel a spot instance it needs to be in the open state, or active or disabled state.
Canceling does not stop instances already running.
To stop, you terminate the associated spot instances. because if you first stop, it goes back to the spot request and if it is a persistent request type, gets respawn

### Spot Fleets 
A set of spot instances and an optional on-demand instance
It allows us to automatically request spot instances with the lowest price
It will try to meet the target capacity with price constraints
Strategies to allocate spot instances:
- lowest price (from a pool with lowest price)
- diversified: distributed across all pool(great for availability and long workload)
- capacity optimized: pool with the optimal capacity for the no of instances
- price capacity optmized (recommended): pool with the highest capacity available, then select the pool with the lowest price

## Spot instance (ai - defined)
This is a request to use AWS's spare computing capacity at a discounted price
As AWS has physical servers that aren't always fully used, instead of letting that capacity sit idle, AWS allows you to use it cheaply.
The catch here is that AWS can interrupt your spot instance when it needs that capacity back
AWS provides a 2 min interruption notice before reclaiming a spot instance

If a question asks to reduce EC2 cost for a batch processing job that can tolerate interruptions, spot instances are usually the best answer

If the app cannot tolerate interruptions, choose a moer suitable capacity option / design it to recover automatically

#### Spot Fleet
This is asking AWS to find and run a group of cheap servers for you automatically
Instead of requesting just one Spot Instance, you ask AWS for a collection of instances meeting your computing need/budget

Imagine you're processing 1,000 videos and need 10 servers to do it.
> You could request 10 Spot Instances individually, but availability and prices may vary.
> With a Spot Fleet, you tell AWS what you need, and it selects suitable EC2 instances from available capacity pools.
### Placement groups
Strategies used to control over the EC2 instance placement strategy so you don't get direct influence but let aws know how you'd like the EC2 to be placed compared to another
- Cluster: instances grouped in a low-latency group in a single AZ
	- All EC2 instances are in same availability zone. Gives you great networking
	- if the AZ fails, all instances fail at same time
	- use-cases: big data jobs, application with low latency and high network throughput
- Spread: spreads instances across the hardware (max 7 instances per group per AZ)
	- we want to minimize failure risks
	- ec2 instances in separate hardware, grouped in an AZ, and repeated for different AZ
	- Limited to 7 instances per AZ per placement group
- Partition: similar to spread, but across many diff partitions
	- grouped in partitions
	- up to 7 partition per AZ
	- can span across multiple AZ in same region
	- big data apps like HDFS, cassandra
- Precision time Placement group
	- instances are placed on hardware with direct access to high-precision time sources
![[Pasted image 20261014161437.png]]


---

Overall explanation

**Amazon CloudWatch** has Amazon EC2 Metrics available for monitoring. CPU Utilization identifies the processing power required to run an application upon a selected instance. `Network Utilization` identifies the volume of incoming and outgoing network traffic to a single instance. The `Disk Read` metric is used to determine the volume of data the application reads from the hard disk of the instance. This can be used to determine the speed of the application. However, there are certain metrics that are not readily available in CloudWatch which can be collected by setting up a custom metric.

![EC2 CloudWatch Monitoring](https://media.tutorialsdojo.com/public/TD-EC2CloudWatchMonitoring-12June2025.png)

You need to prepare a custom metric using CloudWatch Monitoring Scripts which is written in Perl. You can also install CloudWatch Agent to collect more system-level metrics from Amazon EC2 instances. Here's the list of some of the custom metrics that you can set up:

- `**Memory utilization**`

- `Disk swap utilization`

- `Disk space utilization`

- `Page file utilization`

- `Log collection`

---
## EC2 Pricing Models / AWS Cost Optimization
