Some useful whitepapers to skim through include:
- Architecting for the Cloud: AWS Best Practices
- AWS Well-Architected Framework
- AWS Disaster Recovery
# Guiding Principles
- Stop guessing capacity needs
- Test system at production scale
- Automation makes architectural experimentation easier
- Allow for evolutionary architecture as requirements change
- Drive decisions using data
- Improve through game days

# Design Principles
- Scalability - vertical and horizontal
- Disposable resources - servers should be disposable so you can replace infrastructure
- Automation - IaC, serverless, auto-scaling
- Loose coupling - failure in one component should not cascade to failure in others
- Services, not servers - what managed services are available to take workload away from you

# The 6 Pillars
## 1) Operational Excellence
#### Prepare, Operate, Evolve
- All operations should be as code
- Annotate documentation
- Make frequent, reversible changes
- Refine operations frequently
- Anticipate + learn from failure

## 2) Security
#### IAM, Detective Controls, Infrastructure Protection, Data Protection
- Strong identity foundation
- Traceability
- Secure at all layers
- Automate security best practices
- Keep people away from data
- Prepare for security events

## 3) Reliability
#### Foundations, Change Management, Failure Management
- Test your recovery procedures
- Automatically recover from failover
- Scale horizontally over smaller instances
- Use auto-scaling to match demand
- Automate changes to infrastructure
Think *Config, CloudTrail, CloudWatch*

## 4) Performance Efficiency
#### Selection, Review, Monitoring, Tradeoffs
- Use advanced/specialised technology
- Go global in minutes
- Serverless architecture
- Experiment frequently
- Be aware of all available services

## 5) Cost Optimisation
#### Expenditure Awareness, Cost-Effective Resources, Match Supply And Demand, Optimise Over Time
- Adopt consumption model
- Measure efficiency
- Reduce overheads on database operations
- Analyse and attribute expenditure
- Use managed + app levels services to scale effectively

## 6) Sustainability
- Understand your impact
- Establish sustainability goals
- Anticipate to adopt new, more efficient offerings
- Use managed services
- Reduce downstream impact required for customer use

# Example Architectures
See https://aws.amazon.com/architecture and https://aws.amazon.com/solutions for additional guides.
## Event Processing
#### Lambda, SQS, SNS
- Lambda polls SQS queues for a certain number of retries before a message is sent to the dead-letter queue.
- Lambda polls FIFO SQS queues for a certain number of retries, but the FIFO queue will block any attempts to poll messages out of order. Therefore, the queue will become blocked if the next message cannot be processed and messages will begin to pile up in the dead-letter queue.
- SNS is sent asynchronously to Lambda which will process the message for a set number of retries, after which the message can be sent to a dead-letter queue in SQS.
- Fan-out pattern: send your PUT HTTP requests to an SNS topic which will then distribute the messages to any SQS queues subscribed to that topic. This makes your SQS queues more reliable because they are receiving the exact same messages on each and it doesn't matter if one queue goes down, the others will still be running.
#### S3 event notifications
- When an event of a certain type is triggered in S3 e.g. `S3:ObjectCreated`, you can push messages to an SNS topic/SQS queue/Lambda function/EventBridge bus + connected services. Delivery times can vary from a few seconds to a minute or longer.
#### API call interception
- CloudTrail logs all API calls made in AWS which can be forwarded on to EventBridge which can then trigger an alert by sending a message to an SNS topic.
#### External event notification
- A client server may make a request to API Gateway which will send the messages to Kinesis Data Stream which can then forward any records on to Kinesis Data Firehose which generates files to be stored in S3.

## Caching Strategies
- CloudFront - caches data at the Edge, lowest latency required to fetch data as data is as close to the user as possible.
- API Gateway - can cache data at the point where API requests are received which happens at a regional level, slightly more lag than querying the cache at the Edge due to the distance between the user and the gateway.
- ElastiCache (Redis/MemCached)/DynamoDB Accelerator (DAX) - caches frequent queries made to your databases to save on the computation time of the services running your application logic.
Higher TTL means the cache will be retained for longer.
Additional caching capabilities will incur more cost but will reduce latency for your user.

## IP Blocking
- NACL - protects your apps at the VPC level and is cheap to implement.
- CloudFront - resides outside your VPC and can then forward requests on to your ALB inside your VPC. However, NACLs don't provide any protection when everything goes through CloudFront as you need to allow all of the public CloudFront IPs in order to use it.
- ALB - protects your app inside the VPC using connection termination to break up the request from going directly to your instances. 
- WAF - can install WAF on the ALB/CloudFront distribution at additional cost to filter based on IP address filter rules such as geo restriction.
- NLB - protects your app inside the VPC by controlling the traffic allowed to pass through it.
- Security Group - protects your apps/load balancers as requests reach them.
- Firewall software - can be installed inside your EC2 instances at an additional computational cost.

