# Storage
## EBS Volumes
Storage devices attached to EC2s over a network and are mapped to a particular AZ.
They are provisioned with a particular size (can be increased while active) and IOPS.
Multiple volumes can be attached to one instance.
Root EBS volumes are deleted on termination by default.
Other attached EBS volumes are persisted after termination by default.
#### Snapshots
You can copy the data from an EBS volume in one AZ to an EBS volume in another AZ using snapshots.
For a cheaper alternative, you can put snapshots in the archive tier although it can takes between 24-72 hours to restore a volume of this kind.
Can specify retention periods of snapshots from 1 day to 1 year.
If you need a snapshot to be restore *immediately*, you can use fast snapshot restore. Very spenny.
Restoring volumes from a snapshot can help speed up application start times.
#### Volume Types
General Purpose SSD - gp2, gp3
Cost effective, low-latency storage
Sizes from 1GiB - 16TiB
IOPS: 3 IOPS per GB up to 3000 IOPS for gp2, 3000 IOPS up to 16,000 IOPS for gp3
Typically used for system boot volumes for virtual desktops, dev, and test envs.

Provisioned IOPS SSD - io1, io2, io2 Block Express
Size: 4GiB-16TiB for io1, io2 but up to 64TiB for io2 Block Express
IOPS: up to 64,000 for Nitro EC2s and 32,000 for anything else for io1, io2. up to 256,000 for io2 Block Express
Supports EBS Multi-Attach
Good for critical workloads with sustained IOPS like databases or for workloads with >16,000 IOPS requirements.

Throughput Optimised HDD - st1
Cannot be used as boot volume
125GiB-16TiB
Max throughput of 500MiB/s
Good for workloads with data transfer processes e.g. data warehousing, Big Data, log processing

Cold HDD - sc1
Cannot be used as boot volume
Cheapest storage option for infrequent data access
#### Multi-Attach
Can connect up to 16 EC2s within same AZ to a single io1/io2 (Block Express) EBS volume.
Useful for concurrent write operations.
File system must be cluster aware.
#### Encryption
EBS volumes and snapshots are encrypted at rest and in transit by default.
Can encrypt unencrypted volumes by taking a snapshot of the volume, encrypting the snapshot, restoring a new EBS volume from the snapshot and mounting the volume to the instance.

## Instance Store
Rather than attaching EBS volumes which are remote volumes attached to EC2s over a network connection, you can temporarily attach hardware disks physically to your EC2 using an EC2 Instance Store resource. 
They have much higher read/write speed (IOPS) than EBS volumes.
However, you are responsible for any backups as a precaution against hardware failure.
Should be restricted for temporary data storage requirements such as caching due to their ephemeral nature.

## Elastic File System
Similar to EBS volumes but they are network file systems (NFS) that store data across multiple AZs of the same region.
They can be accessed by 100s of EC2s from different AZs, or VPCs at the same time (**as long as they are Linux EC2s**) and even on-prem servers.
More expensive than individual EBS volumes and you pay for each use of the EFS. However, they can be cheaper overall if you use lifecycle policies to put files in Infrequent Access.
No need to provision particular storage amounts as they scale with demand.
Access is controlled by security groups.
Data is encrypted at rest using KMS keys.
 - EFS-InfrequentAccess allows you to more cheaply store files than EFS-Standard as it puts infrequently accessed files in cheaper storage resources.
Can choose one-zone (dev envs, backup enabled by default) or multi-AZ (prod envs)
#### Performance Mode (cannot change after creation)
- General purpose - for latency sensitive workloads e.g. webservers
- Max I/O - for increased parallel writes with impact to latency e.g. big data/video processing 
#### Throughput Mode
- Bursting - 50MiB/s with a burst of up to 100MiB/s
- Provisioned - choose a particular throughput regardless of storage used
- Elastic - scale throughput with demand for unpredictable workloads

