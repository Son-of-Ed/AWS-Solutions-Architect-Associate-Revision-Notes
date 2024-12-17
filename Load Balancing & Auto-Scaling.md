# Load Balancing + Auto-Scaling
High Availability - running your system across 2 or more AZs to prevent outages/data loss in the event of data centre loss.
## Scalability
How well a system can scale up (increase performance of instances) or scale out (increase number of instances).
**Vertical Scalability** - increasing the size/capacity of the instances you have e.g. for non-distributed systems like databases. Typically will hit a hardware limit at some point; instances can only get so powerful.
**Horizontal Scalability (Elasticity)** - increasing the number of instances you have i.e. distributed systems. Can be achieved with auto-scaling groups and load-balancers to distribute traffic between the created instances. High availability is achieved by incorporating the use of multiple AZs into your scaling group/load balancers.

**Dynamic scaling** - automatically scaling capacity in response to changes in load or other metrics.
**Predictive scaling** - increasing/decreasing resources ahead of time based on past behaviour of usage.
**Scheduled scaling** - scaling resources based on fixed schedule.

## Load-Balancing
Load balancers are servers that expose a single point of entry (DNS) to users and send the traffic to multiple instances within a target group based on balancing policies.
*Be aware that load balancers do not have a fixed IP*
Perform frequent health-checks on your instances to check if they are ready to accept requests by checking for HTTP 200 responses.
They separate the failure of individual instances from the user since requests get transferred to healthy instances.
Help separate public traffic from private traffic. They can be configured to distribute any traffic whether it comes from the internet or from your internal network.
ELBs are managed load balancers and come in a few flavours (it's recommended to use the newest ones i.e. Gateway LB):
	- Classic LB (now not available on AWS anymore)
	- Application LB (HTTP, HTTPS, Websocket) - Layer 7
	- Network LB (TCP, TLS, UDP) - Layer 4
	- Gateway LB (IP Protocol)- Layer 3
ELBs are configured with security groups which dictate where the traffic is allowed to come from and which protocols the traffic can use (can restrict to just HTTPS if you like). EC2 instances can then have their security group configured to only allow traffic coming from the ELB's security group (the source will be the ELB SG instead of a CIDR range).
You need at least one listener on your LBs and this should be set to the port you are expecting to receive inbound traffic through.
#### Application Load Balancers
Can distribute traffic to different machines or to multiple applications within the same machine (as in the case where you run multiple containers from one instance).
Used for HTTP(S) connections.
Route tables are based on URL path, URL hostname, or the headers/query string in URLs.
ELBs cannot route based on user geography.
Good for micro-services/container-based applications because of mapping to dynamic ports on an ECS instance.
Works for one application or multiple.
Target groups can include a mix of EC2s, ECR tasks, Lambda functions, private IPs.
An ALB can have one or many target groups assigned to it.
Health checks are done per target group.
Does not use a fixed IP address.
#### Network Load Balancers
Lower latency than ALB.
Used for TCP/UDP (basically anything not HTTP/HTTPS).
Has one static IP per AZ and supports Elastic IP (if you need to whitelist specific IPs).
By using private IPs in your target groups, you can include your own servers as long as they exist within the AWS network.
you may even want to put an NLB in front of an existing ALB so you can use the rules of your ALB and use a fixed Elastic IP of your ALB.
#### Gateway Load Balancers
Used to inspect the traffic coming into your network or for packet manipulation.
It does this by passing the traffic to a target group consisting of a security application. If the traffic passes the security checks, the traffic is sent back to the GLB before being sent on to your application.
Can also assign instance IDs or private IPs to its target group like the NLB.
Specifically uses the GENEVE protocol on port 6081.

## Sticky Sessions (Session Affinity)
It is possible to direct a user to the same instance behind a load balancer which is controlled by a cookie. The cookie has an expiry date set by you.
This can help users stay in the same session so that data isn't lost e.g. they won't have to log in again.
However, this can bring imbalance to the load against your backend machines.
Some cookie names are reserved for ELBs: AWSALB, AWSALBAPP, AWSALBTG.
If you are relying on cookies to store large amounts of data, this can slow down your application if you are having to frequently transfer large amounts of data over your user's network connection.
Cookie size should be limited.

## Cross-Zone Load Balancing
With cross-zone LB enabled, you can distribute your traffic evenly across ELBs in different AZs. 
Even if you have 2 instances in one AZ and 8 in another, the ELBs will distribute the traffic evenly between them.
Without cross-zone LB enabled, the ELBs can only distribute the traffic among the instances within their AZ.
This is only enabled by default for ALBs (but can be disabled per target group).
For ALBs, there are no charges for inter-AZ data transfer.
For NLBs and GLBs, you pay for inter-AZ data transfer when it is enabled.

## SSL + TLS
Secure Sockets Layer - SSL
Transport Layer Security - TLS
Most applications nowadays will use TLS but SSL is still used as the general name for in-transit traffic encryption.
A public SSL certificate is issued by a Certificate Authority to allow for secure traffic and the certificates will expire after a time of your choosing so they need to be renewed.
Load balancers use X.509 certs which are managed in AWS Certificate Manager (but you can upload your own certs if you choose).
#### Server Name Indication
Users can use Server Name Indication (SNI) to specify the hostname they are trying to reach.
SNI tells the load balancer which application they are trying to reach. The LB will then grab the appropriate cert from the ACM and the user's traffic can then go to the correct application and is encrypted so other applications cannot view the data since it is encrypted.
Only possible for ALB + NLB, not CLB.

## Connection Draining/Deregistration Delay
Used to complete in-flight requests before terminating/de-registering an EC2 instance.
The ELB will stop sending new requests to an EC2 in "draining mode" and send the traffic to other instances.
Can set this draining period to between 0-3600s.
Set to 0s to disable connection draining.
Set to short periods for short-lived requests or longer for long-lived requests.
Longer draining periods mean that it will take longer for instances to terminate.


## Auto-Scaling Groups
ASGs handle changing load requests for your EC2s by horizontally scaling up or down the number of EC2s you have but do not alter the type of EC2 you are using. They can also replace unhealthy EC2s with new ones.
Can scale ASGs based on CloudWatch alarms which can monitor specific metrics of your instances.
You define a minimum, desired, and maximum capacity.
#### Dynamic Scaling Policies
- Target Tracking - pick a metric target for the ASG to aim towards e.g. I want average CPU usage to stay at 40%
- Simple/Step Scaling - when particular CloudWatch alarm notifications occur, do something e.g. add 2 units when average CPU usage is at > 70% or remove a unit when average CPU usage < 20%.
- Scheduled Actions - perform a certain action at a particular time e.g. add 3 units at 4:45pm on Fridays
- Predictive Scaling - uses machine learning to predict usage patterns and scale appropriately.

Good metrics to measure your ASG performance on are:
- CPU utilisation - obvs
- Requests per target - if you know the optimal number of requests to send to a target from testing, you may want to scale up or down to meet that.
- Average network in/out - if you have network limitations, this can be good for avoiding bottlenecks
- Custom metrics - can also configure other metrics suited to your use case

Scaling activities have cooldown periods (default 300s) to allow metrics to stabilise so the ASG won't start/terminate additional instances during this period.

#aws