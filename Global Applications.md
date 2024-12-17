# Global Applications
Better to deploy apps globally so that global users have decreased latency.
Distributed applications also provide better disaster recovery and are harder to attack since they would have to attack all data centres at once.
Some AWS services are required to be global in scope such as IAM, CloudFront, Route 53 and WAF.

*NOTE: It is not always possible to deploy global applications e.g. if you have to keep data within a particular country due to compliance reasons or if some products are only available in specific regions.*

Regions - clusters of data centres (AZs) distributed around a particular region e.g. eu-west-2 (London).
Availability Zones - each AZ is one or more discrete data centres that are separated from other AZs to provide disaster protection for your services. They are connected via high bandwidth, ultra-low latency networks.
Points of Presence (Edge locations) - used to provide low latency connections to AWS services
## Route53
Authoritative Managed Domain Name System (DNS) that help users reach a server through various URLs.
By authoritative, we mean that the user is able to modify the DNS records.
Can purchase a domain through a domain registrar e.g. GoDaddy but create your hosted zone and configure your DNS records yourself using Route53.
Minimises latency by looking at user location and giving IP for closest server.
Helps with failover of servers using routing policies.
Provides health monitoring of applications.
You pay for each request made to Route53.
Provides domain name resolution at the Edge which helps mitigate the effects of DDoS attacks.
#### Records
Can use `dig` and `nslookup` to query a record for its associated records.
Records consist of the following:
- Domain/subdomain name e.g. google.com
- Record type:
	- A - maps to an IPv4 address
	- AAAA - routes to an IPv6 address
	- CNAME - maps to hostname to another hostname which must have an A or AAAA record
		As long as it's not a root domain e.g. example.com :( sub.example.com :)
	- NS - maps to named servers for a specific hosted zone
	- Alias - A or AAAA records that maps hostname to AWS service and automatically picks up resource IP changes
		Free to query
		Built in health checks
		Can work on root domains as well e.g. example.com, sub.example.com :)
		Targets include ELBs, CloudFront distributions, API Gateway, Elastic Beanstalk envs, S3 websites, VPC Interface endpoints, Global Accelerator accelerators, Route53 records in same hosted zone. Not EC2 DNS names!!!
- Value i.e. the IP address we are routing to
- Routing policy
- TTL (Time to level) - amount of time the record is cached at the DNS resolvers. Mandatory for all records except Alias records.
	Higher TTLs mean less queries are made to Route53. However, some records may become outdated and direct users to the wrong IP address. You can decrease the TTL for your servers so that they refresh quickly, update your servers so they take on new IPs, then extend the TTL again to reduce costs.
