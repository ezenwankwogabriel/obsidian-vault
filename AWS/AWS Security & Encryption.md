## Encryption
- what encryption is
- where data is encrypted
- who controls the keys
- which AWS service handles the encryption

Encryption converts readable data into unreadable data (ciphertext) using cryptographic key.

### Encryption at rest
Data is stored on a disk, database, backup or object storage system
ex: an encrypted EBS volume, S3 object, RDS db, or snapshot
### Encryption in Flight/Transit (TLS/SSL)
protects data while it travels from one place to another across a network
Examples: HTTPS using TLS btw a browser and an application or TLS btw an application and a db

### Encryption in use
protects data while it is being processed
Example: A database processes sensitive patient information while protecting it from unauthorised access to memory.

---
## AWS KMS (key manaement service)
The go to for any encryption for an AWS service. AWS manages encryption Keys for us
Gives the advantage of auditing KMS key usage using CloudTrail
KMS can be used seamlessly in most AWS services

### KMS Key Encryption Types
- Symmetric (AES-256 key)
	- A single encryption key used for Encrypting and Decrypting data
	- AWS services integrated to KMS use this symmetric keys
	- The idea is that you never get access to the key itself. You use the KMS api to manage/use the key
- Assymetric (RSA & ECC key pairs)
	- Public (Encrypt) and Private (Decrypt) pair
	- Two keys for encryption and decryption
	- You can download the public key out of KMS but can't access the private key
	- use-case: encryption outside of AWS by users who can't call the KMS API
- HMAC (hash-based message authentication code) key
	- Generate & verify message authentication codes (a hash key)
	- use-case: verify data authenticity and integrity

### KMS Key Types
1\. AWS Owned Keys (free)
	This is the default key. AWS owns and operates these keys for integrated services.
	They are not managed by you as individual keys in your account

	They are used behind the scenes by AWS services to protect customer data without expsing those keys in your AWS account.

	use-case: think => least customer key-management overhead
		Imagine you use an AWS "service" that encrypts data using AWS-owned KMS keys.
	
	Example: S3 uses AWS-owned keys as part of its SSE-S3 encryption mechanism.
	You don't need to create a KMS key or manage anything.
	
	You cannot view them in kms console
	Cannot modify their policies / configure their rotation

2\. AWS Managed Key (free)
	AWS manages the key in your account for a particular service. These commonly have aliases beginning with "aws/" follwed by the service they manage. such as "aws/S3"

	use-case: think => convenient service-specific KMS encryption
	For example, when you use SSE-KMS with S3 and don't specify your own KMS key, S3 can use:
	`alias/aws/s3`
	
	You can view your key in kms console, see it's key id and meta, and view usage in CloudTrail activity

Both owned and managed keys are managed by AWS

3\. Customer managed KMS keys
	A KMS key that you control.
	You manage key policy, permissions, enabled state, deletion, ...	
	costing $1 per month

	use-case: think => greater control, auditability, and configuration responsibility

AWS key rotation
- AWS-managed KMS key: automatic every 1 year (rotated)
- Customer managed KMS key (must be enabled): could either be automatic or on-demand
- Imported KMS key: only manually managed requiring an alias

### Copying Snapshots across regions
Given an EBS volume encrypted with KMS
- First take a snapshot of the EBS volume that's encrypted with KMS
- Attach a KMS key policy to authorize cross-account access (like below)
- ```
  {
	  "Sid": "Allow use of key with destination account"
	  ...
	  "Condition": {
		  "StringEquals": {
			  "kms:ViaService": "ec2:<REGION>.amazonaws.com",
			  "kms:CallerAccount": "<TARGET-ACCOUNT-ID>"
		  }
	  }
  }
  ```
