Question: 
When service A needs access to Service B, do you give Service A an IAM role, put a resource policy on Service B, or use Both?

Answer:
This depends on which service is making the request, and how the target service authorizes it

1\. Understanding the two sides
Imagine your Lambda function (caller) needs to save a file to S3 (target).
	Questions:
	- IAM role / identity-based policy: What is the lambda function allowed to do?
	- Resource-based policy: Who is allowed to access the S3 bucket?

An IAM role is an identity (defines action) that a service or workload can assume. The permissions attached to that role tells AWS what actions the role can perform

A resource policy is attached to the target resource and specifies which principals may access it.

*key:* A role and policy are not same. Role is an identity, policy defines permission.

2\. When do you use each

Case A: IAM role only
Use IAM role with the appropriate permission when the caller needs permission to access a resource, and then the target service's authorization doesn't require a separate resource policy for access

example: an EC2 instance needs to read an SSM (systems manager) parameter in same account
	Attach an IAM role to the EC2 instance
	Grant the role the permission "ssm:GetParameter"
	Ensure the parameter's encryption key, .. and other requirements permit access
The EC2 instance uses temporary credentials from its role rather than storing long-lived access keys inside the server

Case B: Resource-based policy only
The target resource needs to specify which principal can access it, and that policy is sufficient for the particular auth path

example: EventBridge needs permission to invoke Lambda function
	Add a lambda resource-based permission allowing EventBridge invoke the function
	EventBridge does not normally need you to create an execution role to invoke a Lambda target in this scenario
The target Lambda function is the resource being protected, so the permission belongs on the lambda function

Case C: Both
Use both when the caller needs its own permissions and target resource must authorize the caller

Example: a lambda function in account a needs to read  an S3 bucket in account b
- Give lambda an execution role permission to call "s3:GetObject"
- Add an S3 bucket policy allowing that role from Account A 
- Ensure no applicable policy explicitly denies the request



## The easiest way to decide: ask these three questions

## 1Who is making the request?

If an application or AWS service needs to call another service, check whether the caller needs an IAM role with permissions. This is common for EC2, Lambda, ECS tasks, Step Functions, and EventBridge targets such as Kinesis.

## 2Does the target need to authorize the caller?

If the target supports a resource-based policy, you may need to configure one. Lambda invocation permissions, S3 bucket policies, SQS queue policies, and KMS key policies are important examples.

## 3Is it cross-account?

Cross-account access generally requires authorization on both sides: the caller's identity policy and the target resource's policy, or an alternative role-assumption design. KMS and some other services have their own specific authorization rules.

## Remember with these examples

| Scenario                                   | Typical authorization setup                                               |
| ------------------------------------------ | ------------------------------------------------------------------------- |
| Lambda calls S3 in the same account        | Lambda execution role                                                     |
| EventBridge invokes Lambda                 | Lambda resource-based policy, or a supported execution-role configuration |
| EventBridge writes to Kinesis Data Streams | IAM role assumed by EventBridge                                           |
| Lambda in Account A reads S3 in Account B  | Lambda role and S3 bucket policy                                          |
| Lambda decrypts data with KMS              | Lambda role plus the necessary KMS key authorization                      |