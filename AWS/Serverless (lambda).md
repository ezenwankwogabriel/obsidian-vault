With serverless, you don't manage servers anymore (but the servers are still there, just you don't manually manage them)

Log in with Cognito, and using API Gateway you access Lambda, dynamodb, ...

# Lambda
Virtual Functions –– no servers to manage
Limited by time - short executions

Lambda Container Image:
	must implement the lambda runtime API
	ECS/Fargate is preferred for running arbitrary Docker images

API Gateway: creates our APIs and invoke our lambda
Kinesis: used to do some db transformations in the files

## Lambda Limits
	Execution
		Memory allocation: 128mb - 10GB (i mb increments)
		Max execution time is 900 secs (15 mins) (any above this time isn't good)
		Env variables can be up to 4kb only
		Disk capacity in the function container (in /tmp): 512mb - 10gb
		concurrency executions 100 (can be increased)
	Deployments:
		Lambda function deployment size (compressed.zip): 50mb
		Size of uncompressed deployment (code + dependencies): 250mb
		Can use /tmp dir to load other files at startup
		size of env veriable is 4kb

If questions ask beyond capabilities of this, like 30 mins execution time, 30GB of RAM, ... then Lambda isn't the suitable service for the request

## Lambda Concurrency
This is the number of Lambda function executions running at the same time

Concurrency limit: up to 1000 concurrent executions
You can set a "reserved concurrency (limit)" at the function level. This is a limit, like only 50 concurrent executions of the lambda

Imagine a Node.js API using Lambda to process uploaded files. If one file takes 2 sec to process and 100 files arrive at roughly same time, Lambda may need to run many executions simultaneously.

Each invocation over the concurrency limit will trigger a "Throttle". 
Suppose limit is 1000 and your app suddenly needs 1500 simultaneous executions. Results to a throttle
	- If sync invocation => return ThrottleError - 429 [APi Gateway request]
	- If async invocation => lambda queues event intenally and retries certain failures, but sustained throttling can lead to delayed processing and event loss if retry & event-age limits are exceeded.  [eventbridge event]
	- SQS event source mapping: Lambda polls the queue and adjusts processing concurrency.
If higher limit, open a support ticket

## Concurrency Settings
1\. Unreserved Concurrency:
Concurrency available to functions that havn't had concurrency reserved for them.
Ex: if yoour account has 1000 concurrency quota, and other functions use 300, and you havn't reserved capacity for your function, the shared pool has 700 left

2\. Reserved Concurrency:
Reserves capacity for a particular function & set its max concurrency
Ex: You set a payment functions reserved concurrency to 100.
	up to 100 executions can run concurrently for that function
	other functions cannot consume that reserved capacity
	if demand exceeds 100 for the function, they are throttled (even if some unreserved exist)

3\. Provisioned concurrency
Prepares execution envs in advance so functions can respond with less initial delay

This is best used for cold starts. you provision the needed capacity before the instance is started

Ex: Say you expect a burst of traffic when your app opens for bix. You configure 50 provisioned concurrent executions to reduce cold-start latency for that capacity

Concurrency isn't the total no. of requests, but is the no. of executions in progress at a given time.

# Lambda concurrency issue
Lambda functions share an account level concurrency limit. So one busy function can affect other functions.

Shared concurrency Limit: Suppose your AWS account has regional lambda concurrency quota of 1,000.
And you have 3 lambda functions:
	A. processes uploaded files
	B. handles payment
	C. sends emails
They generally draw from the same pool of "unreserved" concurrency unless you've configured "reserved" concurrency for individual functions

![[Pasted image 20261020124454.png]]

If function A needs 1,100 concurrent executions:- it cannot simply take all the concurrency it wants.
Also if function B needs more, function A already took up all shared capacity, and function B now gets throttled

## Lambda snapstart
Improves lambda functions performance up to 10x at no extra cost for Java, Python & .Net

## Customization At the Edge
The idea is that many modern apps execute some form of logic at the edge
These are called "Edge Functions"
	A code you write and attach to cloudfront distributions
	It runs close to your users to minimize latency

CloudFront provides two types:
	CloudFront functions: 
		lightweight functions written in JavaScript. 
		Simple things that can be executed in less than 1 milliseconds(header transformation, URL rewrites/redirects, Request auth)
		For high-scale, latency-sensitive CDN customization
		Used only to change Viewer request and response
			Viewer Request: after CloudFront receives a request from a viewer
			Viewer Response: before CloudFront forwards response to the viewer
		
	Lambda@Edge:
		more powerfule code for complex request & response processing @ edge
		(adjust cpu/memory, network access, 3rd party libaries, file system access)
		written in Nodejs or Python
		scales to 1000s of request/second
		Used to change CloudFront requests and responses
			View Request - after cloudfront receives a request from viewer
			Origin Request - before cloudfront forwards the response to the origin 
			Origin Response - after cloudfront receives the response from the origin
			Viewer response - before cloudfront forwards response to viewer

Use-cases for Edge Functions
- Website security & privacy
- Dynamic web application at the edge
- SEO
- Intelligent route across origins and data center
- Bot mitigation
- Real time image transformation
- A/B testing
- User authentication & authorization
- .....

## The exam comparison table

| Feature                  | CloudFront Functions            | Lambda@Edge                               |
| ------------------------ | ------------------------------- | ----------------------------------------- |
| Best for                 | Simple, lightweight logic       | Complex processing                        |
| Languages                | JavaScript                      | Node.js, Python                           |
| Execution time           | Submillisecond                  | Up to 30 seconds                          |
| Viewer request           | Yes                             | Yes                                       |
| Viewer response          | Yes                             | Yes                                       |
| Origin request           | No                              | Yes                                       |
| Origin response          | No                              | Yes                                       |
| Network access           | No                              | Yes                                       |
| Request body access      | No                              | Yes, with restrictions                    |
| File system access       | No                              | Yes                                       |
| Scale                    | Millions of requests per second | Lower regional invocation capacity limits |
| CloudFront KeyValueStore | Supported                       | Not supported                             |

## Defaults in Lambda

Lambda runs outside your VPC by default (unless you explicitly configure it to connect to your vpc). This is as it is AWS managed environment

By default, it can access services like S3, DynamoDB without you placing it inside you VPC. However, it cannot auto access private resources inside your VPC such as an RDS database, ElastiCache, internal ELB that has no public access.

To do these, launch Lambda in your VPC.

If Lambda functions directly acess your db, they may open too many connections under high load. hence the need for a "Proxy"
### RDS Proxy
This sits between your app and your db, reusing db connections so your db doesn't get overwhelmed by too many connections

Imagine you have 2k users accessing your app running on AWS Lambda.

Without RDS Proxy, many lambda execution might open their own connections to your RDS database at the same time (tadaaa)
The problem isn't that d db can't process queries, but it may be overwhelmed by the number of open connections

RDS Proxy solves this by maintaining a pool of database connections and reuses them across app request when possible

But for this to work, the Lambda must be deployed in your VPC, because *RDS PROXY IS NEVER PUBLICLY ACCESSIBLE*

- Improves scalability by pooling and sharing dB connections
- improves availabilty by reducing by 66% the failover time and preserving connections

## Invoking Lambda from RDS and Aurora

This allows invoking lambda functions from within your DB instance
Which allows for processing data events from within a db
Supported for RDS for PSQL and Aurora MySQL


## AWS Step Functions
A way to build serverless visual workflow to orchestrate your lambda function

Features: sequence, parallel, conditions, timeouts, error handling

Can integrate with EC2, ECS, On-premises server, API Gateway, SQS queues, etc

**Imagine you're building an online shopping application.**
When a customer places an order, your system needs to:
1. Validate the order.
2. Process payment.
3. Check inventory.
4. Arrange shipping.
5. Notify the customer.
What happens if payment fails? What if inventory is unavailable? What if the notification service temporarily fails?

Without Step Functions, you might write a large Lambda function that calls all these services, manages branching, retries failed operations, and tracks progress.

With Step Functions, you define the workflow, and AWS manages its execution.

## Cognito
Helps you add user sign-up/sign-in/access control to your app without building an authentication system from scratch

For exam:
user pools -> authenticate users
identity pools -> provide temporary AWS credentials

There are two main Cognito components

1\. Cognito User Pool (user authentication)
A user pool is a user directory and an authentication service

It can handle
> user sign-up & sign-in
> password resets & account recovery
> MFA
> Federated sign-in with supported identity providers
> issuing jwt

example: customer logs into your app with email & pass. Cognito verifies their identity & issues tokens your app can use to establish an authenticated session

2\. Cognito Identity Pools (temporary AWS credential) (federated identity)
An identity pool can exchange supported identity information for temporary AWS credential through AWS Security Token Service (STS)

These credentials let an app access permitted AWS resource directly, subject to the IAM role & permission

example: A logged-in user uploads a photo directly from a mobile app to a specific S3 location without routing the file through your backend
more like for accessing more resources/services inside AWS

![[Pasted image 20261020164328.png]]

Cognito User Pool integrates with API Gateway and Application Load Balancer