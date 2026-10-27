This provides governance, compliance & audit for your AWS account
Is enabled by default
Can get history of events/API calls made within your AWS account
Can put logs from CloudTrail into CloudWatch log or S3
A trail can be applied to all regions (default) or single region

Say an EC2 instance was deleted, and we need an answer to who carried out the action:> CloudTrail helps find out

## Events
1\. Management Events
	Operations performed on resources by your AWS Account
	Are configured to trail such events by default
	Can separate Read Events from Write Events
2\. Data Events
	Are not logged by default
3\. Insights Event

