This is AWS Content Delivery Network. Delivers content to users from locations closer to them.

Concepts:

1\. Distribution

The configuration you create to tell cloudfront how to deliver your content. On creating one, CloudFront gives you a domain name

2\. Origin

The origin is where cloudfront gets the original content when needs it, and then caches them at the Edge.
While CloudFront is global, your origin might be in a single AWS Region. CloudFront doesn't auto replicate your entire app/db worldbase

	S3 Buckets
		For distributing files & caching them at the edge
		for uploading files to S3 through cloudfront
		secured using origin access control
	
	VPC Origin
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

Issue: You have a deployed app in one region, but users are in all parts of the world. Ideally you'd have delay/high latency accessing this from different parts of the world

Imagine your application runs on an Application Load Balancer in London, but users access it from Lagos, New York, and Singapore. Rather than relying entirely on the public internet's routing, Global Accelerator directs users through AWS's global network toward the appropriate endpoint.

Uses two static anycast IPv4 addresses/DNS/provide a stable entry point. This sends traffic directly to the Edge locations closest to you. the edge location sends the traffic to your app.

It doesn't auto create a second copy of your application. You must configure healthy alternative endpoints and deploy your application there if you want regional failover

think this as a smart global entrance to your application

It works with Elastic IP, EC2 instances, ALB, NLB, pub or privvate
