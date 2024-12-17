# Infrastructure At Scale
## CloudFormation
The proprietary IaC tool on AWS that creates templates in JSON/yaml to provision infrastructure in AWS. You can also get a handy networking graph to see how your infrastructure us connected up.
Only pay for the resources you create.
Resources created through a CloudFormation stack are tagged with specific identifiers so that you can identify which stacks are costing you more money.
Can automate the creation/deletion of resources in a stack at certain times to provide cost savings.
Can leverage existing documentation/templates available online.
#### Stack Designer
Can visualise what goes into your stacks so that you can view it like an architecture diagram.

## Cloud Development Kit
Can define infrastructure in many familiar programming languages which is then compiled into a JSON/yaml template to be built using CloudFormation.
Can deploy infrastructure and application runtime together because of this (not sure why???)

## Elastic Beanstalk
Most application are structured in 3 tiers: load balancer -> compute resources -> databases + cache (backend stuff).
Beanstalk is a managed PaaS that uses a dev-centric view to deploy resources like ASGs, ALBs, EC2s, RDS, cache etc.
Devs can configure the template structure but their main responsibility will be to maintain their code, not the infrastructure.
Code changes can be made within the environment and saved to a new app version which can then be deployed using Beanstalk again.
Beanstalk has two deployment modes: single instance or HA i.e. multi-AZ with read replica DBs.
Supports 13 different coding platforms (languages + Docker) but you can create a custom platform if you require one that is not supported.
Can be used to monitor the health of your resources as well.
Only pay for the resources you create.
Beanstalk apps can be described using the following parts:
- Application - the whole package made up of components including the environments, versions, configurations etc.
- Application Version - an iteration of your application code
- Environments - collection of AWS resources for running the code (1 app version at a time). Comes in two tiers: Web Server Environment Tier & Worker Environment Tier
	- Web Server Environment - typical architecture e.g. DNS, ELB, EC2 used for running an we app/API
	- Worker Environment - used for long-running applications on demand/on a schedule
	Can use a worker environment that works asynchronously with an SQS queue that sends messages to worker instances. This can be coupled with a Web Server Environment as well.

## CodeDeploy
Automatically update applications with EC2s or on-prem servers.
Any servers/instances must be deployed using the CodeDeploy agent in order to use CodeDeply.

## CodeCommit
Version control in AWS.

## CodeBuild
Serverless code compiler that can run tests and produce packages (for CodeDeploy to deploy).
Only pay for the builds you make.

## CodePipeline
Orchestrates the steps in the software development lifecycle e.g. commit, build, test, deploy.

## CodeArtifact
Used for artefact storage so that CodeBuild can retrieve any dependencies it may have.

## CodeStar
Manage all of the above CodeX services from a unified UI for software development projects.

## Cloud9
IDE for editing, running, debugging code running in the cloud.

## SystemsManager (SSM)
Patch/run commands across all instances/servers in your estate whether they are in AWS or on-prem.
Need to install the agent on all managed servers.
Track and resolve operational issues in groups of AWS services/instances from this UI.
Useful for providing SSH access without having to open new inbound ports on instances or using public IPs.
Better for compliance and auditing since it is providing a unified window for accessing instances through AWS.
#### Session Manager
Allows you to start a secure shell into your EC2s and on-prem servers.
No SSH access or bastion hosts required so you don't need to open port 22 in your security groups or manage SSH keys.
Access is managed via IAM.
Supports Mac, Linux, or Microsoft.
Session logs can be forwarded to S3 or CloudWatch Logs.
#### Run Command
Execute scripts/commands across multiple EC2s/on-prem servers with the SSM agent running on them.
Any outputs from the commands can be sent to S3/CloudWatch Logs.
SNS notifications can be sent about the command status e.g. success/fail.
Integrates with IAM + CloudTrail.
Can be invoked by EventBridge.
#### Patch Manager
Automates the process of applying updates to your instances/on-prem servers.
Covers things like OS/application/security updates.
Can schedule maintenance window duration and schedules for patching/commands to be run.
Generates patch compliance reports for auditing/identifying unpatched instances.
#### Automation
Simplifies maintenance/deployment tasks or EC2 and other AWS resources e.g. restart instances, take snapshots, create AMI etc.
Uses automation runbooks which are SSM documents which define the actions taken.
Can be triggered by EventBridge, CLI, SDK, maintenance windows, or by Config for rule remediations.

## OpsWorks
Apply EC2 configuration using Chef & Puppet but only in the case of using those two tools in advance of moving to the cloud.

#aws #iac #pipelines 