#### Hosted Zones
Hosted zones contain several records which say how to connect to a domain and its subdomains
- Public hosted zone - can access via the Internet
- Private hosted zone - has to be accessed from an organisation's VPC
#### DNS
Domain names have a hierarchy as to have they are evaluated:
- Top level domains e.g. .com, .org, .gov
- Second level domains e.g. example.com
- Sub domains www.example.com
- Fully qualified domain names (FQDN) e.g. api.www.example.com
A FQDN fronted with a protocol (e.g. http://) is called a URL.

DNS works in the following way:
1. A users web browser will attempt to reach a URL by querying the local DNS server (either created by employer or internet service provider).
2. The local DNS server will then query a root DNS server (managed by ICANN) for the whole URL. The root DNS server may not know how to resolve the full URL, but it will know the IP of the appropriate TLD DNS server (e.g. the .com server, managed by IANA; a branch of ICANN).
3. The local DNS server then queries the TLD DNS server for the same URL. The TLD DNS server may not know how to resolve the full URL, but it will know the IP of the correct SLD DNS server (e.g. example.com, managed by a domain registrar).
4. The local DNS server then queries the SLD DNS server for the URL and the SLD DNS server will return the public IP of the web server that corresponds to the URL. The local DNS server passes this back to the user's web browser and both the browser and the local DNS server cache the IP of the web server for future use.
#### Routing Policies
Determines how Route53 queries are responded to.
##### Simple
Typically for one record-one resource scenarios.
Can specify multiple values in the same record and one is picked at random to route to.
When used with Alias records, you should only point to one AWS resource.
Only policy that cannot use health checks.
##### Weighted
Control percentage of requests that go to each target by creating several records of the same name with a particular weight associated with it.
Each record that has the same name must be of the same type.
An individual weight of 0 will stop traffic going to that target. When all records have a weight of 0, the traffic is distributed evenly.
##### Latency-based
Redirect to resource with lowest latency to person making request.
##### Geolocation
Dictate which resources a user's traffic will go to based on the user's location.
Separate records point the user to the target they are closest too geographically with a default record for if a location cannot be matched up.
Useful for region restricted content e.g. Netflix
##### Geoproximity
Route traffic from users based on which resources are closest, but can apply a bias to individual targets so more/less traffic will go to them.
##### Failover (Active-Passive)
Configure a record pointing to a primary target with a mandatory health check. If that target becomes unhealthy, the record name will then begin routing requests to a target in a secondary record.
##### IP-Based
Map endpoints to specific client IP addresses.
Can be good for reducing networking costs and improving performance if the client IPs are known ahead of time.
##### Multi-Value
Used for routing to multiple resources.
Similar to simple routing but with health checks enabled.
Unhealthy resources are blocked from queries
NOT A SUBSTITUTE FOR LOAD BALANCING
#### Health Checks
Can only be used to directly monitor failovers on public resources.
Pay based on frequency of health checks.
Automated DNS Failover:
1. Endpoint monitoring - 15 global health checkers in different regions will monitor an endpoint.
	Health is judged based on a threshold e.g. 3 positive responses or percentage of total responses
	Can choose the regions you send checks from
	Pass when you get 2xx or 3xx HTTP codes
	Can also be based on the first 5120 bytes of text based responses
	CIDR range of health checkers must be allowed in security group of resource
2. Calculated health check that monitor other health checks
	Combine results of up to 256 health checks using OR, AND, or NOT statements
	Useful while performing maintenance but don't want health checks to fail
3. CloudWatch alarms for monitoring private resources
	Route53 health checkers exist in the public VPC so cannot exist resources in a private VPC
	Create a CloudWatch Metric which can be associated with a CloudWatch Alarm to alert you to unhealthy metrics.

## CloudFront
Content Delivery Network (CDN) that caches content at all of the 216 Points of Presence to improve user experience by increasing read speed.
Provides protection against Distributed Denial of Service (DDoS) attacks since content is distributed.
Also has integration with Shield and AWS Web Application Firewall (WAF).
Requests sent to a particular DNS will go through CloudFront first rather than straight to the origin and will see if it already has the result cached. If it doesn't have the result cached, CloudFront will query the origin and will store the result in its cache for later use.
Can provide cost advantages for S3 downloads if your applications will support data being hosted in Edge locations.
#### Origins
- S3 bucket - for distributing files and caching at the edge. Has enhanced security thanks to Origin Access Control (which replaces Origin Access Identity) which can enforce a policy that only allows traffic to go through your CloudFront distribution with your S3 only accepting traffic from your OAC. CloudFront can be used as an ingress for uploading files to S3. This is the preferred approach over something like S3 Cross-Region Replication if you have static data required globally that can be refreshed after a TTL period.
- Custom origin (HTTP) - can be used in front of a custom backend for public ALBs (with instances in a private *or* public subnet), public EC2 instances, S3 static websites, or any other HTTP backend. In these cases you must make sure that you have set up inbound SG rules that allow for all the CloudFront instances to connect to the origin.
#### Geo Restriction
Useful for copyright restricted content.
Allowlist - users with IPs from these geos will be allowed to access your distribution.
Blocklist - users with IPs from these geos will *not* be allowed to access your distribution.
The countries are determined by a 3rd party Geo-IP database.
#### Pricing
The cost of data out will vary for each region you are exporting data from.
You can reduce the number of CloudFront edge locations to distribute data from to reduce monthly costs.
There are the following price classes available for CloudFront locations:
- 100 - only the least expensive regions (USA, Mexico, Canada + Europe, Israel).
- 200 - most regions excluding the most expensive ones (100 + South Africa, Kenya, Middle East + Japan + Hong Kong, Philippines, Singapore, South Korea, Taiwan, Thailand + India).
- All - all regions for the best performance globally (200 + South America + Australia, New Zealand).
#### Cache Invalidation
You can force a refresh that ignores the TTL of your caches by using a cache invalidation that can apply to all objects in a bucket or the objects on a given path. This removes those files from the caches of Edge locations so that CloudFront has to refresh the content.
## Global Accelerator
For a global application hosted in a particular region, requests from far off locations can require many "hops" from router to router in order to get a request to its destination which increases latency.
Global accelerator uses the concept of Anycast IP (servers have the same IP and the user is directed to the nearest one, as opposed to Unicast IP where servers have their own IPs).
Uses AWS internal network in a similar way to CloudFront to speed up requests made to your application by passing your request through edge locations before hitting your application.
Works as an alternative to CloudFront if your backend won't work with CloudFront.
The distributed nature of Global Accelerator helps to mitigate DDoS attacks.
Clients will route to one of 2 fixed IPs (that correspond to a regionally located resource which can be public/private Elastic IPs, EC2 instances, ALBs, NLBs).
The 2 IPs will have a weighting assigned to them which will send traffic to their nearest Edge location and then route traffic to your actual application resources.

Works for public or private networks.
Performs health checks on your applications which help with < 1min failovers and quick disaster recovery so you will only be routed to healthy instances.
Removes issues with caching and IP management since the IPs don't change.
Means only 2 IPs need to be whitelisted.
Also integrates with Shield and WAF.

## S3 Transfer Acceleration
Upload/download with S3 buckets faster by using edge locations as an intermediary step.


## Outposts
Fully managed service that allows you to access AWS services from your on-prem servers. *You* are now responsible for the security of your infrastructure, not AWS.

## Wavelength
Infrastructure deployments at 5G edge data centres for ultra-low latency apps.

## Local Zones
Extend AWS regions to local zones e.g. us-east-1 is in North Virginia but you can place selected AWS services in local zones such as Boston, Chicago, Dallas to reduce latency.

## Global Applications Architecture
Multi-region, active-passive - instances active in one region but passive in others which means that you can read from all instances but only write to the active instances.
	- Low global read latency
	- High global write latency
	- Multiple regions therefore moderate difficulty to architect

Multi-region, active-active - instances active in all regions which means you can read/write to instances in all regions
	- Low global read latency
	- Low global write latency
	- Increased difficulty to architect due to multiple regions and all instances being active

#aws #dns 