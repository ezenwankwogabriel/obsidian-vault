It answers the question: "How are my AWS resources configured, and how have those configurations changed over time"

Think of this as an inspector and history book for your AWS infrastructure

Helps with auditing and recording compliance of your AWS resources based on rules you set

Example: you have an S3 bucket storing sensitive data. You want to know
- is the bucket publicly accessible?
- is encryption enabled?
- who or what changed its config?
- which resources were noncompliant last week?


Your AWS resources (S3 · EC2 · Security Groups · RDS · IAM resources (where supported))
				||
AWS Config (Records configuration changes and maintains configuration history)
				||
Configuration history (What changed, and what did the resource look like before?)
and
Compliance evaluation (Does the resource satisfy the rules?)

Then based on the rules, you can receive alerts (SNS notifications) for any changes
This is a per-region service
Can save the config in S3, to be analyzed by Athena

Can use AWS managed config rules or
Can make custom config rules (these must be defined in AWS Lambda)

NB: These are just compliance, and do not prevent actions from happening (no deny)

You can do/automate remediations (not deny) of non-compliant resources using SSM Automation Documents

A rule can report `COMPLIANT`, `NON_COMPLIANT`, `ERROR`, or `NOT_APPLICABLE`.

ex: IAM Access keys are expired (older than 90 days -> Non Compliant), you trigger a remediation action (SSM Document : AWS Config Remediate-RevokeUnusedIAMUserCredential).
The document invokes a lambda function, that can do whatever you wish to do

Example:
Step 1: Deploy your resources (create EC2, S3 bucket, ...)

Step 2: Enable AWS Config (choose the AWS resource you want Config to record. Config records their supported configuration changes)

Step 3: Set up Config rules (ex: use a managed rule to check whether EBS volumes are encrypted. these rules exist already, just select if aWS already provides one)

Step 4: Review Compliance (Config reports whether resources are compliant and helpts you investigate configuration history)