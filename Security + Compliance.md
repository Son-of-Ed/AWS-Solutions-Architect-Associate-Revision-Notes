# Shared Responsibility Model
AWS - responsible for security *of* the cloud
Customer - responsible for security *in* the cloud
Shared controls - patching, training

# Encryption
## Encryption in flight (TLS/SSL)
Data is encrypted before being sent and only decrypted upon arriving at the destination.
TLS certs help with encryption (HTTPS).
This prevents man-in-the-middle attacks which can happen when data is in transit.
## Server-side encryption at rest
A data key that is accessible by a server that is storing your data will be used to encrypt the data after it has been received and that same key will be used to decrypt the data when it is requested by a user.
## Client site encryption
Similar to server-side encryption at rest except the data key is stored somewhere on the client side rather than the server side (where the data file is stored).
When a file is sent to the server to be stored it is encrypted before transit by the client-side data key.
When a user wishes to access that file, the file is retrieved and only decrypted by the client-side data key once it has arrived on the client side.
This means the file can never be decrypted while it is on the server side as the server does not have access to the data key.
Can leverage envelope encryption.
## Key Management Service (KMS) 
Allows you to use encryption keys to encrypt your data both at-rest and in-transit.
AWS managed your encryption keys but doesn't necessarily create them.
You never want to store secrets as plain text so you encrypt them with keys.
Integrated with IAM for authorisation and CloudTrail for tracking key usage. as well as EBS, S3, RDS, SSM...
#### Key Types
Symmetric keys (AES-256):
- Uses a single key to encrypt and decrypt data.
- AWS services integrated with KMS use symmetric keys.
- You never get access to the key itself. You call the KMS API to use it.
Asymmetric keys (RSA + ECC key pairs):
- Public (encrypt) and Private (decrypt) pair of keys.
- Used to encrypt/decrypt or sign/verify operations. RSA can be used for either, but not both, whereas ECC can only be used for signing/verifying.
- The public key is downloadable but you never access the private key.

AWS owned keys - free, e.g. SSE-S3, SSE-SQS, SSE-DDB (default keys)
AWS managed keys - free, automatically rotated every year, `aws/<service name>` e.g. `aws/rds`
Customer managed key created in KMS - $1/month, automatic rotation every year (must be enabled).
Customer managed key imported into KMS - $1/month, must be symmetric keys, can only be manually rotated using an alias.
Every API call made to KMS costs $0.03/10000 calls.

KMS keys are region-scoped which can make copying encrypted objects more of a lengthy process.
To copy a snapshot to a different region you have to re-encrypt the snapshot using a KMS key from your destination region. Then when you copy the snapshot across you will be able to decrypt the snapshot using the KMS key you are using in the destination region.
#### Key Policies
Similar to S3 bucket policies except that key policies *must* be enabled in order for anyone to use the key.
If a key policy is not given, then the key is assigned the default policy which assigns the key to the root user which allows all users to use the key. **Why is this?**
By assigning a custom policy to the key, you can restrict the key to be only used by particular users/roles. This is useful for cross-account use of keys as well.

In order to use a snapshot between accounts in the same region, you need to assign a policy to your KMS key which encrypted the snapshot that allows a target account to use that same KMS key so they can decrypt the snapshot.
Once you have done this, share the snapshot with a target account and make a copy of the snapshot in the target account but encrypt the copy with a KMS key from the target account.
#### Multi-Region Keys
You can replicate keys in multiple regions with multiple replica keys that sync to a primary key. 
This allows you to use the same key ID in multiple regions.
It also allows you to encrypt in one region then decrypt in another region without having to add extra steps including cross-region API calls.
These are generally not recommended as it is often better to have keys bound to specific regions.
However, if you are doing something such as global client-side encryption or managing global DynamoDB tables/global Aurora databases then multi-region keys are useful.

## CloudHSM (Hardware Security Module) 
Gives customers the freedom to manage their encryption keys and AWS just provisions the encryption hardware.
CloudHSM keys are generated with CloudHSM hardware, cryptographic operations completed within CloudHSM cluster.

