Think this as an event router. It receives events from AWS services, your apps, ... checks them against matching rules, then routes matching events to appropriate targets (lambda, sns, sqs, step functions)

Example1: an EC2 instance changes state, eventbridge can match that event and trigger a Lambda function, notify an operation workflow, or send a message to an SQS queue

example2: IAM Root user sign in to the console, trigger lambda that calls sns to send a message

It is a regional, managed AWS service that's configured in your AWS account. 
You create an event bus, rules, optional schedules
You then connect those rules to event sources and targets

### EventBridge vs Cloudwatch

EventBridge routes events to targets and coordinates event-driven actions
	This is for Event routing and event-driven architecture
	NB: A specific AWS service / application event is typically an EventBridge rule use case
	
	ex: An EC2 instance changes state to stopped
	
Cloudwatch monitors metrics, logs and operational health, and can trigger alarms when conditions are met
	Typically for monitoring and observability
	ex: CPU exceeds 90% for 5 minutes
	NB: A metric threshold is typically a Cloudwatch alarm use case.

## EventBridge Event buses
1\. Event Buses: Receives events. 
The. main types are:
	default event buses: where enventbridge serves as the bridge for diff aws services (all aws events lands here)
	
	partner event buses: when event bridge serves as the bridge for partner services (so you can react directly to changes outside of AWS in your account)
	
	custom event buses: you can create your own event buses and your own apps can send their events to your own custom event bus

Event buses can be accessed by other AWS accounts using "Resource-based policies"

You can archive events sent to an event bus (indefinitely or set period)
You can replay archived events

The above comes in handy if you need to revisit certain events

You store the events to be received by event bridge using Schema Registry

## CloudWatch Container Insights
A way to collect, aggregate, and summarize metrics & logs from containers
Available on Amazon ECS, EKS (kubernetis), kubernetes platforms on EC2, Fargate

Key: Cloudwatch Insight uses a containerized version of CloudWatch Agent to discover containers

## CloudWatch lambda Insights
Runs next to your lambda, to monitor performance of your serverless apps running on AWS Lambda

## CloudWatch Contributor Insights
- Find top talkers and understand who is impacting system performance
- you can identify heaviest network users, bad hosts, urls that generate most error