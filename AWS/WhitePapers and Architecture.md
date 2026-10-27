# AWS Well Architected Framework & Tool
The AWS Well-Architected Framework is a set of best practices for designing and operating workloads on AWS so they are secure, reliable, efficient, cost-effective, and sustainable.

For AWS SAA-C03, you need to understand its six pillars, the problems each pillar addresses, and which AWS solutions fit common scenarios.

## The six pillars

## 1. Operational Excellence

Question: Can you operate, monitor, and improve the workload effectively?

- Monitor applications and infrastructure.
- Automate deployments and routine operations.
- Learn from failures and improve processes.

AWS examples: CloudWatch, CloudTrail, Systems Manager, Infrastructure as Code.

## 2. Security

Question: How do you protect data, systems, and access?

- Apply least-privilege permissions.
- Encrypt data at rest and in transit.
- Detect threats and maintain traceability.

AWS examples: IAM, KMS, CloudTrail, GuardDuty, AWS WAF.

## 3. Reliability

Question: Can the workload recover from failures and continue operating?

- Design for component failures.
- Back up data and test recovery.
- Scale and recover automatically where appropriate.

AWS examples: Multi-AZ, Auto Scaling, Route 53 health checks, AWS Backup.

## 4. Performance Efficiency

Question: Are you using the right resources and architecture for the workload?

- Select appropriate compute, storage, and database services.
- Use scalable architectures and managed services.
- Continuously evaluate newer technologies.

AWS examples: EC2 instance selection, Lambda, CloudFront, ElastiCache.

## 5. Cost Optimization

Question: Are you getting the required business value without unnecessary spending?

- Right-size overprovisioned resources.
- Use appropriate purchasing models.
- Eliminate idle resources and review spending.

AWS examples: Cost Explorer, Compute Optimizer, Savings Plans, Spot Instances.

## 6. Sustainability

Question: Are you minimizing the environmental impact of your cloud workloads?

- Avoid unused or underutilized resources.
- Improve application and infrastructure efficiency.
- Use managed services and efficient architectures when appropriate.

Focus: resource utilization, efficient code, and reducing unnecessary compute and storage.

## How to distinguish the pillars in exam questions

A common exam trap is that one AWS solution can support multiple pillars. Identify the primary problem described in the question, rather than matching a service to only one pillar.

| If the question emphasizes…                               | Think…                 |
| --------------------------------------------------------- | ---------------------- |
| Automated deployments, monitoring, continuous improvement | Operational Excellence |
| IAM permissions, encryption, threat detection             | Security               |
| Recovery, fault tolerance, backups, availability          | Reliability            |
| Latency, throughput, choosing the right technology        | Performance Efficiency |
| Lower bills, eliminating waste, purchasing options        | Cost Optimization      |
| Resource efficiency and environmental impact              | Sustainability         |

### Example: the same action, different pillars

Suppose an application runs on oversized EC2 instances.

- If the goal is to reduce the AWS bill, that's primarily Cost Optimization.
- If the goal is to select instance types that meet workload performance needs, that's primarily Performance Efficiency.
- If the goal is to automatically adjust capacity when demand changes, it could support both Performance Efficiency and Cost Optimization.

The wording of the business objective determines the best answer.

## The Well-Architected Tool

The AWS Well-Architected Tool helps you review a workload against AWS best practices, identify risks, and track improvements. It organizes reviews around the six pillars and includes questions, best practices, and improvement plans.

Do not confuse it with a monitoring service: CloudWatch monitors operational metrics and logs, while the Well-Architected Tool helps assess architectural decisions.


Memory aid: Operational Excellence = run it well; Security = protect it; Reliability = keep it working; Performance Efficiency = make it perform well; Cost Optimization = avoid waste; Sustainability = reduce environmental impact.

---
# AWS Trusted Advisor
This is a service that gives a high level AWS account assessment. Checks like for snapshots, IAM use, ...
Grouped in 6 categories:
- Cost Optimization
- Performance
- Security
- Fault Tolerance
- Service limits
- Operational Excellence
There is a Business and Enterprise Support plan