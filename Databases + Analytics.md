Databases are structured data stores that use indexes to rapidly query the data.

Relational database - data which is strongly linked to each other e.g. Excel, SQL, OLTP
	- RDS (+ Aurora)
Non-relational database - data with no given relation which gives the flexibility to set up specific data models e.g. NoSQL
	- DynamoDB (for JSON), ElastiCache (key/value pairs), Neptune (graphs), DocumentDB (for MongoDB), Keyspaces (for Apache Cassandra)
Object store - data dropped into a bucket where everything is stored at the same level and assigned prefixes.
	- S3 (for big objects), S3 Glacier (for backups/archives)
Data Warehouse - for SQL analytics and business intelligence workloads
	- Redshift (OLAP), Athena, EMR
Search - free text, unstructured searches
	- OpenSearch (for JSON)
Graphs - displays relationships between data
	- Neptune
Ledger - for financial transactions
	- Quantum Ledger Database
Time series
	- Timestream

A customer can configure backups of databases that AWS then performs.

## RDS
Relational database service - managed database service with automated patching, continuous backups (using io1/io2 EBS), monitoring dashboards, disaster recovery. Takes some of the database management away from the user. The user still manages the schema though.
Supports Oracle, Microsoft SQL Server, MySQL, Db2, PostgreSQL
IAM authentication not available for Oracle
*Can't SSH onto these databases*
Well-suited for Online Transaction Processing (OLTP).
Typically have load-balanced EC2s that will all perform read/writes to a single database.
Storage auto-scaling means that the database storage can scale to meet unpredictable demands. Gets increased to use larger EBS volumes if free storage is less than 10% of total for at least 5 minutes and 6 hours have passed since the last modification.
Can reserve instances like you can for EC2s.
#### Security
In-flight and at-rest encryption available.
Can make use of security group.
Audit logs are available to send to CloudWatch.
IAM authentication plugin can be used to provide short-lived credentials instead of username/password.
#### Read Replicas
Scale up to 15 replicas of your main database that are only used for read operations in order to lessen the load on the main database if many read/write operations are taking place. 
Can only run SELECT queries on replicas i.e. no INSERT, UPDATE, or DELETE queries are permitted.
Useful for scalability and for disaster recovery (in conjunction with multi-AZ deployments).
Replications occur asynchronously so the replicas are not always fully up to date but will get there eventually.
No additional cost for moving data to replicas between AZs within the same region as RDS is a managed service.
##### Multi-AZ 
Failover replica of your main database that is only accessed when there is a failover of your main database for whatever reason.
Uses synchronous replication and a single DNS name for the master DB and the replica.
These replicas can be created with zero downtime. A snapshot is taken of the master DB and restored to a read replica. Synchronisation begins after that.
##### Multi-Region
Replicas are in different regions so greater performance when reads happen across the globe. Writes happen across region as any writes are still made to the main database which incurs additional data transfer costs.
#### Backups
Automated full backup of DB allows for restoration of DB to a particular point in time from oldest backup to five minutes ago.
Automated backups are retained for 0-35 days. Use 0 days to disable backups.
Manual backups are retained as long as the user decides
To restore an on-prem MySQL DB into AWS, you need to take a backup and store it in S3. Then you can restore the backup to an RDS cluster.
This works the same for Aurora but it requires a different step of taking your backup using XtraBackup to create the backup.
Restoring from a snapshot can help speed up application start times.
#### RDS Custom
Partially managed Oracle/Microsoft SQL Server Database but you have access to the underlying OS and EC2 instance. This is the only one of the three types you can SSH onto.
#### Aurora
Proprietary relational database service only available on AWS for MySQL and PostgreSQL that offers greater performance than using MySQL (5x improvement) and PostgreSQL (3x improvement) on RDS. 20% more expensive than RDS but more efficient.
Storage auto-scales from 10GB -128TB
High-availability native - keeps 6 copies of data across 3 AZs (4 required for writes, 3 for reads)
Self-healing because of peer-to-peer replication.
Very quick failover repair times ~30s
Supports cross-region replication.
Specific columns/fields can be encrypted using the AWS Encryption SDK which utilises KMS keys.
Uses a single writer endpoint which doesn't change between failovers.
Read-replica-native - can also be accessed via a single read endpoints as well.
Can use "backtrack" to restore your database to a previous point in time using backups.
Backups cannot be disabled in Aurora.

