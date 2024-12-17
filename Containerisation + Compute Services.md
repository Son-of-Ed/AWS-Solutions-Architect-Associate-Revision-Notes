# Containerisation + Compute Services
## ECS 
ECS tasks have specific IAM roles attached to them since each task will be used to access various AS services. The role used is defined in the ECS task definition.
Can be integrated with ALBs for most use cases or NLBs for high throughput/performance use cases or if you want to pair ECS with AWS Private Link.
Use EFS for both launch types as ECS cluster tasks running in any AZ will be able to be able to share the same data. Data is persistent upon container failure.
S3 cannot be mounted to ECS tasks.
Service - long running group of tasks that can be stopped and restarted e.g. web app.
Task - standalone task that executes and ends upon completion e.g. batch jobs.
Can autoscale tasks, i.e. containers, based on target tracking (average CPU usage, RAM utilisation, or LB requests per target), step scaling based on a CloudWatch alarm, or scheduled scaling for predictable demand.
#### EC2 Launch Type
Used to launch Docker containers on EC2s and can use base images from Docker Hub or ECR (Elastic Container Registry) which can be used to host private images.
Used in conjunction with EC2s and can be integrated with App Load Balancers
Configure an IAM role for the EC2 instance profile which is used by the ECS agent to do things like make API calls to ECS, send container logs to CloudWatch, pull Docker images from ECR, reference sensitive data in Secrets Manager or SSM Parameter Store.
Autoscale tasks using ECS autoscaling or by using the cluster capacity provider which will create tasks in response to not meeting specific demands e.g. CPU, RAM. Can also be paired with traditional EC2 ASGs. 
#### Fargate Launch Type
Container management on AWS without EC2 management -> serverless
#### Common Architectures
- ECS tasks invoked by EventBridge e.g. object is uploaded to bucket and triggers an ECS task that process the object and uploads the data to a database.
- EventBridge schedule runs regular tasks e.g. run batch operations on a bucket every hour.
- SQS queue e.g. can enable tasks to poll for messages from a queue and use ECS service autoscaling to create more tasks if the queue size increases.
- Notification of service disruption e.g. services stopping can trigger EventBridge to push a message to an SNS topic which will then notify an administrator.

## ECR
Can create public/private image repositories.
Backed up by S3.
Supports vulnerability scanning, versioning, image tags, image lifecycle policies.

## EKS
Managed Kubernetes cluster for scaling and managing containerised applications.
Supports EC2 launch type for deploying worker nodes and Fargate launch type for deploying serverless containers.
Deployed into a VPC in 3 AZs with a private and public subnet in each. The private subnets are what host the EKS nodes i.e. EC2 instances (which host the pods (which host the containers)).
The nodes in each subnet are managed by an ASG when deployed as managed nodes. Self-managed nodes won't necessarily have this unless you configure it.
ELBs can be deployed in the public or private subnets to expose services running in your EKS cluster.
Can use EKS optimised AMIs.
Need to specify a storage class for your cluster that will leverage a particular Container Storage Interface (CSI) driver. The storage classes you can choose from are EBS, EFS (required for Fargate launches), FSx for Lustre, or FSx  for NetApp ONTAP.

## AppRunner
Managed service for deploying web apps/APIs at scale without needing to know about infrastructure.
Builds from your source code/image and deploys with scalability, HA, encryption, and load balancing.
Supports VPC access.
Can connect to databases, message queues, and cache services.
Makes use of ECS under the hood.
Acts more like automated CI/CD tooling whereas Beanstalk acts more like a PaaS that does allow you to manage the infrastructure.
Used more for cloud-native apps that can support serverless infrastructure.

## Lambda
Serverless service used to execute virtual functions which can be written in many programming languages such as Create, Read, Update, Delete (CRUD) operations on databases.
Lifecycle phases of a running Lambda function consist of Init, Invoke, and Shutdown.
Only pay for number of executions or number of GB-seconds (size of RAM x duration) and don't have to manage EC2 instances.
Monitor through CloudWatch.
Can be triggered by events or scheduled (good for CRON jobs).
Autoscaling is baked in to scale with demand.
Free tier includes up to 1,000,000 invocations per month ($0.20 for each 1,000,000 requests after that) and 400,000GBs, e.g. 400,000s if using 1GB of RAM, of compute time per month (calculated in 1ms increments, costs $1 for 600,000GBs after outside of the free tier).
Supports:
- Node.js (JavaScript)
- Python
- Java (Java 8 compatible)
- C# (.NET Core)
- GoLand
- C#/Powershell
- Ruby
- Custom runtime API (community supported e.g. Rust)
- Lambda container image (must implement Lambda runtime API. ECS/Fargate is probs more appropriate for Docker images with arbitrary runtimes)
Lambda has a few main integrations e.g:
- API Gateway
- Kinesis
- DynamoDB
- S3
- CloudFront
- CloudWatch Events EventBridge
- CloudWatch Logs
- SNS
- SQS
- Cognito
#### Limitations (per region)
RAM allocation: 128MB - 10GB (1MB increments)
Execution time: < 15min
Environments vars: < 4kb
Disk capacity in the "function container" (in /tmp): 512MB - 10GB
Concurrent executions: Up to 1000 (can request more)