# DDoS Protection
By combining Route53, Shield Advanced, Global Accelerator or WAF+CloudFront, WAF+API Gateway, ELB, and EC2 with auto-scaling groups, you are providing protection from DDoS attacks on your application.
## AWS WAF
Filter Internet requests based on rules, gives protection from Layer 7 HTTP attacks, deployed on ALB, CloudFront, API Gateway, AppSync GraphQL API, or Cognito User Pool. 
Protects from common web exploits.
Defines web Access Control List (ACL) rules to filter based on things such as:
- IP List - filter based on up to 10,000 IP addresses. Can use multiple rules for more IPs.
- HTTP header/body or URI strings - protects from common web attacks such as cross-site scripting (XSS) and SQL injection.
- Size constraints - to block requests over a certain size.
- Geo-match - blocks requests from certain countries.
- Rate-based rules - to counteract DDoS attacks.
The web ACLs defined in WAF are regionally scoped except for CloudFront.
A rule group is a set of reusable rules that can be added to many web ACLs.
## AWS Shield 
#### Standard 
Free service that offers standard protection for all customers against SYN/UDP floods, reflection attacks and other Layer 3 (network layer) or Layer 4 TCP/UDP (transport) attacks.
#### Advanced 
24/7 access to DDoS response team (DRP), $3000 per month per organisation, access to response team, protection against high fees due to DDoS.
Intended for services prone to DDoS attacks.
Provides expanded DDoS protection for the following resources: EC2, ELB, CloudFront, Route 53, Global Accelerator.
Automatically creates, evaluates, and deploys WAF rules to mitigate layer 7 attacks.
## Firewall Manager
Manages firewall rules in all accounts of an AWS organisation.
This allows your to apply a common set of rules across the resources in your accounts:
- WAF rules - ALB, API Gateway, CloudFront
- AWS Shield Advanced - ALB, CLB, NLB, Elastic IP, CloudFront
- Security groups - EC2, ALB, ENI
- AWS Network Firewall - VPC
- Route53 Resolver DNS Firewall
All policies are created at the region level and are applied to new resources as they are created across all present and future accounts in your AWS organisation.

## CloudFront + Route53 
High availability across a global network of edge locations
Use auto-scaling to be even more protected
Typical architecture goes something like:
Web -> Route53 -> CloudFront + WAF -> Shield inside SG of public subnet

## Penetration Testing
AWS can carry out security assessments and penetration testing on their infrastructure for 8 services.
You are allowed to do some penetration testing on your infrastructure. However, anything that looks like an attack will be picked up by AWS so there is a list of penetration testing activities that are prohibited on AWS.

## Multi-Factor Authentication
There are 3 main kinds of MFA device:
- Virtual device - what I'm used to e.g. Google Authenticator
- Hardware device - separate piece of hardware specifically for generating a 6 digit code
- U2F device - USB device that plugs into your machine which you tap when logging in

## Certificate Manager (ACM)
Provision, manage, and deploy SSL/TLS certificates for in-transit encryption i.e. HTTPS instead of HTTP.
Supports public TLS certs free of charge and private TLS certs for a fee.
Automatic cert renewal can be enabled as well.
Integrates with ELBs, CloudFront distributions, APIs on API Gateway.
Doesn't integrate with EC2.
#### Requesting Public Certs
1. List domains to be included in cert: FQDN such as `corp.example.com` or wildcard domains such as `*.example.com`. Can have as many as you want.
2. Select validation method: 
	- DNS validation (preferred for automation purposes) where you leverage a CNAME record to verify that you own the domain.
	- Email validation which sends emails to contact addresses in a domain registrar who verify that you requested that cert.
3. Wait a few hours to get verified.
4. The public cert will be enrolled for automatic renewal. ACM automatically renews certs generated with ACM 60 days before expiry.
#### Importing Public Certs
ACM can host certs created externally but doesn't support automatic cert renewal for these.
ACM sends daily expiration events to EventBridge starting 45 days before your cert is due to expire which can then trigger Lambdas or send messages to SQS/SNS.
You can also configure the `acm-certificate-expiration-check` in Config to send events to EventBridge for any certs that are due to expire after X days.
#### Integration with ALB
Can redirect users from HTTP to HTTPS with ALB. The HTTPS request then uses a TLS cert from ACM to connect to your service behind the ALB.
#### Integration with API Gateway
ACM can be used with Edge-optimised and Regional API Gateway endpoints.
You need a custom domain name in your API Gateway for either endpoint type.
Edge-optimised endpoints:
- Requests are routed through CloudFront Edge locations for improved latency.
- API Gateway only exists in one region.
- The TLS cert must be created in us-east-1 which is the region where CloudFront is hosted (regardless of the Edge locations).
Regional:
- For clients that exist within the same region.
- The TLS cert must be stored in ACM in the same region as the API Gateway.
For both, you need to set up an A-Alias record (if you are using the cert to secure a connection to an AWS service) or a CNAME record (for anything else) in Route 53.