Replica auto-scaling - creates additional read replicas to bring down the average CPU usage of the replicas.
Custom endpoints - some replicas may be more powerful and be more suited to analytical queries. Therefore you can define custom endpoints for certain types of queries to use the replicas best suited to those problems.
Serverless - useful for infrequent/intermittent workloads and pay per second per instance
Multi-master - have multiple writer endpoints for continuous write availability. Data is replicated between all master DBs.
Global Aurora - use one primary region and up to 5 secondary regions each with up to 16 read replicase in each region (can make for a total of up to 80!). Useful for decreasing latency for global applications. Typically takes less than a second for cross-region replication.
Machine learning - integrate with SageMaker or Comprehend for things like fraud detection, ad targeting etc.
Database cloning - used to quickly and cheaply stand up new database clusters without having to backup and restore. The new cluster DB is initially using the same storage volumes as the first DB but as new data is written to the clone DB, their storages are separated and the clone DB moves to its own storage. Useful for creating DBs for lower environments without affecting production.
##### Aurora Migrations
###### MySQL
- RDS MySQL -> Aurora MySQL:
	- Option 1: Take snapshots of RDS and restore them to a new Aurora MySQL database.
	- Option 2: Create an Aurora read-replica from your RDS database and promote the Aurora db to its own db cluster when the replication lag reaches 0 (more costly to achieve).
- On-prem MySQL -> Aurora MySQL:
	- Option 1: Percona XtraBackup creates a file backup in S3 which can then be restored to Aurora.
	- Option 2: Create an empty Aurora db and use the `mysqldump` utility to migrate directly to Aurora (slower than Option 1).
- DMS - use this if both the source and destination databases are running.
###### PostGreSQL
Same options as MySQL for RDS -> Aurora.
For on-prem databases, you can still back up to S3, but instead use the aws_s3 Aurora extension to restore to a new Aurora db.

#### RDS Proxy
Managed proxy server for RDS and Aurora.
Pools database connections and improves DB efficiency by reducing the stress on individual resources.
Helps reduce failover time by connecting to other database instances while one is terminating.
Enforces IAM authentication for DBs.

## Redshift
Data warehouse capable of processing petabytes of data used for PostgreSQL databases such as those used for Online Analytics Processing (OLAP). Not used for Online Transaction Processing (OLTP).
Can reserve nodes for 1/3 years.
10x better performance than other data warehouses and scales to PBs of data.
Integrates with BI tools such as QuickSight.
Uses columnar storage instead of row based and uses a parallel query engine.
Indexes mean queries can be run very efficiently
Better for complex aggregations + join queries.
Uses a leader node to plan queries and aggregate results. Compute nodes then perform the queries and send result back to the leader.
Multi-AZ is available for some cluster types.
Automated and manual snapshots are available.
Snapshots are also able to be moved to other regions (automatically or manually).
Better to write data to Redshift in large batches as it is more efficient, especially with enhanced VPC routing if you are using a VPC.
#### Redshift Spectrum
Allows you to query data straight from S3 buckets without having to load the data into Redshift first. You still need a Redshift cluster though.
The query is submitted to a leader node which then distributes the query to thousands of Redshift Spectrum nodes which run the query. This makes the query much more efficient.

## Athena
Severless SQL service to perform queries on S3 objects.
Used for business intelligence/analytics.
Priced at $5 per TB of data scanned.
Can be combined with QuickSight to create dashboards/reports.
#### Performance Improvements
Get a performance improvement by using columnar data to reduce the amount of data scanned. Use Glue to convert data to Parquet/ORC data.
Compress your data so the retrievals are smaller.
Partition data to make it more optimised for querying i.e. specific partitions to reduce the data used in common queries.
Use fewer, larger files as it is easier to retrieve than lots of smaller files.
#### Federated Queries
Can run SQL queries across relational, non-relational, and custom data sources from AWS and on-prem databases.
Use data source connectors that run on Lambda to run the queries and store the results in S3.

## DynamoDB
NoSQL database with replication across multiple AZs. 
Fast performance and a serverless database.
DynamoDB consists of tables which contain rows. The primary key for the rows must be decided upon table creation (must contain a partition key, but can contain an optional sort key), but that isn't necessarily the case for row attributes.
Each row can be up to 400kb and supports scalar (strings, numbers, binaries, booleans, null), document (lists, maps), and set (string sets, number sets, binary sets) data types for its keys/attributes.
Individual row attributes can be encrypted using the DynamoDB encryption client which uses KMS keys to encrypt specific attributes.
Possible to provision specific reserved capacity if you can estimate your throughput needs.
Schema-less databases are more flexible than databases with schemas e.g. SQL so DynamoDB is useful when you either have evolving schemes or no schemas at all.
Can enable TTL on items in tables so that they are removed after a certain expiry time. This helps you to only maintain current items in your tables, web session handling, and remove items due to any regulatory obligations.
Has disaster recovery options:
Point-in-time recovery (PITR):
- Continuous backups that can be optionally enabled for the last 35 days.
- Restoring from these backups creates a new table.
On-demand backups:
- Full backups for long-term retention (up to an expiry period).
- Doesn't affect performance/latency.
- Can be managed using AWS Backup for cross-region copying.
- Also creates a new table upon restoring tables from backups.
#### S3 Integration
##### Exporting data
Can integrate with S3 (must have PITR enabled) without affecting read capacity of your table so that you can perform data analytics using services that also integrate with S3 e.g. Athena. It's also useful for auditing as it means you can preserve your backups for longer. you can also perform ETL jobs on the data before importing the data back into DynamoDB.
Data must be exported in DynamoDB JSON or ION.
##### Importing data
Used to import CSV, DynamoDB JSON, or ION data into DynamoDB without consuming write capacity. Any import errors are logged in CloudWatch Logs.
#### Capacity Modes
Provisioned (default):
- Specify number of read/writes per second.
- Requires you to plan your capacity beforehand.
- Pay per provisioned Read Capacity Units (RCU) and Write Capacity Units (WCU).
- Can also add auto-scaling to your RCU/WCU.

