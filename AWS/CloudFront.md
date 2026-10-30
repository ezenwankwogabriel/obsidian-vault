This is AWS Content Delivery Network. Delivers content to users from locations closer to them.

Concepts:

1\. Distribution

The configuration you create to tell cloudfront how to deliver your content. On creating one, CloudFront gives you a domain name

2\. Origin

The origin is where cloudfront gets the original content when needs it, and then caches them at the Edge.
While CloudFront is global, your origin might be in a single AWS Region. CloudFront doesn't auto replicate your entire app/db worldbase
Origins could be:
	S3 Buckets
		For distributing files & caching them at the edge
		for uploading files to S3 through cloudfront
		secured using origin access control
	
	VPC Origin (keeps connection private to vpc hosted apps)
	A vpc origin lets AWS Cloudfront securely reach an app hosted inside a VPC, without requiring that app to have a public IP address
		For applications hosted in VPC private subnet
		Private app Load Balancer / Network Load balancer / EC2 instances
	
	Custom Origin (HTTP)
		An S3 website
		Any public HTTP backend you want
	
	Elastic Load Balancer, API Gateway, Elemental Media Package

3\. Edge Locations & Regional Edge Caches

Edge Location: The first caching layer that handles the viewers request. designed to serve users with low latency
Content is cached at the edge (for frequently requested contents)

Regional Edge Cache: an additional caching layer btw edge location & the origin.  retains content that isn't frequently requested at an individual edge location, reducing trips to the origin

Origin: The source of the original content

*Flow*
*Viewer ––> Edge Location ––> Regional edge cache, if applicable ––> Origin*


By having contents globally, it gives us DDoS protection

## Securing CloudFront & S3
	 Origin Access Control
		 Allows CloudFront to make authenticated requests to an S3 origin. 
		 You configure a bucket policy to allow intended CloudFront distribution access the object
		 This keeps bucket private while serving content. 
		 OAC is recommended over the older Origin Access Identity (OAI)
		
		Exam clue: “Serve private S3 content through CloudFront without making the bucket public” → choose Origin Access Control (OAC).
	
	Signed URLs vs signed cookies
		These control who can access private content through CloudFront
		It doesn't replace OAC: 
			OAC protects the origin, while Signed URL & cookies restrict viewer access

| Feature                 | Signed URL                    | Signed cookie                                          |
| ----------------------- | ----------------------------- | ------------------------------------------------------ |
| Best for                | One specific file or download | Multiple protected files                               |
| Example                 | A paid PDF download           | A subscriber watching a video made up of many segments |
| Requires authorization? | Yes                           | Yes                                                    |

## CloudFront vs Cross Region Replication

CloudFront: (CDN)
	uses a global edge network
	cached for a TTL
	specific for static content that must be available everywhere

S3 Cross Region Replication (Replicate a whole bucket to other region)
	must be setup for each region you want replication to happen
	Read only files
	Files are updated in real time
	Great for dynamic content that needs to be available at low latency in few regions
	Can serve for a CMS site

## CloudFront vs S3 Transfer Acceleration vs Global Accelerator

These AWS services can sound similar in exam questions.
Both CloudFront and Global Accelerator both use AWS global network & it's edge locations around the world
Both integrate with AWS shield for DDoS protection

There's no caching for Global Accelerator, the requests still gets the applications running in one/more Regions from the edge
GA is usefulf is you need a static IP address for HTTP and for non-HTTP such as gaming/IoT/VoIP...

| Service                  | Main purpose                                                                                                | Remember it as              |
| ------------------------ | ----------------------------------------------------------------------------------------------------------- | --------------------------- |
| CloudFront               | Cache and deliver web content globally                                                                      | Deliver content faster      |
| S3 Transfer Acceleration | Speed up uploads to S3 using AWS edge locations and the AWS network                                         | Upload to S3 faster         |
| AWS Global Accelerator   | Route traffic to healthy application endpoints using static anycast IP addresses and the AWS global network | Reach applications reliably |
| Route 53                 | DNS routing and domain-name resolution                                                                      | Find the destination        |

For example, if a company wants to distribute static images and videos worldwide, CloudFront is the natural choice. If users around the world need to upload large files to a central S3 bucket faster, consider S3 Transfer Acceleration.

unicast IP: one server holds one IP address
anycast IP: all servers hold same IP address and client is routed to the nearest one 

## AWS Global Accelerator

Global Accelerator & cloudfront both improve application performance globally.

The easiest way to distinguish them:
- Global Accelerator: Improves the network path users take to reach your application.
- CloudFront: Caches and delivers content closer to users.

Imagine you have an application hosted in two AWS Regions:
- US East (N. Virginia)
- Europe (Frankfurt)
Users access your application from Nigeria, the UK, the US, and other locations.

Without Global Accelerator, users connect to your application through its public endpoint, potentially taking different internet routes.

Global Accelerator gives you two static anycast IP addresses that users connect to.

It routes traffic through the AWS global network to an optimal healthy endpoint.

How it works
1. A user connects to one of the Global Accelerator static IP addresses.
2. Traffic enters AWS at a nearby AWS edge location.
3. AWS carries the traffic over its global private network.
4. Traffic reaches the selected healthy endpoint, such as an ALB in an AWS Region.
5. The ALB distributes requests to your EC2 instances.
Important: Global Accelerator doesn't cache your content. It accelerates network traffic.

think this as a smart global entrance to your application

It works with Elastic IP, EC2 instances, ALB, NLB, pub or private

use-cases:
- Global gaming application
- Multi-Region application: you have an app in two region, if primary is unhealthy, global accelerator can redirect traffic to another region, subject to health checks
- static IP requirements: Your customers need to allow fixed IP addresses in their firewalls. Global Accelerator provides two which remain same anywhere you go