## SSM Parameter Store
Secure storage of configurations and secrets using KMS.
Provides version tracking of configs/secrets.
Integrates with IAM to restrict access to certain configs/secrets.
Can use notifications for EventBridge.
Integrates with CloudFormation for infrastructure deployment.
Can store parameters with a hierarchy similar to JSON format.
Able to access secrets being stored in Secrets Manager as well as publicly available parameters.
#### Parameter Tiers
Standard:
- Total number of parameters allowed per account + region: 10,000
- Max size of parameter values: 4kb
- Parameter policies: No
- Cost: Free
- Storage cost: Free
Advanced:
- Total number of parameters allowed per account + region: 100,000
- Max size of parameter values: 8kb
- Parameter policies: No
- Cost: Charges apply
- Storage cost: $0.05/advanced parameter/month
Can create policies for advanced parameters which can be integrated with EventBridge: expiration policy, expiration notification policy, no change notification policy.
## Secrets Manager
Mostly meant for integration with RDS and is used to store secrets.
Can enforce rotation of secrets every X days with services like Lambda.
For services integrated with Secrets Manager, e.g. RDS, Redshift, DocumentDB, when the secret is rotated in Secrets Manager, it is also rotated in the database you have linked to that secret.
Secrets are encrypted using KMS.
Secrets can be replicated across multiple regions from a primary secret.

## Artifact
Portal that gives you access to AWS compliance and agreement documents.

## GuardDuty
Intelligent threat discovery using machine learning to help detect 3rd party threats by analysing events in AWS services and developing findings. These findings are then used to send alerts if GuardDuty detects any suspicious activity in future.
Input data includes:
- CloudTrail - management events, S3 data events
- VPC flow logs - unusual traffic/IP addresses
- DNS logs - compromised EC2 instances sending encoded data within DNS queries
- EKS Audit logs, RDS/Aurora, EBS, Lambda
Can set up EventBridge rules to send notifications to Lambda/SNS when there is an alert since GuardDuty alerts are classed as events in AWS services.
Can protect against crypto-currency attacks.
Offers a 30 day trial.

## Inspector
Automated security assessments that takes data from SSM and generates findings which are sent to EventBridge and Security Hub.
Only used on EC2s with the SSM agent installed on them, container images pushed to ECR, and Lambda functions when they get deployed.
Looks for package/OS vulnerabilities using the CVE database and issues with network reachability i.e. unintended (un)availability.

## Config
Used for auditing and recording compliance by listing configurations and changes made over time.
Can store results in S3 which can be analysed by Athena.
Receive alerts for changes to any of your configs or when rules are broken.
Can be aggregated across accounts/regions.
Comes with over 75 rules for your configuration but you can also create custom ones that are defined in Lambda. The rules do not prevent actions from happening!
Pricing - no free tier, $0.003 per config item per region, $0.001 per config rule evaluation per region.
Use the Config dashboard to view the compliance, config changes, and CloudTrail API calls for a resource over time.
You can invoke mitigations using Config e.g. triggering an SSM document that revokes non-compliant access keys or Lambda functions that can do whatever you specify.

## Macie
Data security/privacy tool that uses machine learning to spot patterns and identify any sensitive data, such as personally identifiable information (PII).
Notifies EventBridge which can then integrate with other services.

## Security Hub
Central security tool for multiple AWS accounts to help automate security checks across multiple security services e.g. Inspector, GuardDuty.
AWS config service must be enabled first.
Costs per check.
Can output findings to Detective to investigate.

## Detective
Produces graphs and other visualisations to help identify the root causes of security incidents.

## Abuse
Team that can be contacted with any reports of AWS services being misused for abusive/illegal purposes.

#aws