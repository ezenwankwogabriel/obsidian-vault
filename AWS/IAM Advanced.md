## AWS Organization
This is a global servcie allowing you manage multiple AWS accounts

The main account is the **management** account, while other joining accounts are called **member** accounts

Billing is consolidated across all account (single payment method) [pays for all orgs cost]
Hence you get volume discounts

You can share reserved instances and savings plans discount

Advantages:
- Multi Account vs One Account Multi VPC (gives you better security, as accounts are more separated)
- Use tagging standards for billing
- Send CloudTrail, CloudWatch logs to central logging point
- Establish cross account roles for admin purposes
- You can assign "**Service Control Policies**" (SCP)
	- these are IAM policies applied to OU or Accounts to restrict Users and Roles
	- These do not apply to the management account
	- Explicit allows are required for member OU, else they cannot do anything. "FullAWSAccess + Deny EC2"

---
## IAM Conditions
These let you control when an IAM policy statement applies. Extra rules to Allow/Deny decision, such as requiring specific IP address, MFA, region, tag or time

Think IAM policy as a security guard:
>Effect: Should access be allowed / denied?
>Action: What are you trying to do? (s3:getobject)
>Resource: Which AWS resource are you accessing? (resource arn)
>Condition: Under what circumstances can you do it?

example: allow an employee access to S3 only on company IP address

```
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::company-files/*",
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "203.0.113.10/32"
    }
  }
}
```

---
## IAM Roles vs Resource Based Policies
For a Cross Account: when you want to perform an API call on an S3 bucket cross account, you have two options.
	You can attach a resource-based policy to a resource. for example, an S3 bucket policy on an S3 bucket
	or you can decide to use a role that can actually access the resource.

The key is that we want the least access. and a role gives more permission. so we prefer a resource policy.

So always, if there is a resource policy, go for that (ex: bucket policy). else can issue a role

## IAM - Permission Boundaries
These are use to set the maximum policy an IAM entity can get. 
They are for users & roles (not groups)

You get!!? these are policies applied to actual users.
For example: Permission Boundary for a user => Action: ["S3:*", "cloudWatch:*"] grants the user only this two action access even if granted Admin/all access permisson by default. the boundary takes precedence.

If with the above, you now grant an IAM Permission through "IAM Policy" for this user who was granted 2 user permissions. The IAM Permission of: Action: "iam:CreateUser"

The above results to "No Permission" as the user permission only grants the user S3 & CloudWatch

---
## IAM Identity Center
A service that lets employees sign in once, and access the AWS accounts & apps they have permissions to use. One log in for all your AWS account in AWS organizations.

Imagine you're the IT Admin of a company with 10 AWS accounts and 50 developers

Without centralized access management, you'd have to manage access separately for every account. This becomes difficult to maintain, mostly when employees leave/change team/ join

IAM identity center gives a central place to manage who can access which accounts & what they can do there

access to biz cloud orgs (salesforce, box, microsoft 365, ...), SAML 2.0-enabled app, EC2 window instances

#### How Identity Center Works
1\. Identity Sources/Providers –– who are you
IAM identity center needs to know who is signing in. They store/provide identities used to authenticate employees. where your users are stored for this login.

It can use:
>Built-in identity directory
>Microsoft Active directory
>External identity provider, such as Microsoft Entra Id or Okta

2\. Users & groups –– who belongs to which team
You can organize employees into groups (Developers & Finance) instead of configuring each person individual. When a new member joins a group, he receives the access assigned to that group

3\. Permission set
A reusable collection of permissions for accessing an AWS account

>Fine-grained permissions: This is a reusable template containing the permissions a user receives when accessing an AWS account

Example: 
	Readonly: view resources without modifying them
	Developer: Perform approved dev tasks
	Administrator: Broad admin access
	ViewCloudWatchLogs: Read objects in S3 bucket
	
> Fine-grained access using ABAC (Attribute-Based Access Control)
> This allows AWS make access decisions based on attributes associated with the user & resource. you can pass "user attributes" in IAM Identity Center. Attributes like "Payments", "Inventory", ...
	
4\. Assignments –– where can you go?
Connects groups/users, an AWS account / support app, and relevant access configs

These determine who gets access to which AWS accounts/applications

For AWS account access, the permission set is part of that assignment.

For example:

| Group      | Account     | Permission set   |
| ---------- | ----------- | ---------------- |
| Developers | Development | Developer        |
| Developers | Testing     | ReadOnly         |
| Finance    | Finance     | Finance-ReadOnly |

5\. Temporary credentials –– how do you access AWS?
When John (a Developer) signs in and chooses an account (ex: Finance), IAM identity center facilitates access using a temporary AWS credential. 

AWS provisions an IAM role in the target account for the permission set. John assumes the role, and receives temp credentials to make authorized AWS request

He isn't granted a separate long-lived IAM user / access key for every AWS account he chooses

