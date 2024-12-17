Total cost of ownership (TCO) - power/cooling costs + server administration costs
Only the root IAM user can change the account name.
Resources can be created through the AWS management console or CLI as well as using CloudFormation.
## Root User
Only the root user can:
- Change account settings
- Close your AWS account
- Change/cancel AWS support plan
- Register as seller in Reserved Instance Marketplace

## Organisations
Manage multiple member accounts from a master account.
Gain benefits such as pooling instances + larger billing discounts as long as instances are in same AZ (more spend across individual accounts -> greater saving overall), single payment window.
Automate account creation using an API.

There are various multi-account strategies one could take e.g. one account per department, dev + test + prod, better resource isolation such as per VPC, to fit regulatory restrictions etc.
Logs from all accounts should be sent to a central S3 with CloudTrail Logs on a central account.
Operational Units (OU) allow you to group accounts i.e. many dev accounts in the one dev OU and a root OU used to manage all OUs.

Control privileges on each account using Service Control Policies (SCP).
SCPs are applied at the account/OU level and apply to all the users and roles within accounts. They must have explicit allow statements for services otherwise nothing will be allowed.
Start with an SCP to allow full access for the root OU then restrict privileges the lower down the OU/account tree.

Advantages includes:
- Better networking security as multiple accounts can have their own VPC rather than one account managing multiple VPCs in one place.
- Can enforce tagging standards for easier billing.
- CloudTrail on all accounts that send logs to a central S3.
- CloudWatch Logs can work from a central account for all other accounts.
- Can manage cross-account roles for admin purposes.

## Control Tower
Greater security and governance based on best practices.
Allows you to automate the set up of accounts in a few clicks.
Automates policy management using guardrails:
- Preventive Guardrail - use SCPs across your accounts e.g. to restrict certain regions.
- Detective Guardrail - uses Config e.g. to identify untagged resources.
Detects policy violations with remediation features as well.
Provides a dashboard for monitoring compliance.

## Pricing Models
- Pay as you go - remain agile, free to scale up and down as needed
- Save when you reserve - greater savings over time, predictably manage budgets, retention compliance
- Pay less by using more - volume based discounts
- Pay less as AWS grows - as AWS becomes more available and cheaper to maintain by economy of scale, you save
Free services (although you pay for resources created by the last three):
- IAM
- VPC
- Consolidated billing
- Beanstalk
- CloudFormation
- ASG
Some paid services have a free tier up to a point:
- t2.micro EC2 instance for up to a year
- S3, EBS, ELB, AWS Data Transfer up to a certain usage

#### Compute Pricing
EC2
- Charged for what you use e.g. number of instances and what type of instances, data processed, ELB running time, monitoring level
- Shared hosting models:
	- On-demand instances - minimum 60s running time, pay by time used
	- Reserved instances - 1-3 year commitment, ~75% discount, greater saving for upfront payments. Not a physical instance but a commitment to using EC2s for a period of time which reduces costs. Can purchase convertible RIs to accommodate changing workloads.
	- Spot instances - up to 90% saving, bid for instances
- Dedicated hosting model - on-demand, reserved for 1/3 years. Physical instance that is yours and only yours to use. Great for server-bound software licences.
- Savings plan - saving based on sustained usage
Lambda
- Pay per call
- Pay per memory duration
ECS + Fargate
	- Pay for resources created to run containers on

#### Storage Pricing
S3 (and EFS)
	Pay for number + size of objects stored, number of IOPS requests, data transferred *out* of S3 to other regions/Internet
EBS
	Pay for volume performance, storage used per month, data transfer, backups

#### Database Pricing
RDS
	Pay per hour used, database performance, purchase type e.g. on-demand, reserved, backups when your database is full, underlying volumes used, availability, number of requests and data transfer out

#### Content Pricing
CloudFront
	Pay for geographical location of content, number of edge locations, data transfer out, number of requests

Traffic within an AZ is free of charge as long as it is between private IPs, whereas travel between AZs/regions has a cost (lower for between private IPs).

## Savings Plan
Commit a certain budget per hour for 1/3 years.
EC2 savings plan - commit to usage of instance type families in a region.
Compute savings plan - commit to usage of EC2, Fargate, and Lambda. Most flexible
Machine Learning savings plan - commit to usage of ML services

## Compute Optimiser
Recommends how many compute instances to be using across EC2, EBS, Lambda, ASG for optimal performance, regardless of the cost implications.

## Billing + Costing Tools
The root user has to enable IAM access to billing information for IAM users to access it.
Estimating future costs in the cloud:
- Pricing Calculator - provides estimates of costs based on planned services and usage.
Tracking current/past costs in the cloud:
- Billing dashboard
- Cost allocation tags - group costs by tag e.g. owner, department, cost centre, stack. Each resource tag key must be unique and only have one value. Must be activated to start appearing on reports.
- Cost + usage reports - most comprehensive cost + usage data, can be integrated with database services. Reports can be uploaded to an S3 bucket.
- Cost explorer - visualisation of where your costs are down to the hour and on which resources, allows for choosing savings plan, uses up to 12 months of historic data to provide forecasts for the following 12 months, right-sizing recommendations for over/under-utilised instances
Monitor against cost plans:
- Billing alarms - simple alarms for when you exceed a certain spend
- Budgets - create budget thresholds for cost, usage, and utilisation of reserved instances. Can also set "under utilisation" thresholds and budget action e.g. terminate instance when budget threshold is crossed.

## Support Plans
Only the root IAM user can change account's level of support plan.
Basic
- Free
- 24/7 access to customer service, docs, whitepapers, support forums
- 7 core checks in Trusted Advisor
- Personal health dashboard

Developer
- Access to email support from support associates
- Allowed one contact to open unlimited numbers of support cases per month
- Get general architectural guidance on how services can be used for different payloads, use-cases, and applications

Business
- Full Trusted Advisor checks
- 24/7 email, phone, and chat support from support engineers
- Architectural guidance based on the context of your use-case.
- Access to infrastructure event management for additional fee.
- Allowed unlimited contacts to open unlimited support cases per month

Enterprise
- For production/business critical workloads
- Technical account managers
- Infrastructure, architecture, operations reviews
- Can get responses in less than 15 minutes

![pic1](/pictures/20221114095733.png)
![pic2](/pictures/20221114095743.png)

## Credits
Credits are applied in the following order:
1. Soonest expiring
2. Least number of applicable products
3. Oldest credit

## Service Quotas
Used to define the maximum for resources, actions, and items in your AWS account.

## Migration Evaluator
Managed service that creates data-driven business cases for cloud migration.
Uses data and expertise on multiple migration strategies to come up with an approach tailored for you.
Also tries to reduce costs by incorporating existing software licences into its calculations.

#aws