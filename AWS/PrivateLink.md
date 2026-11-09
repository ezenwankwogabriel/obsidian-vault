AWS PrivateLink allows you to privately access services hosted in another VPC without sending traffic over the public internet or requiring VPC peering.

Think of it as a private connection between your VPC and another company's or team's service.

## Concrete example

Imagine you have two companies:

- Company A provides a payment processing service.
- Company B runs an e-commerce application on AWS.

Company B needs to call Company A's payment API, but Company A doesn't want to expose its service to the public internet.

PrivateLink solves this

![[Pasted image 20261104075629.png]]

### How it works

1. Company A deploys its payment service behind a Network Load Balancer (NLB).
2. Company A creates a VPC Endpoint Service backed by that NLB.
3. Company B creates an Interface VPC Endpoint in its own VPC.
4. AWS creates private network interfaces (ENIs) with private IP addresses in Company B's selected subnets.
5. Company B's application connects to the endpoint.
6. Traffic travels privately through AWS PrivateLink to Company A's service.

Company B doesn't need access to Company A's VPC or its CIDR range.