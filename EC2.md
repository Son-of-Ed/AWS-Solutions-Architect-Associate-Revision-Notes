## Elastic Cloud Compute - EC2
Compute resources which can be built with an OS of your choice and can have storage attached to them.
Pay for the time used (pay for the first 60s and any use after that).
Configuration options:
- Operating system - Linux, windows, Mac OS
- Compute Power & Cores (CPU)
- Random-access memory (RAM)
- Storage:
	- Network-attached (EBS & EFS)
	- Hardware-attached (EC2 Instance Store)
- Network card - speed of the card and public IP address
- Firewall rules - security groups
- Bootstrap script - user data script ran at launch
- IAM role - best practice to give your EC2 instance an IAM role with specific permissions rather than giving it access to someone's personal credentials.

Instance types take the following form: m5.2xlarge
	m is the instance class
	5 is the generation
	2xlarge is the size within that instance class

Instance classes include:
- c - compute optimised for high performance processors
- i - IOPS optimised for high throughput requirements
- m - general purpose
- r - memory optimised for processing large datasets
- p/g - accelerated computing optimised for GPU processing

## Security Groups
Inbound rules can be used to control which IP addresses can connect to the instance the SG is attached to.
Outbound rules configure what your instance can connect to e.g. is your EC2 able to access the Internet.
Security groups are created for specific VPCs so cannot be shared between them.
If your attempted connection hangs and hangs until it times out, you are likely seeing an issue with your SG rules.
"Connection refused" tends to appear quicker than timeout errors and is probably an application issue not an SG issue.
#### Ports to know
SSH (Secure Shell), 22 - accessing a Linux instance
	SSH can be used on Mac, Linux, Windows >= 10
	Putty can only be used on Windows
	EC2 Instance Connect can be used on any EC2 instance
FTP (File Transfer Protocol), 21 - upload files to a file share
SFTP (Secure File Transfer Protocol), 22 - upload files using SSH
HTTP, 80 - for accessing unsecured websites
HTTPS, 443 - for accessing secured websites
RDP (Remote Desktop Protocol), 3389 - logging on to Windows instance
SQL, 1433
MySQL, 3306

## Pricing
On-demand - best for short and unpredictable workloads as these are quite expensive.
Reserved instances - scoped to particular regions/AZs. Useful for steady-state workloads e.g. databases.
Spot instances - useful for short-lived workloads as instances can be outbid from you.
	Persistent spot requests work like auto-scaling groups. They will attempt to fulfil your desired number of instances when they get terminated. To actually remove spot instances from your fleet, you will need to cancel the persistent spot request. One-time spot requests do not do this.
	Spot fleets can be configured to use particular strategies e.g. cost-optimised, capacity-optimised, diversified, price-capacity-optimised.
Dedicated hosts - physical servers that run instances that are solely for your use. Useful for use-cases with strong compliance requirements or complicated software licence requirements.
Dedicated instances - instances that are only usable by you but some hardware may be shared with other subscriptions. *Note: will likely be removed in place of the EC2 Savings Plan which allows users to dedicate a given expenditure ($/hour) to a specification of instances over a 1/3 year plan.*

## IP Addresses
#### Public IP
Public IP addresses allow your machine to be accessible over the public Internet.
Must be unique to avoid clashes with other public IPs.
Can allow for easy geo-location.
You can SSH onto an instance by its public IP.
#### Private IP
Private IP addresses are used to connect to machines within a private network.
Only need to be unique within a private network i.e. two companies with their own private networks can use the same private IPs.
Machines with private IPs connect to the Internet via a proxy with a public IP.
Only a specific range of private IPs can be used within a private network.
Cannot SSH onto an instance by its private IP unless you are using a VPN that connects you to that instances private network.
#### Elastic IP
Can be used to retain a specific IP address for your instances if they get destroyed + recreated to hide any outages. 
Only have 5 Elastic IPs in your account at a given time (but can ask AWS for more).
Not a common pattern as you are better off redirecting your services traffic to be distributed across all your instances with DNS names/load balancers, rather than just holding on to a specific IP.

## Placement Groups
Allows you to direct the placement of your instances based on a particular strategy:
- Cluster - all instances placed within the same AZ for low-latency. Very risky... but useful for applications with big data jobs that need to complete quickly or have high network throughput.
- Spread - instances spread across different physical hardware between AZs. Provides high-availability and isolates instance failure, but is limited to 7 instances per AZ in your placement group.
- Partition - separates instances by hardware rack in an AZ. Can have up to 7 partitions/racks per AZ in a placement group and can scale to include 100s of instances. Partition/rack failures affect many instances but each partition is isolated from the other partitions.

## Elastic Network Interfaces (ENIs)
Represent virtual network cards that you can attach to EC2 instances.
They are bound to particular subnets i.e. particular AZs.
Can be configured to have the following:
- One public IPv4
- A primary private IPv4 as well as secondary private IPv4s.
- An Elastic IP per private IPv4
- Security groups
- MAC address
EC2s can have multiple ENIs.

## Hibernate
Used to "pause" instances and preserve the RAM by dumping its contents to the root EBS volume attached to the instance as the instance is shut down.
This can help shorten startup times and is useful for keeping the progress of long processes.
The root EBS volume should be encrypted and have enough room to store the RAM dump.

## Connecting to EC2
You can connect to EC2 instances from the AWS console using EC2 Instance Connect which allows you to connect in your browser using SSH without having to manage SSH keys.

## Amazon Machine Image (AMI)
Can save the state of EC2s so that you can quickly launch new EC2s using the same configuration as other EC2s. This speeds up boot time since the instance is built with the user data already populated.
Must launch EC2s in the same region as where the AMI is hosted. This does not affect EC2 performance though.
The process of creating an AMI is to start an EC2 instance, stop the instance (for data integrity), the data is then saved to a snapshot which future instances can be restored from.
You can create a "golden AMI" to speed up application creation i.e. you start an EC2, install all your required applications/dependencies, create an AMI from this instance. Then your applications can get started much quicker with a lighter user data script.
#### AMI Sharing
1. To share an AMI between accounts of the same region, you need to add a Launch Permission to the AMI in your source account which specifies the target account you wish to share the AMI to.
2. Then share the KMS key used to encrypt the AMI with the target account and add the following permissions to an IAM role in the target account so it can use the AMI: `kms:DescribeKey, kms:ReEncrypted, kms:CreateGrant, kms:Decrypt`.
3. You may also want to re-encrypt the AMI in the target account using a different KMS key from the target account.
#### VM Import/Export
You can download AMIs from AWS in the `.iso` format and use them to launch your own VMs with software such as VMWare, KVM, VirtualBox (Oracle VM), or Microsoft Hyper-V.
This also works going the other way if you wanted to migrate important VMs to AWS e.g. cloud migration, disaster recovery

## Image Builder
Used to create, validate, and automate the production of EC2 AMIs.

## VMWare Cloud
Can extend your on-prem data centre infrastructure to run on AWS if you are already managing your VMs using VMWare Cloud on-prem.
Use cases include cloud migration, managing hybrid workloads between the cloud and your on-prem data centre, and disaster recovery.


#aws 