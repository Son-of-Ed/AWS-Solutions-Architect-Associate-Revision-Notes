# Disaster Recovery
Recovery Point Objective - how often your run backups. Time between last backup and when disaster occurs equates to some data loss taking place.
Recovery Time Objective - how long it takes for your service to be up and running after an event. Time between the disaster occurring and the services being operational again equates to the downtime of your service.

## Disaster Recovery Strategies
- Backup and restore - data is backed up but there aren't any backup applications running in the event of your main instances going down (High RPO + High RTO, but cheap).
- Pilot light - application running on-prem with a smaller version of your core functionality is running on cloud instances with minimal setup/size e.g. smallest group of instances required.
- Warm standby - application running on-prem with a full setup of app running on cloud instances with minimum setup/size.
- Multi-site/Hot site - full version of app running on both on-prem and cloud instances at maximum setup/size.
- Multi-region cloud setup - similar to multi-site but with no on-prem infrastructure, everything is in the cloud.
#### Tips
- Backups
	- EBS/RDS snapshots
	- Regular pushes to S3 with replication
	- Snowball or Storage Gateway to back up from on-prem.
- High Availability
	- Route53 to migrate DNS from failing region to a different one
	- RDS/ElastiCache multi-AZ
	- EFS
	- S3
	- Site-to-Site VPN to recovery from Direct Connect failover
- Replication
	- RDS cross-region replication
	- DMS for on-prem databases
	- Storage Gateway
- Automation
	- CloudFormation/Elastic Beanstalk to recreate environments quickly
	- Recover/reboot EC2 with CloudWatch on activated alarms
- Chaos
	- Try random EC2 termination in all environments including PROD!

## Backup
Automated service to manage and automate backup processes across AWS services.
Support point-in-time recovery, i.e. restore instance to state at given point in time, as well as scheduled and on-demand backups.
Uses tag-based backup policies.
Uses backup plans that configure the following about your backups:
- Frequency of backups
- Duration of each backup
- Transition to cold storage
- Retention period of backups
Works with EC2, EBS, S3, RDS/Aurora, DynamoDB, DocumentDB, Neptune, EFS/FSx, Storage Gateway.
Supports cross-account and cross-region backups.
Can enforce a WORM policy to prevent delete operations on your backups in your backup vault in S3.

## Elastic Disaster Recovery
Block-level replication of servers from corporate data centre into AWS via a replication agent installed in your data centre that is able to write to AWS in seconds.
A staging environment keeps a backup of your apps running on low-cost EC2s and EBS volumes then will scale to the target EC2s and EBS volumes in the event of a failover (which can take minutes).

## Application Discovery Service
Gathers info about your on-prem applications to assist in planning a cloud migration.
Builds a picture of server utilisation and dependency maps which can be tracked within the Migration Hub.
Can use agentless or agent-based discovery depending on the privacy/granularity requirements of your migration (although the agent-based discovery will be able to gather more data).

## Application Migration Service
Lift-and-shift solution for cloud migration.
Works in the same way as Elastic Disaster Recovery but instead of a failover to trigger the target instances, we have a cutover event to trigger the shift to the full size instances.

## DataSync
Move large amounts of data from on-premises servers to AWS.
Large initial load followed by incremental uploads in future.

#aws 