- Share the encrypted snapshot with the Target account
- In the target account, create a copy of the Snapshot, encrypt it with a KMS key in your account [same kms key can't live in two regions]
- Create a volume from the snapshot in the target account (restore the snapshot)

### KMS Key Policies
If you don't have KMS key policy on you KMS key then no one can access it.

IAM Policy => Grants/Denies permissions to an IAM identity
KMS key policy => Controls access to a particular KMS key
KMS grant => Delegates specified key permissions to a principal

Example:
	An IAM role has an IAM policy allowing "kms:Decrypt"
	
	However, the key policy doesn't allow that role's access and doesn't enable the account to delegate access through IAM policies
	
	Result: the roles IAM Allow alone is insufficient. Key policy must provide auth path or suitable grant for the ooperation.

Types of KMS Policy
	Default KMS key policy: Created if you don't provide a specify key policy
		gives complete access of the key to the root user
	Custom KMS key policy: 
		Define users, roles that can access the KMS key
		Define who can administer the key
		useful for cross-account access of your kms key

---
### KMS Multi Region Keys
imagine you app runs in two AWS regions:
- Ireland: eu-west-1
- london: eu-west-2
Your app encrypts sensitive customer data in Ireland. During a disaster, your app needs to decrypt that same data in London.

With ordinary, single Region KMS keys, a key in London cannot decrypt ciphertext encrypted by a key inIreland.

With Multi-Region key, related keys in Ireland & London can decrypt same ciphertext lcoally.
They share the same underlying key material and keyId, while remaining separate regional resource
#### How it works
1\. You create a multi-region primary KMS key in one region (Ireland)
2\. You create a replica key in London and other region (multi-region primary in these regions now)
3\. Your application encrypts the record using the primary key in Ireland
4\. You copy the ciphertext (encrypted record) to London
5\. Your London app uses the replica key to decrypt the cipher text

If you have autorotation, this is replicated in the other regions as well.

NB: These multi-region keys are not global. You have your primary + replicas
NB: they do not auto inherit permission settings. they each have their own key policy
NB: not encouraged except in very key use-cases: like if you want to have a global client side encryption, encryption on global dynamodb, ...

---
## S3 Replication with Encryption Considerations
When you enable replication for S3, unencrypted objects and objects encrypted with SSE-SE are replicated by default
Objects encrypted with SSE-C (customer provided key) can be replicated

For objects encrypted with SSE-KMS, you'd need to enable this option. You would specify which KMS key to encrypt the objects within the target bucket (similar to what we discussed earlier)
Then adapt the KMS Policy for the target policy
And an IAM role with "kms:Decrypt" for the source KMS key and "kms:Encrypt" for the target KMS key

---
## Encrypted AMI Sharing Process
Sharing AMI with another account when the AMI has been encrypted with a KMS key

>NB: Sharing an encrypted AMI is different from using a multi-region KMS key. You don't normally need multi-Region key to share an AMI between accounts in same Region

Imagine:
- Account A: owns an EC2 instance & creates an encrypted AMI from it.
- Account B: wants to launch its own EC2 instance using that AMI

Account A has:
- An AMI called *company-golden-ami*
- An encrypted EBS root volume
- A customer-managed KMS key called alias/account-a-key

Account B wants to use that AMI. This happens in two independent permission checks

1\. Share the AMI using launch permissions
An AMI has an attribute called launch permission. 
For example:
- Account A: `111111111111`
- Account B: `222222222222`

Account A modifies the AMI's launch permissions to include Account B's account ID
```
AMI: company-golden-ami

Launch permissions:
- Account A: Owner
- Account B: 222222222222
```

Now, Account B can see and use the shared AMI, subject to the remaining permissions.
*important:* this doesn't grant access to the encrypted EBS snapshot's kms key

2\. Share the KMS key
Now consider the encryption for the EBS used in the AMI
The AMI references EBS snapshots encrypted using Account A's customer-managed kms key

Account B tries to launch an EC2 instance, AWS needs to access the encrypted snapshot data to create the new EBS volume. But the encryption key belongs to account A. Hence Account B cannot use it.

Account A must modify the KMS key policy to allow Account B use the key.
```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::222222222222:root"
  },
  "Action": [
    "kms:Decrypt",
    "kms:DescribeKey",
    "kms:ReEncrypt*",
    "kms:CreateGrant"
  ],
  "Resource": "*"
}
```

This illustrates a cross-account key-policy statement.

3\. Configure IAM permission in Account B
Even though Account A's kms key policy trust B, and B can see AMI for A, the user/role in Account B needs appropriate IAM permission

Think this as: the launching user / role is authorized to use the shared key

4\. Re-encrypt using Account B\s own KMS key
This is optional but recommended


NB: this most likely works for same region. for cross region, the encryption source for EBS must be using *customer-managed KMS key*. AWS managed keys like *aws/ebs* cannot be shared cross-account.


---
## SSM Parameter Store
Systems Manager Parameter Store is a secure storage for configuration and your secrets

This is for storing single key(path)-value pairs

Instead of hardcoding those values in source-code, store them in Parameter Store 

Types:
- String: any string value
- StringList: Separate strings using commas
- SecureString: Encrypt sensitive data using KMS keys from your account or another account

Can encrypt these configurations using KMS
Is serverless, has easy to use SDK
Security is provided through IAM

Tiers for SSM Parameter store
- Standard: 4kb parameter value max (10,000 parameters)
- Advanced: 8Kb parameter value max (100,000 parameters)

---

For Secret Manager, This is like a config file in AWS, store db host, port, feature flag, endpoints, ...

---
### ACM (AWS Certificate Manager)
Used for provision, management and deployment of TLS certifications
TLS provides in-flight encryptoin for websites
ACM supports both public and private TLS certs

To request a public certificate:
- list the domain names to be included in the certificate
- select validation method: DNS validation / Email Validation 

You can also generate certificate outside of ACM and import it. ACM would send expiration events so you don't loose track
Also, AWS Config has a managed rule "acm-certificate-expiration-check" to check for expiring certificates (configurable number of days)

---
### AWS Cloud HSM (Hardware Security Module)
With KMS => AWS manages software for encryption
With **CloudHSM** => AWS provisions an encryption hardware, for cryptographic operations, and certain AWS services can use those HSMs to protect their encryption keys

Hence you manage your own encryption keys entirely (not AWS)
This is a good candidate for SSE-C encryption on top of S3

CloudHSM Integration with AWS Services
Pattern A: CloudHSM through KMS
	AWS KMS supports a feature called AWS CloudHSM key store, alias (a custom key store)
	It lets KMS keep material in your CloudHSM cluster while retaining KMS's familiar API, permissions and integrations
	
	example: required to encrypt supported EBS volumes while keeping KMS key material in dedicated HSM you control, you can configure KMS to use CloudHSM keystore.

Pattern B: Your app connects directly to CloudHSM
Can integrate an application ruuning on EC2 with CloudHSM using supported libraries & client software.

example:
	web servers: Protect private keys used for SSL/TLS operations
	Certificate authorities: Protect certificate-signing private keys
	...

---
## AWS WAF (Web Application Firewall) [layer 7 protector]
This is used to protect your app from common web exploits at Layer 7 (http)

Think this as a security guard in front of your websites, inspecting web requests and deciding which to allow, block or count

It can be depolyed with supported resources such as:
CloudFront | ALBs | APIGateway | AppSync | Amaozon Cognito user pools | ...

It cannot be deployed on an NLB

Once deployed, you can define Web Access Control List (Web ACL) Rules
	can define IP sets: up to 10k IP addresses
	set HTTP headers, HTTP body, URI strings protects from common attacks
These Web ACLs are regional, except for CloudFront where they are defined globally
A rule group is a set of rules you can add to any Web ACL

#### WAF - Fixed IP while using WAF with a Load Balancer
- WAF doesn't support NLB (layer 4)
- and ALB doens't have fixed IP
- we can use a Global accelerator for this, and apply WAF on the ALb
	- Global accelerator gives your app two static anycast IP addresses that users can connect with. 

---
## AWS Shield (DDoS protector) [ALB, CLB, NLB, Elastic IP, Cloudfront]
Setup Shield to prevent overwhelming your application with DDoS.
Shield standard:
	this is a free service already activated for every AWS customer
	Provides protection from attacks such as SYn/UDP floods, reflection attacks and other layer 3/layer 4 attacks
Shield Advanced:
	Optional migration costing $3k per month per organization
	protects against more sophisticated attack on AWS EC2, ELB, CloudFront, Global Accelerator, and Route 53
	Gives you 24 hours access to the AWS DDoS response team
	Auto deploys WAF rules to mitigate layer 7 attacks

---

## AWS Firewall Manager
A security management service that allows you to centrally configure and manage firewall rules across your accounts and apps in AWS Organization.

Security policies:
	WAF Rules
	AWS Shield Advanced
	Security groups 
	AWS Network Firewall
	Route 53 resolver DNS Firewall
	Policies are created at the region level

If you wish to use WAF rules across accounts, accelerate WAF Configurations, automate protection of new resources, Firewall Manager with AWS WAF is the go to

---
## AWS GuardDuty  (security surveillance)
GuardDuty is an intelligent threat detection service that continuously analyzes AWS data sources to identify suspicious activity & potential threats

example: someone steals credentials of an IAM user and starts making unusual API calls from unfamiliar location.

It uses Machine Learning algorithm, anomaly detection, 3rd party data

## Amazon Inspector
A service allowing you run automated security assessments
Automatically discovers and assesses supported AWS workloads for software vulnerabilities.

use-case: When you want to identify vulnerable software packages/exposed ec2 instances ...

For EC2 instances
	- leveraging your Systems Manager (SSM) agent
	- Analyze against unintented network access
	- Analyze the running OS against known vulnerabilities
For Container Images pushed to ECR (container registry)
	- They'd be analyzed for known vulnerabilities
For Lambda Functions
	- Identify vulnerabiilities in. function code andpackage  deps
	- Assessment of functions as they are deployed

Does reporting & integration with AWS Security Hub and sends findings to Event Bridge

It does continuous scanning of the infrastructure (EC2, Image, Lambda), only when needed

---
## Amazon Macie
A fully managed data security & data privacy service using ML and pattern matching to discover & protect your sensitive data in AWS

Alerts you for sensitive data in your S3 bucket such as PII

For the exam, think of macie as the service for discovering & protecting sensitive data in S3.

Macie is primarily focused on sensitive data in Amazon S3, while GuardDuty detects threats and then Inspector finds software vulnerabilities

The easiest way to remember the three is:
- Macie: What sensitive data am I storing, and is it exposed?
- GuardDuty: Is someone or something behaving maliciously?
- Inspector: Does my software or infrastructure have vulnerabilities attackers could exploit?

---

# **AWS Artifact**
Provides on-demand access to AWS security and compliance report, and access to certain agreements with aWS

use-case: your auditor or customer requests evidence of AWS compliance with standards such as ISO 27001 or SOC

This is your go-to, central resource for compliance-related information that matters to you. It provides on-demand access to AWS’s security and compliance reports and select online agreements. Reports available in AWS Artifact include our Service Organization Control (SOC) reports, Payment Card Industry (PCI) reports, and certifications from accreditation bodies across geographies and compliance verticals that validate the implementation and operating effectiveness of AWS security controls. Agreements available in AWS Artifact include the Business Associate Addendum (BAA) and the Nondisclosure Agreement (NDA).

---
# AWS Security Hub
Aggregates and prioritizes security findings from supported AWS security services and checks your environment against supported security standards.

use-case: when you need a consolidated view of security risks and compliance posture across AWS account

AWS Guard duty detects threats, while AWS Security Hub centralizes security findings and helps you assess your overall security posture