On-demand:
- Read/writes automatically scale with your workloads.
- Doesn't require capacity planning.
- Pay for what you actually use but the rates are much higher.
- Useful for unpredictable workloads or ones that have steep spikes.
#### DynamoDB Accelerator (DAX)
Used to cache data from DynamoDB databases in-memory to decrease latency and prevent read congestion.
Doesn't require changing your application logic if it already works with DynamoDB.
5 minute TTL by default.
Useful for caching individual objects or single query/scan results.
Elasticache is more appropriate if you want to cache larger results such as aggregated results from many queries.
#### Stream Processing
Used to order item-level modifications in a table which can be applied to the following scenarios:
- Real-time reactions to changes e.g. welcome email to new users.
- Real-time usage analytics.
- Insertions into derivative tables.
- Enabling cross-region replication.
- Invoking Lambdas upon changes being made.

There are two methods for stream processing:
DynamoDB streams:
- 24 hour retention
- Can only have a limited number of customers
- Process streams using Lambda triggers or the DynamoDB stream Kinesis adapter
- Can be used to enable table replication globally for low latency across multiple regions using active-active replication (apps can read/write to tables in any region and changes are replicated everywhere else).

Kinesis data streams:
- 1 year retention
- Can have a much higher number of customers
- Can process streams using Lambda triggers, Kinesis data analytics, Kinesis data firehose, Glue streaming ETL

## OpenSearch
DynamoDB can only perform queries on primary keys or indexes. OpenSearch is able to search any field with only partial matches.
Useful when searching for items in data when you don't know how to locate that item explicitly, e.g. in non-relational data. You use OpenSearch to search for IDs of objects that have a specific attribute then your app can retrieve the entire item from your primary database by using that ID.
Can be provisioned as a serverless service or as a managed cluster.
Typically not used on its own but paired with other databases.
Doesn't natively support SQL but can be enabled with plugins.
Ingests data from Kinesis Data Firehose, IoT, and CloudWatch Logs.
Security is provided via Cognito & IAM (for user authentication/authorisation), KMS, and TLS.
Comes with dashboards.

## DocumentDB
Same as Aurora but for MongoDB (which is a NoSQL database) for storing/querying JSON data.
Fully managed highly available (3 AZ replication) .
Uses similar concepts to Aurora.
Automatically scales in increments of 10GB.
Scales for workloads with millions of requests per second.

## ElastiCache
Used for Redis and Memcached databases. Takes load away from read intensive databases by keeping a copy of a database queries in the memory of your EC2s.
Helps make applications stateless by allowing users to connect to any of your instances and their session data can be read from ElastiCache with sub-millisecond speeds due to data being stored in AWS rather than cookies.
Applications first query ElastiCache to see if queries have been performed then passing the query to RDS if it has not been done recently.
Can also use ElastiCache to store sessions for users accessing an application. They don't have to repeatedly log in if their session is saved to the cache.
Can reserve individual nodes for 1/3 years.
#### Redis
Multi-AZ with read replicas to scale reads and have high-availability.
Has backup and restore features.
Provides data durability using AOF persistence
Support Sets - guarantees uniqueness in data and element ordering. Useful for gaming leaderboards
Sorted Sets
#### Memcached
Multi-node for partitioning of data (sharding).
No HA, persistence, or backup and restore.
Multi-threaded architecture
#### Cache Security
IAM auth supported for Redis.
IAM policies used for API level security
Redis - can set a password/token for your cache to add security. Supports SSL in-flight
Memcached - Supports SASL authentication
#### Patterns
- Lazy loading - read data is cached but becomes stale
- Write through - adds/updates data in cache when data is written to a DB -> no stale data
- Session store - store temporary session in cache