Function deployment size (as compressed zip file): < 50MB
Function deployment size (decompressed code + dependencies): < 250 MB
Other files can be loaded into /tmp on function startup 
#### SnapStart (Java 11+)
Free feature that offers up to 10x speedup for Java functions by effectively removing the "init" phase from the execution since the function is in a pre-initialised state.
Upon a new version being published, Lambda will initialise your function, take a snapshot of the memory and disk state of the initialised function, and cache the snapshot for low-latency access.
#### Edge Functions
Code that you attach to CloudFront distributions to run at locations closer to your users to reduce latency. You can also use Edge Functions to customise the content hosted within your CDN network.
Use cases include:
- Website security + privacy
- dynamics web apps
- Search engine optimisation
- Intelligent routing across origins/data centres
- Bot mitigation at the Edge
- Real-time image transformation
- A/B testing
- User authentication + authorisation
- User prioritisation
- User tracking + analytics
##### Lambda@Edge
Lambda functions written in NodeJS or Python.
Used to modify the user request, origin request, origin response, and user response.
Functions are authored in a single region and CloudFront replicates it to other locations.
Scales up to thousands of requests per second.
Max execution time of 5-10s.
Max memory available of 128MB - 10GB
Can access request body, network, and file system.
No free tier.
The longer execution time of these functions makes them useful for:
- Pulling in 3rd party libraries
- Using network access to make use of external services
- Using file system and access to HTTP request body.
##### CloudFront Functions
Managed through CloudFront, not Lambda.
Lightweight JavaScript functions used to modify the user request in the CloudFront distribution before sending it to the origin as well as modifying the user response after the origin has sent a response back to the CloudFront distribution before forwarding it on to the user.
Used for high-scale (millions of requests per second), ultra-low latency (sub-millisecond startup times) CDN customisations e.g:
- Transforming request attributes (headers, cookie, query strings, URL) to create an optimal cache key.
- HTTP header modification.
- URL rewrites/redirects.
- Request authentication & authorisation e.g. validating JWTs to allow/deny requests
1/6th the price of Lambda@Edge functions as well as free tier.
Max execution time of < 1ms.
Max memory available of 10kb.
Cannot access request body, network, or file system.
#### Lambda in VPC
By default Lambdas are launched in an AWS-owned VPC which means they cannot access resources that exist within your VPC.
Can configure Lambdas to access resources within a specific VPC by specifying the VPC ID, subnets to run in, and a security group for the Lambda. This creates an ENI within your subnets to allow Lambda functions to access the resources it needs.
##### Lambda with RDS Proxy
It's often good practice to pair Lambdas that require access to RDS/Aurora with RDS Proxy as the scalability of Lambda functions means that there is the potential to create dozens of open connections to your database which can put unnecessary load on your db. This helps to keep your Lambdas scalable, improve availability by reducing failover time by 66% and preserving connections. It also enforces IAM authentication. However, the Lambdas *must* be deployed inside your VPC because RDS Proxy is never accessible by resources in public VPCs.

## API Gateway
Used by developers to create and maintain APIs.
Can be coupled with Lambda to create entirely serverless APIs.
Supports WebSocket Protocol.
Handles API versioning, APIs in different "stages" e.g. dev, test, prod, authentication + authorisation.
Creates API keys for throttling.
Can transform + validate requests/responses.
Can generate SDK + API specifications.
Capable of caching API responses.
Integrates with products such as:
- Lambda
- HTTP endpoints e.g. on-prem HTTP endpoints, ALBs
- Any API for AWS services
#### Endpoint Types
Edge-Optimised:
- Requests routed through CloudFront edge locations for reduced latency for global clients.
- API Gateway still lives in a single region.
Region:
- For clients that exist within the same region.
- Can manually combine with CloudFront if you want more control over the caching strategies and CloudFront distribution.
Private:
- Only accessible from within VPC using an interface VPC endpoint (ENI)
- Use a resource policy to define access.
#### Security
User authentication using IAM, Cognito (for external users), or a custom authoriser (which uses a Lambda with your own logic).
Can combine with Certificate Manager to enable custom domain name HTTPS. Edge-optimised endpoints must have the cert hosted in us-east-1. Regional endpoints must have them in the same region as the endpoint. Need to set up a CNAME or A-alias record in Route53 to enable this.

## Step Functions
Used to orchestrate multiple Lambda functions as well as other services.
Very useful for any workflow that needs to be visualised by some sort of graph/flowchart.
Can start processes in series/parallel with timeouts and error handling.
Integrates with EC2, ECS, on-prem servers, API Gateway. SQS...
Can include human approval for each step.

## Batch
Used to run thousands of "batch" jobs (jobs that have a start and an end point).
Dynamically launches EC2/spot instances.
Batch jobs defined as Docker images and run on ECS.
Can use any runtime unlike Lambda as long as it's packaged as a Docker image.
More useful than Lambda in the case where you need to run jobs for longer than 15 minutes.
Relies on EBS/disk space for storage whereas Lambda only has access to temporary disk space.

## Lightsail
Multi-purpose service that combines some common AWS services into a dev/test environment.
Limited integrations with other services.
Platform-as-a-Service.
Doesn't support per-second billing
EASY to set up

#aws #containers #serverless