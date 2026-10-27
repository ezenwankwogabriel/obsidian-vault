DNS normally translates domain names to IP addresses
Route53 is a Domain Registrar and the only AWS service providing 100% availability SLA

## Record Types
	A: Maps a hostname to IPv4
	AAAA: Maps a hostname to IPv6
	CNAME: maps hostname to another hostname
		The target is a domain name that must have an A / AAAA record
		Can't create CName for top node of DNS namespace
		example: can create CName for "www.x.com" and not "x.com"
	NS: Name Servers for the Hosted Zone

## Hosted Zones
A container for records that define how to route traffic to a domain / subdomain
	**Public Hosted Zones**: contains records that specify how to route traffic on the internet (public domain names) normal public domains
	**Private Hosted zones**: contains records that specify how you route traffic within one or more VPCs. domains used wthin a private space : application | .company.internal

TTTL: Time To Live
This caches the response for the specified timing (in domain configuration)
High TTL: 24hrs > less traffic on Route 53. Possibly outdated records
LowTTL: 60sec > More traffic on Route53, outdated for less time, easy to change records
	TTL Is mandatory for each DNS record except the ALIAS record

### CName vs Alias
They both point one DNS name toward another destination
	Alias records is an AWS Route 53 feature designed to work with AWS resources

Imagine you own a resource, and want the visitors to reach your load balancer using your own domain (www.anchora.com.ng). This can be done using an Alias record. A CName can also point a hostname to another hostname, subject to DNS rules

| Feature                                          | CNAME                                 | Route 53 Alias (free of charge)                                                   |
| ------------------------------------------------ | ------------------------------------- | --------------------------------------------------------------------------------- |
| What it does                                     | Points a hostname to another hostname | Points a DNS name to a supported AWS resource or another eligible Route 53 record |
| Works with AWS resources                         | Yes, through their DNS names          | Yes, directly supports many AWS resources                                         |
| Can be used at the zone apex (`anchora.com.ng`)? | No                                    | Yes, for supported targets                                                        |
| Additional Route 53 query charge                 | Normal DNS query pricing              | Queries to many supported AWS resource aliases are free; exceptions apply         |
| Standard DNS record type?                        | Yes                                   | No, it's a Route 53-specific feature                                              |
Alias record is always of type A/AAAA
Alias (Route 53 feature) ––	A supported AWS resource or another record –– e.g. An Application Load Balancer/CloudFront Distributions, ... aws resoruces
*You cannot set an Alias for an EC2 DNS name* instead can use *CNAME*

| Your situation                                                   | Recommended solution  |
| ---------------------------------------------------------------- | --------------------- |
| Point a subdomain to an EC2 instance's public DNS name           | CNAME record          |
| Point a domain directly to an EC2 instance's public IPv4 address | A record              |
| Point a domain to an EC2 instance using IPv6                     | AAAA record           |
| Point a domain to an Application Load Balancer                   | Route 53 Alias record |
| Give an EC2 instance a stable public IPv4 address                | Elastic IP + A record |

## Routing Policies
Defines how Route 53 responds ot DNS queries

	Simple Routing Policy
		Typically route traffic to a single resource
		can specify multiple values in same record
		if multiple values are returned, Route53 returns all matches, and a random one is chosen by the client
		*cannot be associated with health checks*
		if alias is enabled, can only specify one AWS resource

	Weighted Policy (like an A/B / weighted distribution)
		Control the % of the requests that go to each specific resource
		Assign each record a relative weight (weights don't sum up to 100)
		DNS records must have same name and type
		can be associated with health checks
		for load balancing between regions, testing new app versions, ...
		A weight of 0 stops sending traffic to that region

	Latency Policy (low-latency)
		Redirect to a resource that has the least latency to us
		Helpful for when latency for users is a priority
		Can be associated with healthchecks

	Failover Policy
		Having a healthcheck on a primary, designate a Disaster Recovery Secondary. If the primary fails, Route53 failsover to the secondary
		you should have these healthchecks setup first in Route53 -> Health check

	Geolocation Policy (closest to location)

	Geoproximity (bias control)
		Allows route traffic to resources based on geographical location of users & resources
		Ability to shift more traffic to resources based on the defined bias
		if you want more traffic to resource > expand bias (1-99) or shrink to reduce traffic
		Resources here can be AWS Resources & non AWS resources. The Advanced Traffic flow feature must be used for this

	IP Based Policy
		routing based on clients IP address

	Multi-Value Policy
		When routing traffic to multiple resources
		Route 53 return multiple values/resources
		Returns values for only healthy resources
		Up to 8 healthy reocrds are returned for each multi-value query	

HealthChecks
	Endpoint: Establishes a connection with the resource to determine its health
	Calculated: The status of the health check is based on the status of the other health checks
	Cloudwatch alarm: the status of the health check is based on the state of a specified cloudwatch alarm
You create health checks for AWS resources like EC2 instances, specify the IP address, port, path

## Hybrid DNS (Resolver Endpoints)
Route 53 resolver endpoints allow DNS queries travel between your AWS VPC and your on-premises network
This is a a DNS gateway between AWS and your own data center.
	Inbound endpoint: Your on-premise ntw sends queries to AWS so it can resolve private DNS name inside your VPC
	Outbound endpoint: Resources in your VPC send DNS queries to your on-premise DNS servers, typically through Resolver forwarding rules