## EMR
Elastic MapReduce is used to create Hadoop clusters for analysing big data.
Comes bundled with lots of typical Big Data tools e.g. Apache Spark, HBase, Presto, Flink.
EMR will provision and configure a fleet 100s of EC2 (using spot instances and auto-scaling).
Useful for data processing, ML, web indexing, big data.
Master node - manages the cluster, coordinates instances, and manages health (must be long-running).
Core nodes - runs tasks and stores data (also long-running)
Task nodes (optional) - just for running tasks (short-lived)
Pricing options include:
- On-demand if you need reliable workloads and can't predict them
- Reserved (minimum 1 year) for cost savings and predictable workloads which EMR will automatically use where possible.
- Spot instances for cheaper, disruptable tasks, but less reliable
You can provision a long-running cluster or a transient cluster.

## QuickSight
Serverless business intelligence service used to create interactive dashboards from database services.
Can be embedded in websites.
Integrated with RDS, Aurora, Athena, Redshift, S3, OpenSearch, Timestream, Salesforce, Jira, on-prem databases...
Use the SPICE in-memory computation engine when data is imported into QuickSight (won't work for dat hosted in other databases).
The enterprise version of QuickSight has Column-Level Security (CLS) to prevent users with inadequate IAM perms from viewing columns you don't want them to.
Has its own users (and groups in the Enterprise version) not in IAM.
A dashboard is a read-only snapshot of an analysis that preserved the config of the analysis.
Analyses/dashboards can be shared with users/groups (dashboard must be published first).
Anyone who has access to view a dashboard also has access to view the underlying data,

## Neptune
Managed graph database i.e. multi-relational databases (see graph on right of screen). 
Useful for knowledge graphs, recommendation engines, fraud detection, social networks etc.
Highly available across 3 AZs with up to 15 read replicas.
Optimal solution for highly connected datasets.
Can store billions of relations and can run queries with millisecond latency.

## Keyspaces
Apache Cassandra is an open-source NoSQL distributed database (CQL).
Keyspaces is a managed equivalent of Cassandra that scales its tables up/down according to demand.
Tables are replicated in 3 AZs.
Millisecond latency at any scale and capable of 1000s requests per second.
Has on-demand and provisioned modes.
Provides encryption and PITR for up to 35 day old data.
Useful for IoT device info, time-series data.

## QLDB
Quantum Ledger Database is a database for storing financial transactions and making sure no edits are made to existing records. Central database as opposed to blockchain which is decentralised.

## Managed Blockchain
Create decentralised blockchain networks. 
Compatible with Hyperledger Fabric and Ethereum blockchain frameworks.
Used to review the history of all changes made to application data over time.
No entry can be removed, or edited after they are created.
Fully managed serverless service that is highly available and has replication over 3 AZs.
2-3x better performance than common ledger blockchain frameworks.

## Timestream
Fully managed serverless time series database with autoscaling used to store and analyse trillions of events per day.
1000x faster than relational databases for a tenth of the cost.
Data storage tiering allows for cheaper storage of data that isn't required as often.
Built in time series analysis functions.
Encrypted data in transit and at rest.
Useful for IoT apps, operational applications and real-time analytics.
Connects to other apps using JDBC e.g. QuickSight, Sagemaker, Grafana

## Glue
Serverless service that extracts, transforms, and loads (ETL) data for analytics.
Stores operational metadata in Glue Data Catalog.
Useful for transformation jobs to convert data from one type into another.
Glue Job Bookmarks help prevent re-processing old data.
Glue Elastic Views is useful for replicating data across multiple stores using SQL.
Glue DataBrew cleans and normalises data using ready-made transformations.
Glue Studio is a GUI for running and monitoring jobs in Glue.
Glue Streaming ETL is used to stream data jobs rather than running them as batches.
## Lake Formation
Serverless service used to discover, cleanse, transform, and ingest data into a data lake in a matter of days.
Automates manual steps and de-duplicates data using ML transforms.
Combines structures and unstructured data into a single location.
Comes with out-of-the-box blueprints for S3, RDS, (No)SQL databases
Has fine-grained access control at the row and column-level so that any service that connects to Lake Formation can only view the rows/columns you permit it to view. This is very useful for centralising your data and being able to govern the permissions across your entire dataset.
Built on top of Glue and stores the data in S3.

## DMS 
Database Migration Service is used to migrate databases via EC2 instances.
Particularly useful for heterogenous migrations, e.g. Oracle to Microsoft SQL Server, as well as homogenous migrations.
Continuous replication to capture changes from your source database automatically.
Schema Conversion Tool allows your to convert the schema of the data from one engine to another during the migration (NOTE: the database engine is different from the database platform e.g. RDS can be running the Db2/MariaDB/Microsoft SQL Server/MySQL/Oracle/PostgreSQL engines).

#aws #blockchain #databases #redshift