## Amazon FSx
Allows you to give client resources outside of AWS access to a shared NFS.
#### Windows 
Supports SMB protocol and Windows NTFS.
Has Microsoft AD integration, ACLs, user quotas.
Can be mounted on Linux EC2s.
Supports Microsoft's Distributed File System Namespaces for grouping of files.
scales to 10s GB/s, 1,000,000s of IOPS, 100s PB of data of SSD or HDD.
Can be accessed via on-prem infrastructure (using VPN or Direct Connect).
High-availability is available as well
#### Lustre
Used for High-Performance Computing (HPC) e.g. video processing, financial modelling.
Similar scalability as FSx for Windows but with more focus on sub-millisecond latencies.
Seamless integration with S3 so you can read S3 like a filesystem and calculation outputs can be written back to S3.
Can also be accessed by on-prem infrastructure like FSx for Windows.
There are two deployment models for FSx for Lustre:
- Scratch File System - temporary storage for short-term processing that is not replicated so data is lost upon server failure. However, the bursting performance is much better (6x faster, 200MB/s per TiB)
- Persistent File System - longer-term storage where data is replicated within the same AZ and files can be replaced within minutes. Useful for long-term processing and sensitive data.
#### NetApp ONTAP
File system compatible with NFS, SMB, iSCI protocols for workloads running on ONTAP or NAS to AWS.
Compatible with many different OSs.
Storage auto-scales to meet demand.
Takes snapshot to provide replication with compression and data de-duplication. 
Cost-effective solution for point-in-time instantaneous cloning e.g. for testing new workloads.
#### OpenZFS
For workloads running on ZFS specifically.
Up to 1,000,000 IOPS with sub-0.5ms latency.
Provides snapshots and compression at a low-cost.
Also, useful for point-in-time instantaneous cloning e.g. for testing new workloads.

## Storage Gateway
Allows for interaction between on-premises applications and AWS storage by installing a gateway in your on-prem data centre on a virtual server (requires virtualisation).
If you don't have virtual servers, you will need to order a hardware version of the storage gateway. This hardware version works for File/Volume/Tape Gateway and can be useful for daily NFS backups in small data centres.
Useful for disaster recovery, backup + restore, tiered storage, on-prem cache + low-latency files access.
Data in-transit is automatically encrypted using SSL
Has 3 gateway types for different kinds of storage:
#### S3 File Gateway
Used to pull files from an on-prem server using SMB/NFS protocol before transfer files to/from S3 using HTTPS.
The most recently used data is cached in the file gateway.
SMB protocol also integrates with Microsoft AD for user authentication.
#### FSx File Gateway
Native access to Amazon FSx for Windows file servers.
Windows native compatibility.
May seem unnecessary since Windows file servers can already access FSx for Windows but you get the bonus of a local cache for frequently accessed data to grant you low-latency access to those files.
#### Tape Gateway
Some clients may back up their data to physical data tapes.
You can create a Virtual Tape Library by processing the tapes through a media changer/tape drive using the iSCSI protocol and replicating the data in S3.
#### Volume Gateway
Block storage using iSCSI protocol.
Backed up by EBS snapshots to help restore on-prem volumes.
Caching of recent data.
Comes in two modes:
- Cached volumes - low-latency access to most recent data queried from S3
- Stored volumes - entire dataset is on-prem with schedules backups made to S3

## AWS Transfer
Managed service for file transfers in/out of S3 or EFS using the FTP protocol.
There are 3 supported protocols covered by Transfer:
- FTP
- FTPS - FTP over SSL
- SFTP - secure FTP
- AS2
Scalable, highly-available over fully-managed infrastructure.
Pay per provisioned endpoint per hour (can optionally use Route53 endpoint instead) + however many GBs of transferred data.
Uses IAM roles to access S3/EFS storage.
User credentials stored + managed within the service.
Integrates with Microsoft AD, LDAP, Okta, Cognito, as well as custom systems.
Useful for sharing files such as public datasets, CRM data, Enterprise Resource Planning (ERP) data, or with 3rd parties.

## DataSync
Used to synchronise/replicate large amounts of active data via an installable agent to/from AWS such as on-prem, another cloud, or another AWS cloud (doesn't require agent installation).
Can synchronise to many AWS storage services including S3, EFS, FSx, Snowcone, EMR etc. on an hourly, daily, weekly schedule.
File permissions + metadata are preserved.
One agent can use up to 10GB/s of bandwidth so you can set a bandwidth limit.

#aws 