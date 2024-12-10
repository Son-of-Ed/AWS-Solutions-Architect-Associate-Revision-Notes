## Workspaces
Desktop-as-a-service used to provision Windows/Linux virtual desktops.
Eliminates management of on-premises virtual desktop infrastructure.
Pay-as-you-go with monthly/hourly rates.
Minimise latency by deploying virtual desktops as close to users as possible.

## AppStream 2.0
Stream any application without acquiring/provisioning infrastructure.
Works with any device that has a web browser.
Configure instance type for each application e.g. PhotoShop needs a lot of RAM

## Sumerian
Used to create VR, AR, and 3D applications by creating 3D models with animations.

## IoT Core
Used to connect IoT devices into AWS via a scalable serverless service.

## Elastic Transcoder
Used to convert media files in S3 buckets into various other file formats for different types of devices.
Pay-as-you-go tariff and very scalable for large files/large numbers of files.

## AppFlow
Managed integration service for SaaS applications (e.g. Salesforce, SAP, Slack, ServiceNow) and AWS.
Can be scheduled to run at set times, in response to events, or upon a manual request.
Capable of performing data transformations like filtering and validation.
Data is encrypted over the public internet or can be streamed privately using PrivateLink.
Takes away management of creating APIs to integrate your services.

## AppSync
Backend for storing and syncing data across mobile and web apps in real-time.
Built on GraphQL -> integrated with DynamoDB + Lambda.

## Amplify
Set of tools for developing and deploying scalable full-stack web and mobile applications.
Makes use of various AWS services to provide the entire technology stack.
Takes care of authentication, storage APIs, CI/CD, analytics, AI/ML, monitoring.
Like Elastic Beanstalk for web and mobile applications.
Connect to your source code repository e.g. GitHub, GitLab, CodeCommit, Bitbucket.

## Device Farm
Test web/mobile apps concurrently on real devices.
Receive logs, bug reports etc. from devices in the farm.

## Fault Injection Simulator
Run chaos engineering experiments to identify bugs and bottlenecks.
Monitor applications through CloudWatch and EventBridge.

## Ground Station
Service for controlling satellite communications for data processing. 
Can download data to AWS VPC in seconds from satellites.

## Simple Email Service (SES)
Managed service for sending/receiving emails securely to a global audience.
Provides insights about email reputation, performance, anti-spam feedback.
Supports DomainKeys Identified Mail (DKIM) and Sender Policy Framework (SPF).
Can deploy with flexible IP using shared, dedicated, or customer-owned IP addresses.
Send emails from your application using the console, APIs or SMTP.
Useful for bulk email communications such as marketing.

## Pinpoint
Scalable two-way marketing communications via email, SMS, push notifications, voicemails, or in-app messages.
Any message stream events such as TEXT_SUCCESS or TEXT_DELIVERED can be streamed to SNS, Kinesis Data Firehose, and CloudWatch Logs.
More scalable than SNS and SES because it can tailor messages to specific segments of your audience and use message templates as well as delivery schedules. This takes away some of the management of your communications as SES requires you to manage all of the content and scheduling within your application logic.

#aws #appstream