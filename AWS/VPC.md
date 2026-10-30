# CIDR (Classless Inter-Domain Routing)
This is a method for allocation IP addresses
In AWS, CIDR notation is how you express an address range

use-case: in Security Group rules and AWS networking in general

It has two parts, the Base IP, and the subnet mask(example: /0, /24, /32)
/8: 255.0.0.0
/16: 255.255.0.0
/24: 255.255.255.0
/31: 255.255.255.255 => allows for 1 IP 2^2 => 192.168.0.0 -> 192.168.0.1
/32: 255.255.255.255 => allows for 1 IP (2º)

Max subnet is 32, so any left would be considered on the power of two for the IP range

For example: 192.168.1.0/24 => 2^8
Total addresses = `2^32-prefix length`

/24 tells you that the first 24 bits identify the network portion of the range. The remaining 8 bits are available for addresses within that range

### Private IP vs Public IP
Private IPs are used within private networks and not reachable from the public internet
There are three private IPV4 ranges you should memorize:
>10.0.0.0/8           ex: 10.0.1.15 => for big networks
>172.16.0.0/12.     ex: 172.20.4.10 => AWS default VPC falls in this range
>192.168.0.0/16.   ex: 192.168.1.0 => for home networks usage

Public IP are used for comm over the internet, provided the network routing and security configuration allow it

NB: Public IP doesn't necessarily mean accessible from the Internet (ex in an EC2 instance).

---
# VPC walkthrough
All new AWS accounts have a default VPC
New EC2 instances are launched into the default VPC if no subnet is specified
Default VPC has internet connectivity and all ec2 instances inside it have public IPv4 addresses

You can have multiple VPC in an AWS region (max. 5 per region - soft limit)

Max CIDR (subnets) per VPC is 5, for each CIDR:
	Min size is /28 (16 IP address)
	Max size is /16 (65536 IP addresses)

Your VPC CIDR should NOT overlap with your other networks. They have to be unique IP ranges

---
# Subnets
This is a sub-range of IP addresses within your VPC as we said earlier

AWS reserves 5 IP addresses (first 4 and last) in each subnet

The cannot be assigned to an EC2 instance

Example: if CIDR/subnet block 10.0.0/24 (256 IP addresses), the reserved IPs are
>10.0.0.0 - Network address
>10.0.0.1 - reserved for the VPC router
>10.0.0.2 - reserved for mapping to Amazon-provided DNS
>10.0.0.3 - reserved by AWS for future use
>10.0.0.255 - Network Broadcast Address. AWS doesn't support broadcast in a VPC, therefore the address is reserved

Always remember this for the exam: 5 IP addresses are reserved, so when allocating for a subnet, take that to account, to deduct the default allocations

---
# Internet Gateway (IGW)
This allows resources (like EC2 instances) in a VPC connect to the Internet
Must be created separately from a VPC
One VPC can only be attached to one IGW and vice-versa

IGW on their own do not allow Internet Access, **Route tables** must be updated to make it work.

---
# Bastion Hosts (a regular EC2 on public subnet)
This is a server that you use as a secure entry point to access instances in a private subnet.

Administrator ->getting access to-> private server

