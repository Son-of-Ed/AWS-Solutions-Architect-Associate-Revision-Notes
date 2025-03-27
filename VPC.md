# VPC
Virtual private clouds are private networks that span all of the AZs within a region so can be used to deploy resources regionally.
An AZ is a collection of one or more data centres in the same location.
You can have a soft maximum of 5 VPCs per region.
VPCs are private by nature therefore can only use IPv4 ranges within the private ranges established by IANA (see [[#CIDR (Classless Inter-Domain Routing)]]).
Maximum number of CIDR ranges is 5:
- Minimum size of each CIDR range is /28 i.e. 16 addresses
- Maximum size of CIDR range is /16 i.e. 65536 addresses
You VPC CIDR should not overlap with your other networks.
Can set whether EC2s inside the VPC use dedicated/shared hardware by default.
VPCs consist of the following:
- CIDR ranges - One or more IPv4/IPv6 ranges specifying the range of private IP addresses within your VPC
- Subnets
- Route tables
- Network ACL
## Networking Costs
Traffic going into EC2s - free
traffic between EC2s in the same AZ - free
Between EC2s in different AZs:
- Via public/Elastic IP - $0.02/GB
- Via private IP - $0.01/GB
Between EC2s in different regions - $0.02/GB
Takeaways:
- Use private IP for traffic between EC2s in the same region
- Use the same AZ for your EC2s for greater savings.

Ingress traffic from outside networks to AWS services is typically free.
Egress traffic from AWS to other networks will have a cost associated with it.
This means that you want to move large queries/requests to services within AWS then you can export the result of the request to the end user.


## CIDR (Classless Inter-Domain Routing)
Method for allocating IP addresses.
Used in security groups + AWS networking in general.
CIDR ranges consist of:
- Base IP - an IP contained in the range
- Subnet mask - how many bits can change in the IP
	- /X - there are $2^{32-\text{X}}$ IPs in the range for IPv4 (there would be $2^{128-\text{X}}$ for IPv6).
IP addresses can vary from 0.0.0.0 to 255.255.255.255.
192.168.0.254/30 -> you can have from 192.168.0.254 up to 192.168.1.1
The maximum CIDR range in AWS is /16

The Internet Assigned Numbers Authority (IANA) established specific blocks of IPv4 addresses for private (LAN) addresses and public (internet) addresses:
- Private IPs can only exist within:
	- 10.0.0.0/8 in big networks
	- 172.16.0.0/12 which is the default VPC in AWS
	- 192.168.0.0/16 for home networks
- All other IPs on the internet are public IPs.

## Default VPC
All AWS accounts have a default VPC which EC2 instances are launched into if not other subnet is specified.
The default VPC can connect to the internet and all EC2s inside it have public IPv4 addresses.
It also comes with public and private IPv4 DNS names.

## Subnet
Logically isolated sub-range of IP addresses within your VPC connected to a single AZ. 
	Public subnet - can connect to the Internet
	Private subnet - can only connect to things inside your VPC
AWS reserves 5 IPs (first 4 and last 1) within your VPCs CIDR range(s) that cannot then be assigned to EC2s:
- First IP - network address
- Second IP - reserved for VPC router
- Third IP - reserved for mapping to AWS-provided DNS
- Fourth IP - initially unused but still reserved for future use
- Last IP - network broadcast address
If you require X IP addresses for the EC2s in your VPC, your subnet must have a size of Y such that $2^{32-\text{Y}}\geq X+5$ e.g. if you require 50 IPs for your EC2s ($2^{32-\text{Y}} \geq 55$), then you must choose a CIDR range size for your CIDR of at least 26 ($2^{32-27}=2^5=32,\quad 2^{32-26}=2^6=64$).
You probably want to choose a smaller CIDR range for your public subnets (which will probably only house your load balancers and public facing services). This means you have more IP addresses available for your internal services that are not public facing.

## Route Table
Used to define which resources can access the Internet and connectivity between subnets.
If set to the main route table then it will be used by any subnet that doesn't explicitly have a route table associated with it.
You define a destination IP range that an EC2 in your subnet will attempt to connect to and a target that traffic to/from the destination will be routed through e.g. internet gateway.

## Internet Gateway 
Provides access for instances in your public subnet to access the Internet when a route table is also attached to your public subnet.
Has a one-to-one relationship with VPCs so cannot attach an internet gateway to multiple VPCs.

## Bastion Host
An EC2 instance called "bastion host" that resides in your public subnet with it's own SG that allows users to access EC2 instances that reside in your private subnet via SSH.
Users first SSH onto the bastion host in the public subnet then SSH onto the EC2s in your private subnet.
The bastion host SG must allows inbound traffic on port 22 from a restricted CIDR i.e the public CIDR range of your corporation (don't allow inbound traffic from any IP as this poses a security risk for the instances in your private subnet).
The SG of the EC2s in your private subnet must allow inbound traffic from the SG of your bastion host or from the private IP of your bastion host.

## NAT (Network Address Translation) Instances - *outdated*
Allow your private instances to access the Internet while keeping them private.
They are EC2 instances with their own SG that are instead managed by the AWS user.
The SG allows inbound HTTP(S) traffic from your private subnets and SSH from your home network. Also need to have outbound traffic going to the public internet.
Need to have source/destination checking disabled upon launch. This allows the NAT instance to send/receive traffic when the NAT instance is not the destination of the request passing through it.
Must have fixed Elastic IP attached to it.
Route tables in the private subnet must be configured to allow traffic destined for the public internet to target the ENI attached to the NAT instance in the public subnet.
Requests made to an external server will see the source IP as the public IP of the NAT instance rather than the IP of the instance that made the request.
**NOT** highly available/resilient out of the box. Would require an ASG with resilient user data script.
The pre-configured Amazon Linux AMI for NAT instances has reached its end of standard support.
Small instances sizes have lower internet bandwidth.
Pay per hour the EC2 is running, instance type + size, network throughput as well.

## NAT Gateway
Similar to NAT instances but they are managed by AWS instead of you.
Higher availability (within an AZ), resilience, and bandwidth than NAT instances.
Pay per hour of usage and bandwidth used.
Created in a specific AZ (like subnets) and uses an Elastic IP.
Still requires an internet gateway to be created.
Has 5GB/s of bandwidth but can automatically scale up to 100GB/s.
Update the route table of your private subnet to route traffic destined for the public internet to your NAT gateway.
To make your NAT gateways highly available across AZs, you will need a NAT gateway in each public subnet you have. No connectivity required between NAT gateways since if one AZ goes down then so does the NAT gateway.
Cannot be used as a bastion host unlike NAT instances.

## Network ACL (NACL)
- Firewall that ALLOWS/DENIES traffic in and out of a *subnet*. 
- Rules only include IP addresses
- Return traffic from ALLOWED instances must also be ALLOWED back (stateless).
- Rules are evaluated in order of priority based on the number of that rule. Therefore, conflicting rules are then evaluated based on their priority.
- The default NACL for a subnet without a user-defined NACL allows all traffic in and out of a subnet.
- Applies to all instances that exist within the assigned subnet.
#### Ephemeral Ports
When two endpoints establish a connection they connect using ports.
Clients will initiate a connection with a defined port. They will receive a returned connection on one of their ephemeral ports which is dependent on the OS of the client machine:
- IANA & Windows 10 - 49152-65535
- Common on Linux machines - 32768-60999
Requests made by clients will contain the following information:
- Destination IP + fixed port - what you are connected to
- Payload - any information you send with the request
- Source IP + ephemeral port - where the request has come from and which ephemeral port to send return traffic to
The return request from the client will have similar information except the source IP + port will be swapped with the destination IP + port.

In order to set up NACLs to allow for the variance in ephemeral ports we need to define inbound/outbound rules that use a range of port numbers e.g. if we have an EC2 in our public subnet that makes requests to a database in our private subnet, we create the following rules:
- Outbound rule on the public subnet NACL to the database CIDR with the fixed port number of the database.
- Inbound rule on the private subnet NACL to the database CIDR with the fixed port number of the database.
- Outbound rule on the private subnet NACL to the EC2 CIDR with the range of ephemeral port numbers for that EC2.
- Inbound rule on the public subnet NACL to the EC2 CIDR with the range of ephemeral port numbers for that EC2.


## Security Groups
- Firewall that ALLOWS traffic in and out of an *ENI/EC2 instance*. 
- Rules include IP addresses and other SGs
- Return traffic from ALLOWED instances is automatically allowed (stateful) e.g. if you have already allowed outbound traffic to a particular IP, you do not need an additional inbound rule for the returning traffic from that IP. The rules in SGs are to allow the initial inbound/outbound request for the thing the SG is attached to.

## VPC Flow Log
Capture information about traffic going into your interfaces e.g. VPCs, subnets, ENIs.
Also captures data for traffic in and out of AWS-managed interfaces e.g. ELB, RDS, ElastiCache, RedShift, Workspace, NAT Gateway, Transit Gateway...
Helpful for troubleshooting connectivity issues.
Logs can be send to S3, CloudWatch Logs, or Kinesis Data Firehose.
Can pick out IPs that are repeatedly being rejected.
Queries can be run on Flow Logs using Athena on S3 for big analysis or CloudWatch Logs Insights for filtering/alarms based on metadata.
(For CloudWatch Logs integration) - requires an IAM role with full CloudWatch Logs access, a log group in CloudWatch Logs

## VPC Peering
Allows VPCs to communicate in same/different accounts when they do not have overlapping IP address ranges (CIDR) so that they behave as the same network.
VPC peering is not transitive so a connection between VPC A and VPC B and between VPC A and VPC C does not mean that VPC B and VPC C can communicate.
Becomes less useful the more VPCs you try to connect.
Allows you to use SGs from other accounts for the source in your SG rules.
Need to create routes in the main route table for each VPC allowing traffic to the CIDR range of the peered VPC by giving the CIDR range as the destination and the peering connection ID as the target.

## VPC Sharing
Allows multiple AWS accounts in the same organisation to make use of a VPC by sharing subnets that belong to that VPC. This allows each account to benefit from the increased trust that comes from multiple resources existing within the same VPC, thus allowing for less complicated networking.

## VPC Endpoints
Allow you to connect instances to AWS services/your own services using a private network instead of `www` which is public.
This allows for greater security and lower latency.
Can be very expensive to use NAT Gateways/instances to connect to AWS services as you pay for the data transferred through a NAT Gateway whereas VPC Endpoints cost nothing to run and you only pay for the data transferred in/out. 
It also means that you won't have to connect to S3 via the public internet so you won't have to pay the S3 egress costs either.
Creates a route in the route tables for the associated subnet(s).
- VPC Gateway Endpoint - for connecting to S3 and DynamoDB
	- Must be used as a target in a route table route.
	- Free
- VPC Interface Endpoint - for every other service or if you require access to S3/DynamoDB from on-prem servers/different region/different VPC
	- Provisions an ENI (private IP address) as an entry point but you must attach a security group.
	- Pay per hour running and per GB processed.

## PrivateLink
Used to connect AWS VPC to any other AWS VPC.
Requires Network Load Balancer on their end and an ENI on your end to create the PrivateLink connection.

## Site To Site VPN
Connect on-prem data centre to your VPC in AWS by establishing a VPN on the public network. The traffic is encrypted but can suffer from lower bandwidth and you may still have compliance concerns.
Requires a Customer Gateway (CGW, installation or physical device) set up on the data centre side and a Virtual Private Gateway (VGW)/Transit Gateway on the VPC in AWS.
Must connect the VPG to the CPG using the public IP of either the CGW directly or the NAT device in front of it.
Must enabled route propagation in your VPC's route table associated with the subnet you want to connect to your on-prem data centre.
If you need to ping your EC2s from on-prem servers after the VPN is set up, you need to enabled an inbound rule with the ICMP protocol on the SG of your EC2s.
#### VPN CloudHub
Low-cost hub and spoke model for VPN connections between multiple VPC subnets and your on-prem networks.
Only requires one VPG but routes in the route tables for each subnet you wish to connect to the hub.

## Direct Connect (DX)
Physical connection between your on-prem data centre and your VPC that doesn't use the public Internet so it is entirely private, secure, and fast.
Data comes in through a virtual private interface created on your VPC.
Requires a connection be set up at Direct Connect locations which can take about a month to set up. 
High resiliency requires a connection be set up in multiple Direct Connect locations. Maximum resiliency requires multiple connections being set at multiple Direct Connect locations.
Site-To-Site VPN can be used to create a quickly available backup if your Direct Connect fails.
Data is private but not encrypted in-transit. May want to also set up a VPN for added security.
Can't be used to extend your VPC. Must use Outposts to do so instead.
You must also create a VPG in your VPC.
Allows you to access public AWS resources such as S3 as well as private resources such as EC2s without a connection to the public internet.
Particularly useful if you have high bandwidth requirements if you are working with large datasets.
Provides more consistent network experience e.g. if you applications use real-time data feeds.
Supports IPv4 and IPv6.
you want to co-locate your Direct Connect location in the same AWS region as your services to minimise the egress costs.
#### Connection Types
- Dedicated connections - 1GB/s, 10GB/s, or 100GB/s
- Hosted connections - 50MB/s, 500MB/s, or 10GB/s. Capacity can be added/removed on-demand.
#### Direct Connect Gateway
Used to connect to one or more VPCs in different regions (within the same account).
The gateway connects to private virtual interfaces in each VPC.

## Transit Gateway
Hub-and-spoke style connection between 1000s of VPCs and on-prem infrastructure since networking topology can get very complicated.
More useful than VPC Peering in the case of wanting to connect all VPCs as VPC Peering is not transitive.
VPCs can be from multiple regions and you can even connect VPCs from multiple accounts using Resource Access Manager (RAM).
Can peer with other transit gateways in other regions.
Use route tables to control which VPCs can talk to each other.
Works with Direct Connect for on-prem data centres as well as VPN connections.
Supports IP Multicast (only supported by Transit Gateway, not any other AWS service).
Can use a Transit Gateway to share Direct Connect between AWS accounts to allow multiple VPCs  to forward traffic to your on-prem data centre.
#### Equal-Cost Multi-Path (ECMP) Routing
Can set up routing to a particular VPC/on-prem data centre that allows you to forward a packet of data over multiple potential VPN connections to allow for greater bandwidth (rate of data being sent).
By setting up multiple VPN connections, your packet will follow the best path e.g. if one VPN is overloaded, it will choose the other VPN connection.
Using a VPN with 2 tunnels connected to a VPG, you can get 1.25GB/s.
With multiple VPNs connected to a single transit gateway, you get double the throughput for a single VPN, now at 2.5GB/s but you can also add additional VPN connections that adds an additional 2.5GB/s to the throughput e.g. 3 VPN connections from one VPC to another via the transit gateway means you can have 7.5GB/s.
You will have to pay for the GBs of data sent.
## Traffic Mirroring
Allows you to send network traffic to security appliances you manage when it gets to the ENI of an EC2 instance. The traffic is then forwarded on to an ENI or a Network Load Balancer that sits in front of your security appliances auto-scaling group.
The source ENI and target ENI/NLB don't have to be in the same VPC but you would have to use VPC Peering/transit Gateway to allow your source to forward traffic on to another VPC.
Captures all packets or just some packets of interest using packet truncation.
Useful for content inspection, threat monitoring, troubleshooting.
## IPv6
IPv4 addresses are looking to be existed in the near future.
IPv6 is designed to allow for $3.4\times10^{38}$ uniques IPs.
Every IPv6 address is public and internet-routable i.e. there is no private range.
All VPCs use IPv4 but IPv6 can be enabled to be used alongside IPv4 addresses in "dual-stack mode".
You EC2s created in a dual-stack VPC will get at least a private IPv4 and a public IPv6.
## Egress-Only Internet Gateway
Similar to Internet Gateway in that it provides internet access for your VPC.
However, it only works for IPv6 addresses and doesn't require your instances to be routed to a NAT Gateway in a public subnet before being routes to the egress-only gateway.
Doesn't allow internet users to initiate a request going back to your private instances much like what the NAT Gateway is there to do for IPv4 addresses.
Need to update your private subnet route table to allow the subnet to access the internet.

## Network Firewall
Protects entire VPC from Layer 3 to Layer 7.
Allows you to inspect:
- VPC to VPC traffic
- Outbound traffic to the internet
- Inbound traffic from the internet
- Traffic sent to/from Direct Connect as well as Site-to-Site VPN
Supports 1000s of rules for the following:
- IP and port filtering
- Protocol blocking
- Stateful domain rule group e.g. only allow outbound traffic to .mycorp.com
- Pattern matching rules using regex
Allows for traffic filtering to allow, block, or alert based on traffic that meets certain rules.
Active flow inspection means you can protect from network intrusion.
Send logs to S3/CloudWatch Logs/Kinesis Date Firehose.

#aws #networking 