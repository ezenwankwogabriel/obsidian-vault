You have an app running in prod, your manager calls you by 2am and says its not working. You forgot to turn on "Monitoring"

## CloudWatch Metrics (numerical measurements)
These collect numerical measurements over time, such as EC2 CPU Utilization, request counts, Lambda errors, and API latency

CloudWatch provides metrics for every service in AWS

Metrics belong to namespaces. Dimension is an attribute of a metric (an instance id, environment, ...)
There are up to 30 dimensions per metric. Metrics have timestamp.

You can have custom metric (for the RAM for example)

The idea is to constantly stream CloudWatch metric to a destination of choice, with near real-time delivery & low latency. For this we can use "Amazon kinesis Data Firehose". 
Or also to 3rd party service providers like Datadog, Dynatrace, Splunk, Sumo logic, ...

## CloudWatch Logs
A place to collect and store log events from applications and supported AWS services.
#### Log Group
You have to first define "Log Groups", which are arbitrary names representing an application

You can create one group for payment API, another for auth API, another for backgroud jobs. 
#### Log Stream
ex: logs from one lambda execution env / one EC2 application file

Multiple streams can exist inside a single group

Within a Log Group, you'd have "Log Stream", which are instances within your app/logfiles/container
#### Log Event
Example: `2026-10-21 09:45:02 ERROR Payment failed: database timeout`

An event contains a timestamp and a log message.

Log group -> Log Stream –≥ Log events


These logs are encrypted by default. and you can setup kms-based encryption with your own keys

#### How do logs get into CloudWatch Logs
CloudWatch logs doesn't auto collect every log from every machine. It all depends on the source

1\. EC2 App. You app writes log to files/standard output. To collect log files from EC2, you install and configure the "CloudWatch Agent", granting it IAM permissions to collect specific files
	CloudWatch Logs Agent
		Old version of the agent
		Can only send to CloudWatch Logs
	
	Cloudwatch Unified Agent
		Collect additional system-level metrics like Ram, process, ...
		Send logs to CloudWatch Logs
		Can do a centralized config using SSM params for all agents

2\. AWS Lambda: Lambda sends supported function logs to CloudWatch logs when its execution role has the required permission

3\. API Gateway: Configure execution / access logging for the relevant API type and stage

Logs Insight lets you query and analyze these logs (find payment API errors in last 15 minutes). Remember, this is just a query engine, not a real time engine

Use "CloudWatch Log Subscriptions" to get real-time log events from CloudWatch Logs for processing and analysis

### Practically
Your app sends messages to Cloudwatch Logs
You open cloudwatch > Logs > Log groups > /payment/production
Select the log stream and inspect d event

### Turn log to an alarm
A\. Metric Filter could count log events containing Error, an alarm could notify you when the count exceeds a threshold. It only consider logs from after its creation

B\. CloudWatch Logs anomaly detection and alerting
You can use supported-log monitoring features and anomaly-detection capabilites to identify unusual behavior. 

Alarm States: 
	OK: metric is within configured threshold 
	INSUFFICIENT_DATA: doesn't have enough data to determine the stat
	ALARM: metric has breached the configured threshold


# CloudWatch Network Synthetic Monitor
For when you've connected your cloud account to an on premises server through DX or VPN connection and you'd like to monitor and detect network issues btw your apps hosted no AWS and your premises data center
This helps test ICMP or TCP traffic
This majorly guarantees your connection is working as expected, and it gives you the information in realtime

