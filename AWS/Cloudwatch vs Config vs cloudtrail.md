
Using An Elastic Load Balancer as a case study:

CloudWatch
	Monitors incoming connection metrics
	Validates error codes as a % over time
	Make a dashboard to get an idea of your load balancer performance

Config: This tells you what the resource looks like and whether its configuration follows the rules
	Track security group rules for the load balancer
	Track configuration changes for the LB
	Ensure an SSL certificate is always assigned to the LB(compliance)

CloudTrail: tells you what happened
	Track who made any changes to the LB with API calls
	changes in Security group/IAM permissions

*imagine someone changes a security group and accidentally opens ssh to the entire internet*

AWS CloudTrail – the activity log
Answers: Who did it?  What action did they take? When?
example: User `Alice` called `AuthorizeSecurityGroupIngress` API at 10:32PM.

AWS Config – the configuration inspector
Answers: What is the resource configured like? Does it comply with the rules
Example: The security gorup now allows SSH from 0.0.0.0/0, violating a rule that prohibits unrestricted SSH access.


---
Amazon CloudWatch has available Amazon EC2 Metrics for you to use for monitoring CPU utilization, Network utilization, Disk performance, and Disk Reads/Writes. In case you need to monitor the below items, you need to prepare a custom metric using a Perl or other shell script, as there are no ready-to-use metrics for the following:

Memory utilization

Disk swap utilization

Disk space utilization

Page file utilization

Log collection