This is the main security benefits: centralized access management and temporary, role-based credentials.

### Active Directory (Microsoft Directory Service)
Microsoft AD is a software that's found on any Windows Server with AD Domain services

AD DS is Microsofts core directory service that stores network object information & manages user access, identity, and security policies across windows-based networks

It is used by organizations to centrally manage users, computers, auth, and access to company resources.

Imagine a company with 500 employees. Instead of creating separate login account on every employee's computer and every internal server, AD DS is used, to manage identities centrally.

An employee can sign in with a company account, and the org can enforce policies (like password requirements, expiry, computer restrictions, and access to shared files)

#### Core Concepts
1\. Domain: A logical collection of users, computers, and resources managed under a common directory and security boundary, such as "corp.example.com"

2\. Domain Controller (DC): A windows server running AD DS. It stores directory data and handles domain auth and related directory operation

3\. Organizational Units (OU) and Group Policy: OUs organize directory objects. Group Policy lets admin centrally configure supported user and computer settings

How authentication works:
>Employee signs in to a domain-joined computer
>The computer contacts a domain controller
>AD DS validates the user's credentials, using Kerberos in a Windows domain
>The user receives auth info and can access permitted resources

#### Relationship to AWS (3 core options)
1\. AWS Managed Microsoft AD (Actual Microsoft AD):
AWS operates Microsoft AD Domain Controller (server running AD DS) for you in AWS. You manage users, groups, and directory settings, while AWS manages much of the underlying infrastructure.

use-case: when your app needs real MS AD features, or you need an AWS-hosted directory with trust relationships to your existing on-premises AD (i.e they share trust in auth and other related services)

2\. AD Connector (Proxy to existing AD)
This Connects compatible AWS services to your existing on-premise AD and forwards request to it. Doesn't cache directory info in AWS

use-case: when your company already has AD and wants employees to use their existing credentials with supported AWS Service

3\. Simple AD (Basic AD compatibility)
A lower cost, AWS-managed directory based on Samba 4, providing a subnet of AD functionality.

use-case: you need basic directory features without full capabilities of Microsoft AD

#### IAM Identity Center integration with Active Directory
1\. Connecting to Identity Center using the AD -> AWS Managed Microsoft AD (managed on AWS) then integration is out of the box

2\. If you have a self managed directory. you have two was for this:
	You'd have to create a Two-way Trust relationship using AWS Managed Microsoft AD
		Youd create a directory on AWS Managed MS AD, then create the two-way trust relationship btw AWS and you on-premise
	Create an AD connector, and proxy onto your AWS services

---
# AWS Resource Access Manager (RAM)
This is a service that enables you to easily and securely share AWS resources with any AWS account or within your AWS Organization. You can share AWS Transit Gateways, Subnets, AWS License Manager configurations, and Amazon Route 53 Resolver rules resources with RAM.

Many organizations use multiple accounts to create administrative or billing isolation, and limit the impact of errors. RAM eliminates the need to create duplicate resources in multiple accounts, reducing the operational overhead of managing those resources in every single account you own. You can create resources centrally in a multi-account environment, and use RAM to share those resources across accounts in three simple steps: create a Resource Share, specify resources, and specify accounts. RAM is available to you at no additional charge.

---
## AWS Control Tower
An easy way to setup and govern a secure and compliant multi-account AWS environment based on best practices

Think of this as a company headquaters

Imagine AWS Organization manages all company's branches, while Control Tower establishes the rules every branch must follow.
it uses AWS organizations to create accounts

#### Landing Zone: the foundation
A landing zone is the overall, preconfigured multi-account aWS env that Control Tower establishes and governs. It can be setup in a new / existing AWS organization.

An Account Factory is Control Tower's standardized account-provisioning capacity. Suppose a team needs a Testing Account. Rather than manually creating one with all configs, an admin can use Account Factory to provision an account.

Exam clue: If a question asks how to provision new AWS accounts consistently within a governed multi-account environment, think AWS Control Tower Account Factory.

#### Controls (formerly called guardrails)
This is a governance that helps prevent, detect or proactively block policy violations across accounts. 

1\. Preventive Controls: Stop the action from happening
example: prevent users in governed accounts from disabling a protected security configuration

2\. Detective controls: Detect a problem after a resource or configuration exists
example: detect an S3 bucket that permits public access

3\. Proactive controls: Checks supported deployments before resources are created
example: reject a noncompliant resource deployment made through an applicable CloudFormation template

*NB* AWS Orgs can manage accounts & SCP(service control policy) without "Control Tower". Control Tower is the higher-level setup and governance service, not a replacement for Organizations


Remember:

- `aws:RequestedRegion` = which AWS endpoint is called?
- `aws:SourceIp` = where is the request coming from?
- `aws:ResourceTag` = what tag is on the resource?
- SCP = what actions can be permitted within an AWS account?