## High Performance Computing (HPC)
#### Data Management & Transfer
- Direct Connect - move GBs of data quickly over a private connection.
- Snowball/Snowmobile - move PBs of data to AWS via the movement of physical devices.
- DataSync - move a large amount of data from on-prem to S3/EFS/FSx for Windows.
#### Compute
- CPU/GPU optimised instance types
- Spot instance fleets for savings + quick auto-scaling
- Placement groups provide better network performance as the instances can be housed in the same rack/AZ for low latency (up to 10GB/s).
#### Network
##### EC2 Enhanced Networking (SR-IOV):
- Higher bandwidth + packet per second, lower latency.
- Option 1) Elastic Network Adapter (ENA) - up to 100GB/s
- Option 2) *LEGACY* Intel 82599 VF - up to 10GB/s
##### Elastic Fabric Adapter (EFA)
- Improved version of ENA, but only works on Linux.
- Good for inter-node communication like in tightly coupled workloads.
- Uses Message Passing Interface (MPI) standard to achieve its speeds.
- Bypasses Linux OS for low-latency, reliable transport.
#### Storage
##### Instance-Attached Storage
- EBS - scale up to 256,000 IOPS when using io2 Block Express.
- Instances Store - scales to millions IOPS with low latency.
##### Network Storage
- S3 - blob storage instead of file system for large object storage.
- EFS - scales IOPS based on total size or can provision a set IOPS.
- FSx for Lustre - optimised for HPC with millions of IOPS and is backed by S3
##### Automation & Orchestration
- Batch - sallows you to schedule jobs and EC2s with support for multi-node parallel jobs for running single jobs that span multiple EC2s.
- Parallel Cluster - cluster management tool designed to assist HPC in AWS. Automates the creation of VPC, subnets, cluster types and instance types and can also use EFA for improved network performance.

## EC2 High Availability
#### Manual Setup
1. Lets say you have a single EC2 running your application at a time with an Elastic IP attached to the front of it and a standby EC2 in a different AZ that is not active.
2. When something causes the initial EC2 instance to terminate/use a high CPU percentage, for example, this information can be forwarded onto CloudWatch Event/Alarm (if based on metric).
3. This can then be used to trigger a Lambda function that will start the standby EC2 instance and re-assign the Elastic IP to it.
#### Auto-Scaling
1. Set up an ASG across 2 or more AZs with EC2 instances, the user data of which will attach the Elastic IP to that instance as it starts.
2. As one instance is terminated or reaches a CPU threshold, a new instance can be spun up in another AZ and will attach itself to the Elastic IP as the first instance shuts down.

If we want to preserve the EBS volumes attached to the instances (which are locked to a single AZ), we can set up a termination lifecycle hook that will run a script that creates a snapshot which can then be shared with the other AZ(s).
You would then need a launch lifecycle hook to be created on the ASG to create a new EBS in the additional AZ from the snapshot and attach it to the newly spun up EC2.

# Well-Architected Tool
Used to evaluate your architecture against the 6 pillars based on your responses to various questions.
You choose a lens to evaluate your architecture against e.g. well-architected, SaaS, Serverless, Foundational Technical Review.
Advice is then provided based on the responses to your survey questions.

# Trusted Advisor
Analyses your accounts and provides recommendations on the following categories:
- Cost optimisation
- Performance
- Fault tolerance
- Security
- Service limits
- Operational excellence
Gives you different types of support plan: Basic, Developer, Business, and Enterprise.
Basic and Developer have 7 core checks (some security + service limits checks):
- S3 bucket permissions
- SG port restrictions
- One IAM user per account
- MFA enabled on root user
- No EBS public snapshots
- No RDS public snapshots
- Service limits
Business and Enterprise have full core checks:
- Full checks on all 5 categories
- Programmatic access to AWS Support API
- Set CloudWatch alarms when reaching service limits

# Right-Sizing
Cloud is elastic so start small and size up as you need.
Optimise costs and scale down unnecessary instances.

# Ecosystem
Lots of free resources available including:
- blogs
- forums
- Whitepapers/guides
- QuickStarts - automate deployments from templates
- Solutions - vetted technology solutions

Can also get varying levels of AWS Support depending on your plan.

Marketplace - buy software listings from 3rd party vendors such as AMIs, CloudFormation templates etc.

Training is available in many formats and for specific use case e.g. government, enterprise etc.

# Professional Services and Partner Network
Professional Services and Partner Network are a team of SMEs to help you work in the cloud.
APN Technology Partners - hardware, connectivity, software. Usually part of a broader customer solution team.
APN Consulting Partners - professional services to build on AWS
APN Training Partners - find trainers to help you learn AWS
AWS Competency Programme - competencies are granted to APN Partners who have demonstrated technical proficiency and have proved customer success in specialised areas

# Knowledge Centre
Source for answers to FAQs

# IQ
Quickly find AWS-certified experts to help you with your project - similar to a freelancer platform.

# re:Post
AWS-managed Q&A service similar to Stack Overflow.
Premium Support customers will receive support from engineers if the community is not able to help

#aws