## 1. AWS resources that are typically private

These resources are commonly deployed inside a VPC and do not need direct internet exposure.

- EC2 instances (depending on subnet,IP,routing)
- RDS / Aurora (placed in private subnet)
- ElastiCache
- Redshift
- EFS
- ECS/EKS workloads

They generally rely on private IP and VPC networking

## 2. AWS resources and services with public endpoints

Many AWS services expose public service endpoints. This does not mean your data or resources are publicly accessible.

| AWS service     | Public endpoint available? | Can be accessed privately?                  |
| --------------- | -------------------------- | ------------------------------------------- |
| Amazon S3       | Yes                        | Yes, Gateway or Interface Endpoint          |
| DynamoDB        | Yes                        | Yes, Gateway or Interface Endpoint          |
| SQS             | Yes                        | Yes, Interface Endpoint                     |
| SNS             | Yes                        | Yes, Interface Endpoint                     |
| AWS STS         | Yes                        | Yes, Interface Endpoint                     |
| Secrets Manager | Yes                        | Yes, Interface Endpoint                     |
| KMS             | Yes                        | Yes, Interface Endpoint                     |
| CloudWatch      | Yes                        | Yes, Interface Endpoints for supported APIs |
| ECR             | Yes                        | Yes, Interface Endpoints                    |
| Lambda API      | Yes                        | Yes, Interface Endpoint                     |