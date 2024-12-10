## CloudWatch Metrics
Get metrics of any AWS services e.g. billing, CPU usage, status checks, network, reads/writes, other general usage metrics. All billing data is stored in the **US East (N. Virginia) - us-east-1** region.
You can monitor a metric and each metric will have up to 30 dimensions e.g. instance ID, environment etc.
Each metric will have a timestamp associated with it.
CloudWatch can be used to create dashboards of metrics.
You can also create custom metrics.
#### Metric Streams
Offers near-real-time delivery with low latency to services such as Kinesis Data Firehose or 3rd party services such as Datadog, Dynatrace, Splunk...

## CloudWatch Logs
Collect logs from different services to see how they are being used and monitor them in real-time.
Need to run CloudWatch agent on instances in order to push logs to CloudWatch logs.
Encryption enabled by default.
Log are collected within log groups which can be given an arbitrary name that usually represent an application.
Inside log groups, you have log streams that represent log instances within an application/log files/logging containers. 
You set an expiration date on your logs e.g. never expire, 1 day, 10 years
Logs can be forwarded on to S3, Kinesis Data Streams/Firehose, Lambda, or OpenSearch.
Logs are encrypted by default and you can set up KMS-based encryption using your own keys.
#### Sources
Logs can come form the following sources:
- SDK/CloudWatch Logs Agent/CloudWatch Unified agent
- Elastic Beanstalk - application logs
- ECS - container logs
- Lambda - function logs
- VPC - flow logs
- API Gateway
- CloudTrail based on a filter
- Route53 - DNS queries
##### CloudWatch agents
To send logs from EC2 instances or on-prem servers, you need to install the Cloudwatch agent and configure an IAM role to allow the agent to push logs to CloudWatch.
the CloudWatch Logs Agent is the older agent that is capable of sending logs to CloudWatch.
The CloudWatch Unified Agent is the newer agent that is capable of sending logs and metrics (CPU, disk metrics, RAM, netstat, processes, swap space) to CloudWatch. It is also capable of having its configuration stored in SSM Parameter store.

#### Log Insights
Querying capability inside CloudWatch Logs that produces a visualisation and can be exported to a file or a dashboard.
CloudWatch Logs comes with some pre-written queries for you to use.
Can query multiple log groups from multiple AWS accounts.
Can only query historical data i.e. not in real-time.
#### S3 Export
Can be called via the CreateExportTask API to batch upload logs to S3.
However, logs only become available up to 12 hours after they are generated.
#### Subscriptions
Can send logs in real-time to Kinesis Data Streams/Firehose, or Lambda.
You can filter the logs sent by each subscription to their respective destinations.
It's possible to aggregate CloudWatch Logs from multiple accounts in different regions so that a consuming service receives all of the relevant logs as one subscription. In order to do this, you need to create a subscription filter in your source account, a subscription destination in your destination account, a destination access policy + IAM that allows logs to be put onto the consuming service. The IAM role in your destination account needs to be assumable from the source accounts.

## CloudWatch Alarms
Trigger alerts based on metrics.
There are 3 alarm states: OK, INSUFFICIENT_DATA, ALARM
You can set a period to evaluate your metric at e.g. every 10s, 30s, or x minutes.
#### Targets
There are 3 main targets:
- Stop, terminate, reboot, recover actions on EC2s.
- Auto-scaling trigger actions.
- Notifications being sent to SNS (which can be placed in front of most services).
Composite alarms monitor the states of multiple alarms using AND and OR conditions. These help to reduce alarm noise.
You can also create alarms based on Cloudwatch Logs metrics filters.
Alarms can be triggered manually using the CLI by setting to the alarm state to ALARM.

## CloudWatch Insights
#### Container Insights
Collects, aggregates, and summarise metrics and logs from container for containers on ECS, EKS, k8s platforms hosted on EC2s, and Fargate.
For EKS and k8s platforms on EC2, CloudWatch Insights is using a containerised version of the CloudWatch Agent to discover containers.
#### Lambda Insights
Collects, aggregates, and summarise system-level metrics e.g. CPU time, memory, disk, network.
Also collects diagnostic information such as cold starts and Lambda worker shutdowns.
It's provided as a Lambda layer.
#### Contributor Insights
Analyses log data and creates time-series that display contributor data.
Allows you to identify the top contributors to certain metrics so you can identify where the most activity, consumption, throughput is going.
Works for any AWS-generated logs.
You can build rules from scratch or use pre-made rules that leverage Cloudwatch logs.
You also have built-in rules to help you analyse metrics from other AWS services.
#### Application Insights
Automated dashboards that show potential problems with monitored apps to identify issues with the various components of your app and with the other AWS services it links to.

## EventBridge (formerly CloudWatch Events)
Can schedule events given by cron jobs or event patterns in services e.g. object uploaded to S3 bucket, sign in with IAM role etc.
Event patterns can be collected from AWS services, AWS SaaS partners, or custom applications via event buses.
#### Event Buses
Services will send their events to an event bus:
- For AWS services you can send them to the default event bus.
- For SaaS products that are partnered with AWS e.g. Zendesk or Datadog, you may be able to send events to an event bus dedicated to that service.
- You can also create a custom event bus for your application.
Event buses can be accessed by other AWS accounts using resource-based policies.
Events can be archived after they are sent to an event bus and retained for a defined length of time. Any events you are archiving can be replayed to assist with troubleshooting.
#### Schema Registry
The schema registry allows you to grab the layout of how data sent from EventBridge will be structured so that you know in advance what the JSON data is going to look like.
These schemas can also be versioned.
#### Resource-based Policies
Allows you to manage permissions per event bus by saying which events are allowed and from which AWS accounts/regions which is useful if you want to aggregate the events in your AWS organisation into a single AWS account/region.

## CloudTrail
History of events/API calls made within your AWS account useful for DevOps monitoring or for compliance reasons.
Can put the logs in either CloudWatch Logs or S3.
Get insights from events using CloudTrail Insights.
A trail can be applied to all regions (which is the default) or to a particular region.
When something gets deleted unexpectedly, look in CloudTrail.
Events are separated into three types of events:
- Management events - operations performed on resources in your account.
- Data events - operations performed within resources in your account.
- CloudTrail Insight events - continuously monitors write events to predict unusual events e.g. inaccurate resource provisioning, hitting service limits, bursts of IAM actions, gaps in periodic maintenance.
Management and data events can be separated into Read and Write events.
Events are retained within CloudTrail for 90 days. If you wish to analyse them after that time, you need to have saved them to an S3 bucket so that you can use Athena to analyse them afterwards.

## X-Ray
Difficult to debug distributed services as log formats can differ.
Allows you to identify bottlenecks in services.
Can pinpoint where failures are coming from.

## CodeGuru
ML powered service for automated code reviews (Reviewer) and application performance recommendations (Profiler).

## Service Health Dashboard
Displays service health in all regions for all AWS services.
Shows historical information about outages.

## Personal Health Dashboard
Displays personalised view of AWS services that *you* are using.
Shows how outages affect you directly and provides remedial actions.
Provides pro-active notifications to help plan for scheduled activities.

#aws #monitoring