A managed service for creating, publishing, securing, monitornig and managing APIs
Creating REST APIs that gets proxied to the required service (Lambda)

The front door to your backend. A client sends hTTP request to API Gateway, and API Gateway routes that request to the backend, such as Lambda, an HTTP service, or another AWS service

API Gateway + Lambda => Serverless APIs
API Gateway + DynamoDB => API that can expose data operations without requiring server managements
API Gateway + auth and throttling => controlled, protected API acces

## Things you can expose using API Gateway
- Lambda Function
	- Easy way to expose REST API backed by AWS Lambda
- HTTP
	- Expose HTTP endpoints in the backend
	- Why? Add rate limiting, caching, user auth, API keys, ...
- AWS Service
	- Expose any AWS API 
	- ex: start an AWS step function workflow, post a message to SQS
	- Why? Add auth, deploy publicly

## API Gateway endpoint types
The key exam distinction is where clients access your API from and whether the API should be publicly accessible or restricted to a VPC.

1\. Regional Endpoint:
Your API is accessed through an endpoint in a specific AWS Region.
- Best for clients primarily in the same Region or when you manage your own CDN.
- You can put CloudFront in front of it if needed.
- It is publicly accessible by default, subject to authorization and resource policies.

2\. Edge-optimized Endpoint
API Gateway uses an AWS-managed CloudFront distribution to improve access for geographically distributed clients.
- Useful when clients are spread across different locations.
- Requests enter through CloudFront edge locations and are routed to the API's Region.
- AWS manages the distribution for you.

3\. Private endpoint
The API is accessible through an interface VPC endpoint powered by AWS PrivateLink.
- Intended for clients inside your VPC or connected private networks.
- Traffic can stay on private AWS networking rather than requiring a public API endpoint.
- Access can be controlled with VPC endpoint policies and API resource policies.

## Security
1\. User Authentication through
	IAM Roles (useful for internal apps)
	Cognito (identity for external users)
	Lambda Custom Authorizer (your own logic/external auth)
2\. Custom Domain Name HTTPs security through integration with AWS Certificate Manager (ACM)
	If using Edge-optimized endpoint, certificate must be in us-east-1
	If using Regional endpoint, certificate must be in the API Gateway region
	Must setup CNAME or A-alias record in Route53