Imagine your VPC contains two EC2 instances:
- a web server in a public subnet (has IGW in it's Routing table)
- a database or application server in a private subnet (no IGW)

You want to SSH into the private app server to troubleshoot, but don't want to give it a public IP or expose SSH directly to the internet. Bastion Host solves this problem

Connection happens in two stages
- You SSH from your laptop into the bastion host (public subnet to the internet)
- From the bastion, you SSH into the private instance using the app servers private IP address

### Security group configuration

Bastion security group settings

| Inbound  | SSH, TCP port 22, from your trusted public IP only              |
| -------- | --------------------------------------------------------------- |
| Outbound | Allow SSH to the private instance, subject to your egress rules |

Private instance security group settings

|   |   |
|---|---|
|Inbound|SSH, TCP port 22, with the bastion security group as the source|
|Public IP|Not required|

#### Is a bastion host still the best option?
Not always. AWS System Manager Session Manager is preferable when supported and configured correctly.
With session manager, you can access managed EC2 instances without opening inbound SSH ports or maintaining a public bastion host.
You'd need the appropriate SSM agent, IAM permission and connectivity to the SSM endpoints.

---
# NAT Instances (outdated, now Nat Gateway)
NAT: Network Address Translation

This allows EC2 instances in private subnets connect to the internet

private server -> internet

Must be launched in a public subnet and have an Elastic IP attached to it
Also must disable EC2 settings: **Source/destination check**

Route tables has to be configured to route traffic from the private subnet to the NAT instance

NAT is also an EC2 instance, on a public subnet, that you route in the route table to be the Internet Gateway.

---
# Route Table
Think this as a GPS for network traffic. Tells a subnet where to send packets based on their destination IP address

When an EC2 instance wants to comm with another server or the internet, the route table helps determine the next place traffic should go
### Example route table

|   |   |
|---|---|
|Destination|Target|
|`10.0.0.0/16`|Local|
|`0.0.0.0/0`|Internet Gateway (IGW)|

This example route table is associated with the public subnet.

Here's how to read those routes:

- `10.0.0.0/16 → local`: traffic destined for addresses within the VPC uses the VPC's internal network routing.
- `0.0.0.0/0 → IGW`: traffic destined for any IPv4 address not matched by a more specific route is directed to the Internet Gateway.

The `0.0.0.0/0` route is called the default route. It means “any IPv4 destination.”

---
# NAT Gateway
These are AWS managed NAT, higher bandwidth, high availability and no administration
Created for a specific AZ, and uses an Elastic IP

It cannot be on same subnet as that used by EC2 instance and requires an internet gateway

Flow:
Private subnet => NatGW => IGW

5 Gbps of bandwidth with autoscaling up to 100Gbps
No security groups to required

### Regional NAT Gateway (RNAT)
This is a highly available NAT Gateway associated with VPC
Eliminates the need to deploy a NAT Gateway per AZ (Shared across AZ)
You also don't need to create Public Subnets in your VPC to host RNAT
You can link your private subnet directly to the RNAT

---
# NACL & Security Group (subnet security)

Network ACL (Access Control List):
This is a security layer controlling which network traffic is allowed into/out of an AWS subnet

Route table: where should this traffic go?
NACL: Is this traffic allowed to enter / leave the subnet (stateless)
Security group: Is this traffic allowed to reach / leave the individual resource, such as an EC2 instance (stateful)

`**Internet ––> NACL ––> Subnet ––> EC2 (Protected by a security group)**`

Being stateless means that even as incoming requests are allowed, doesn't auto allow the response traffic. You must configure rules for both directions
```
Inbound NACL rule
Allow TCP port 443 from the permitted client IP addresses
---
Outbound NACL rule
Allow the response traffic to the client's ephemeral destination port range
```
If you allow inbound HTTPS but block the necessary outbound response traffic, connection can fail even though the inbound looks correct

A default NACL is assigned to VPCs on creation. When adding inbound/outbound rules, they shouldn't have conflicting subnets for same NACL
You have one NACL per subnet, and each subnet is assigned a default NACL

Defining NACL Rules:
	- Rules have a number (1-32766). 
	- Priority to rules is given to lower numbers
	- The first rule drives the decision
	- if you have # 100 Allow 10.0.0.1/32 and # 200 DENY 10.0.0.10/32, the IP address will be allowed (ALLOWED). This is because 100 has higher precedence over 200
	- The last rule is an asterisk (\*), and denies a request in case of no rule match
	- AWS recommends adding rules by increments of 100

Suppose your NACL contains these inbound rules:

| Rule number | Source            | Port | Action |
| ----------- | ----------------- | ---- | ------ |
| 100         | `203.0.113.10/32` | All  | DENY   |
| 110         | `0.0.0.0/0`       | 443  | ALLOW  |
| `*`         | All               | All  | DENY   |

If traffic comes from `203.0.113.10`, rule 100 matches first, so the traffic is denied—even though rule 110 allows HTTPS from everyone else.

The NACL processes rules in ascending numerical order and stops at the first matching rule. The `*` rule is the final implicit deny if no numbered rule matches.

#### Default NACL vs Custom NACL 
Default NACL allows everything inbound and outbound IPV4 traffic (created once you create a VPC)
Custom NACL: Initially denies all inbound and outbound traffic. You must add the required allow rules

If the exam says there's a default NACL and it's associated with subnet, know that i'd allow every connection to it.

---
# Ephemeral Ports
This is a temporary port that a device uses for a network connection (usually on client side)

When your computer connects to a website, it needs a port to receive the websites response. Your computer temporarily picks an available port for that connection. That is the ephemeral port

Your NACL must allow both sides of the conversation:

| Direction at the server subnet | Destination port             | Example rule                       |
| ------------------------------ | ---------------------------- | ---------------------------------- |
| Inbound request                | `443`                        | Allow HTTPS requests               |
| Outbound response              | Ephemeral port, e.g. `53000` | Allow return traffic to the client |

This is the key exam point: NACLs are stateless, so you must allow the response traffic separately.

For many AWS exam scenarios, the expected ephemeral port range is `1024–65535`. This is a broad range, not a universal range used by every operating system.

#### Security Group vs NACL
SG operates at an instance level while NACL operates at a subnet level
SG supports "ALLOW" rules only. NACL supports "ALLOW" and "DENY" rules
SG is stateful (return traffic is auto allowed). NACL needs explicit return rule
SG applies to EC2 instance. NACL applies to subnets (all EC2 instances in it)

---
# VPC Peering
This lets two VPCs communicate using each others private IP addresses, as if their network were connected privately without going through the public internet (say where vpc 1 is the app server and vpc 2 is the database server)

think of two separate networks in AWS
- VPC A hosts your app
- VPC B hosts your database
By default, they are isolated. They can't comm with each other. VPC peering provides a direct network connection for comm without going over the public internet

After creating the peering connection:
- Update VPC A's route table to add support for the peering endpoint
- Update VPC B's route table '' ''
- configure security controls: db security group must permit the intended app traffic. NACL must also permit the required inbound & outbound traffic (this is default to allow all)

---
# VPC Endpoint
This lets resources in your VPC access supported AWS services privately, without needing to send that traffic through the public internet

Think an EC2 instance in a private subnet (no IGW) that needs to download a file from S3.
>Without a VPC endpoint, you'd route traffic through a NAT Gateway (manually configured / the managed Gateway from Amazon) to reach S3 public endpoint (before commands like `aws s3 ...`) would work.
>With an appropriate VPC endpoint, the instance can reach S3 privately instead

Every AWS service is publicly exposed (has a public URL)

VPC endpoint (by AWS PrivateLInk) allows you to connect to AWS services using a **private network** instead of using the public internet

They remove the need for IGW, NATGW when accessing aws services

Types:
Interface Endpoints: powered by Privatelink
	Provisions an ENI with a private IP address in each selected subnet
	- uses security groups to control traffic to the endpoint
	- costs per hour per GB of data processed
	- commonly uses private DNS to route service request to endpoint
	supports most AWS services
	services include: 
		- AWS Service Manager 
		- CloudWatch 
		- Secrets Manager
		- KMS
		- ECR
		- SNS, SQS
Gateway Endpoints (for S3 or DynamoDB)
	- Uses route tables to direct traffic to Amazon S3 or DynamoDB through AWS managed prefix list.
	- Supports both S3 and DynamoDB only
	- No endpoint network interface (ENI) is created
	- No endpoint security group is created
	This is freeeeeeeeee!!!!!!!!

The one time an Interface Endpoint is preferred to Gateway Endpoint is when access is required from an ***on-premises***, ***a different VPC***, or ***a different region*** (requiring an actual interface)

##### How a gateway endpoint actually works
Suppose you have:
- VPC: 10.0.0.0/16
- Private subnet: 10.0.1.0/24
- EC2 instance: 10.0.1.10
- Destination: Amazon S3
You create an S3 gateway endpoint and associate it with the private subnet's route table.

Before creating the endpoint, the route table might look like this:
Destination	Target
10.0.0.0/16	local
0.0.0.0/0	nat-123

After creating the gateway endpoint, AWS adds a route similar to this:
Destination	Target
10.0.0.0/16	local
S3 managed prefix list	vpce-123
0.0.0.0/0	nat-123

##### How an interface endpoint works
Now imagine the EC2 instance needs to retrieve a secret from AWS Secrets Manager.

You create an interface endpoint for Secrets Manager and select a subnet. AWS creates an endpoint ENI with a private IP address in that subnet.

With private DNS enabled, the standard Secrets Manager hostname can resolve to the endpoint's private IP inside the VPC. Your application can usually continue using the normal AWS SDK without hardcoding the endpoint's IP address.

##### Creating a VPC Endpoint
```
- Go to VPC > Endpoint
- Create Endpoint
- Select Category (AWS Services, EC2 Instance C.E, ...)
- Select services for the endpoint and the type (S3, apigateway, ...)
- Select VPC > Select Route Tables (housing subnets) for the endpoint
  (if the source IPs used public IPs they'd be dropped and set to use private IPs)
- Create Endpoint (this auto links to the selected Route table)
```

___
# VPC Flow Logs (IP Logs [from security group / NACL])
Captures record info about IP traffic going to and from network interfaces in your VPC
This is like a CCTV logbook for network traffic

It could be VPC level, Subnet, or Elastic Network Interface (ENI) Flow logs

It has 2 actions:
ACCEPT: The traffic was allowed by relevant VPC network controls
REJECT: The traffic was rejected by a security group or network ACL

A reject doesn't mean a security group/network ACL/route table caused it. You'd always need additional investigation to identify the cause
Flow logs doesn't capture every kind of traffic 
___
# Site to Site VPN
This connects your on-premises network to an AWS VPC over an IPsec VPN connection

Think of an on-premises company, who moved some applications to AWS but still needs its office computers to communicate with those applications. Site to Site VPN creates an encrypted connection btw the two network

Requires two gateways:

1\. Customer Gateway (CGW) [Customer Side]
	This represents your on-premises VPN device in AWS. the actual device could be a physical router / a software VPN applicance

2\. Virtual Private Gateway (VGW) [AWS VPC side]
	An AWS managed VPN Endpoint that you attach to a VPC. It handles the aWS side of the VPN connection

*NB*: The connection wouldn't work if you don't set up Route Propagation for the VGW in the route table that's associated with your subnets

Also: If you need to ping your EC2 instances from on-premises to AWS, make sure you add the ICMP protocol on the inbound of your security groups (key (you experienced this previously))

![[Pasted image 20261023165032.png]]

Now picture you have a VGW housing different instances in different AZ, and the customer has multiple Gateways, how do you securely handle communication?
This is where **AWS VPN CloudHub** comes in

*CloudHub* provides secure communication between multiple sites using multiple VPN connections.
___
# Direct Connect (DX)
This creates a dedicated network connection between your on-premises network and AWS

- Site-to-Site VPN: your traffic travels over the public internet inside encrypted IPsec tunnels
- Direct Connect: your traffic travels over a dedicated network connection provided through an AWS Direct Connect location and a network provider

If you picture VPN as a secure tunnel through a public road, Direct Connect is like reserving a dedicated road connecting your office network to AWS

![[Pasted image 20261023182451.png]]

use-cases: 
	Increased bandwidth througput - working with large data sets - lower costs
	More consistent network experience

#### Connection Types
1\. Dedicated Connections (1Gbps - 400Gbps)
	A physical ethernet port dedicated to a customer
	Request made to AWS first, then completed by AWS Direct Connect Partners
2\. Hosted Connections (50Mbps - 25Gbps)
	Connection requests are made via AWS Direct Connect Partners 
	Capacity can be added or removed on demand

Often takes longer time (1 month) to establish a new Direct Connection.

Data isn't encrypted here by default, but secure as it is dedicated
To encrypt it you setup VPN using IPsec

#### Resiliency
This is designing redundancy for components that could fail. Sections that could fail
1\. A physical connection (traffic on that connection is interrupted)
2\. An AWS networking device fails (multiple connection is affected).
3\. An entire Direct Connect location fails (dependent connections are unavailable).
4\. Your primary Direct Connect path fails, and you need a backup route (Your network no longer reaches the premises router).

Each scenario requires a slightly different design

AWS provides three main designs through the Direct Connect Resiliency Toolkit.
1\. Development and test (2 connections, one location)
You order 2 physical Direct Connect connections at the same location, terminating on separate AWS devices.
- Protects against an individual connection / device failure
- Does not protect against failure of the entire location
- Choose this for dev and testing

2\. High resiliency (2 connections, 2 locations)
You create 2 physical Direct Connect connections, one at each of two different Locations
- Protects against connection failure
- Failure in one location doesn't affect the alternate path
- Designed for a 99.99SLA when all requirements are met

3\. Maximum resilience: four connections, two location
2 at Location A, 2 at Location B, with separate AWS device termination
- Protects against connect, device and location failures. designed for 99.99% SLA

### Backup options
Using the different designs above help improve reliability, but are expensive.
If a user wishes to have no internet connection for his setup, he can still use a Direct Connect for primary connection, and the set up Site-To-Site VPN as a secondary backup connection

#### Direct Connect Gateway
This is a global AWS resource and is necessary when you wish to setup a Direct Connect to one or more VPCs in many different regions (same account) without needing DC connection for every region

>Direct Connect provides the physical network connection btw on-premises and AWS
>Virtual Interface (VIF) establishes logical connection over Direct Connect and uses BGP to exchange routes [Public and Private]
>Direct Connect Gateway (DCGW) provides a way to connect that Direct Connect path to supported AWS gateways in one or more Regions
>Virtual Private Gateway (VGW) connects to an individual VPC
>Transit Gateway (TGW) acts a a central routing hub for multiple VPCs and networks
___

VIF (Virtual Interface)
This is a logical network interface configured on an AWS Direct Connect connection. 
It determines how network traffic travels over that connectino and which AWS network resources you can reach

- Direct Connect Connection: the physical network connection btw your network and AWS
- VIF: the Logical interface configured to carry particular type of traffic over the DC
- BGP: The routing protocol used to exchange network routes across the vIF



___
# Transit Gateway (central routing VPC hub)
This is a network transit hub that connects multiple VPCs and on-premises network through a central point

Think this as a central router for your aWS network

What the problem is:
Imagine you have four VPCs (a, b, c, d for backend, frontend, db, analytics)
Each VPC is a separate virtual network. Without Transit Gateway you might need to create and manage multiple VPC peering connections to enable communication

With four VPCs, a fully connected peering design needs six connection. With 10 vpcs, it needs 45 for each to be able to communicate with another (n(n-1) / 2)

Transit Gateway replaces this web of connections with a central routing hub.
Each VPC connects to the hub, and the hub routes traffic between networks when the routing configuration permits it.

It has its own routing table, separate from that in your VPCs.

*This is the only service in AWS that supports IP Multicasting* 

### Transit Gateway for scaling Site-to-Site VPN connection

It is also used to increase the bandwidth of your Site-to-Site connection using ECMP
ECMP = Equal cost multi-path
This is a routing strategy that allows you to forward packets over multiple best path

use-cases: create multiple site-to-site vpn connections to increase bandwidth of your aws connection

---
# Traffic Mirroring
This allows you copy network traffic from an EC2 instance's network interface, and send that copy to a monitorying or security appliance for inspection

This allows the original traffic to continue to its intended destination while a separate copy is sent for analysis

Captures traffic from a source (ENI) to a target (an ENI or NLB)

---
# IPv6 for VPC
IPv4 was designed to provide for 4.3 Billion addresses
IPv6 is designed to provide unique address for 3.4 x 10^38
You cannot disable IPv4 for VPC but can enable/disable IPv6

---
# Dual Stack VPC
This is a vpc configured to support both IPv4 and IPv6 networking

---
# Egress-only IGW (outbound)
This allows resources in an IPv6-enabled VPC to initiate outbound communication with the internet, while preventing the internet from initiating new connections back to those resources.

IGW: supports internet connectivity for IPv4 and IPv6, subject to routing and security controls
Egress-only IGW: Provides outbound-only internet for IPv6

#### The issue: IPv6 are all globally routable.

With IPv4, a private EC2 might have address 10.0.1.10. That add isn't publicly routable on the internet. To access the internet, you'd need a NAT Gateway, which translates the private to a public IPv4 address

With IPv6, an EC2 instance can have a globally routable IPv6 add even when it's a subnet you consider private

So the practical choice is:
- Private IPv4 instances need outbound internet access → typically use a NAT Gateway.
- IPv6 instances need outbound-only internet access → use an EIGW.
- Instances need general IPv6 internet connectivity, including potential inbound connectivity where routes and security rules allow it → use an IGW with appropriate controls.

---

# Networking Costs in AWS per GB
Use private IP instead of public IP for good savings and better network performance

Do not use a public IP to communicate between two resources in same region in two AZ
If you have a cluster doing a computation, use same AZ for max amount of savings (at the cost of high availability)

### Minimizing egress traffic network cost

Egress traffic: outbound traffic (From AWS to outside) [paid]

Ingress traffic: inbound traffic (from outside to AWS) [typically free as we've noted]

The goal is to keep as much internet traffic within AWS to minimize cost 

Picture this, you queried an endpoint, returns 100mb from db back to your application.
now after computation, you end up only using 50kb from the returned data
This would cost you much on Egress

Best to handle all the computation inside AWS (privately), and then return only 50kb back out of AWS to your app/website to be used directly (saves a lot of cost)

For S3. remember ingress is free. any data going into s3 is free

---
# AWS Network Firewall
A managed, stateful network firewall that protects traffic entering, leaving or moving between subnets in a VPC

Think this as a firewall deployed inside your VPC to inspect & control network traffic (not just a security group attached to an individual resource)

You can use it to:
- block traffic to known malicious IP addresses & domains
- Allow or deny traffic based on IP addresses, ports and protocols
- Inspect and filter HTTP/HTTPS traffic using domain based rules
- Detect / prevent suspicious network activity using Suricata-compatible instrusion

Network Firewall is deployed into dedicated firewall subnets. You must configure VPC route tables to direct traffic through the firewall endpoint.

For example, to inspect outbound internet traffic:
1. An EC2 instance sends traffic from a private subnet.
2. The private subnet's route table directs internet-bound traffic to the Network Firewall endpoint.
3. The firewall inspects the traffic.
4. Allowed traffic continues toward the NAT Gateway and Internet Gateway, according to your routing architecture.

Internally uses the Gateway Load Balancer

## Network Firewall vs other AWS security controls

| Service              | Main purpose                                                   |
| -------------------- | -------------------------------------------------------------- |
| Security Group       | Stateful traffic filtering at resource/network interface level |
| Network ACL          | Stateless subnet-level traffic filtering                       |
| AWS Network Firewall | Centralized, managed network inspection and filtering          |
| AWS WAF              | Protects web applications against HTTP(S) attacks              |
| AWS Shield           | DDoS protection                                                |
| GuardDuty            | Detects suspicious or malicious activity                       |
|Network Firewall + NAT Gateway|Inspect and control outbound internet traffic|
A common exam trap: AWS WAF is not a replacement for Network Firewall.
- WAF protects web applications from threats such as SQL injection and cross-site scripting.
- Network Firewall inspects network traffic at the VPC level.

---
# Firewall Manager

This is a centralized security management service

While Network Firewall actually inspects & filters network traffic, Firewall manager helps you centrally configure and enforce firewall & security policies across multiple AWS accounts

use-case
Imagine a company has 30 AWS accounts:
- Account A has an EC2 application.
- Account B hosts public websites.
- Account C has multiple VPCs.
- Account D has several S3 buckets.

Without centralized management, administrators must configure security protections separately in each account.

With Firewall Manager, the security administrator can define policies centrally and apply them across accounts and resources.

For example:

> Ensure every internet-facing Application Load Balancer is protected by AWS WAF, and every VPC has the required Network Firewall protections.

## What can Firewall Manager manage?

These are the important policy types to know for the exam:

|Policy type|What Firewall Manager does|
|---|---|
|AWS WAF|Centrally deploys web ACLs and WAF rules|
|AWS Shield Advanced|Applies DDoS protections to supported resources|
|Security Groups|Audits and manages security group configurations|
|AWS Network Firewall|Deploys and manages network firewalls across VPCs|
|Route 53 Resolver DNS Firewall|Applies DNS filtering rules across VPCs|
|Network ACLs|Centrally manages subnet-level network ACL policies|
|Third-party firewalls|Manages supported third-party firewall integrations|

For example, you can create one WAF policy and apply it to Application Load Balancers across multiple accounts.