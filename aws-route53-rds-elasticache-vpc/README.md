# Route 53
* Managed DNS(Domain Name System)
* DNS is a collection of rules and records which helps clients understand how to reach a server through URLs
* In AWS, most common records are:
	* **A**: URL to IPv4
	* **AAAA**: URL to IPv6
	* **CNAME**: URL to URL
	* **Alias**: URL to AWS resource
* Route53 can use:
	* public domain names you own(or buy)
	* private domain names that can be resolved by your instances in your VPCs
* Route53 has advanced features such as:
	* Load Balancing(through DNS - client load balancing)
	* Health checks(limited)
	* Routing policy(simple, failover, geolocation, geoproximity, latency, weighted)
* Prefer `Alias` over `CNAME` for AWS resources(for performance reasons)


# RDS
* RDS stands for Relational Database Service
* It's a managed DB service for DB, use SQL as a query language
* It allows you to create databases in the cloud that are managed by AWS
	* Postgres
	* Oracle
	* MySQL
	* MariaDB
	* Microsoft SQL Server
	* Aurora(AWS proprietary database) 

## Advantage over using RDS versus deploying DB on EC2
* Managed service:
	* OS patching level
	* Continuous backups and restore to specific timestamps (Point in Time Restore)
	* Monitoring Dashboard
	* Read replicas for improved read performance
	* Multi AZ setup for DR(Disaster Recovery)
	* Maintenance windows for upgrades
	* Scaling capability(vertical and horizontal)
	* But you can't SSH into your instances

## RDS Read Replicas for read scalability
* Up to 5 Read Replicas
* Within AZ, Cross AZ or Cross Region
* Replication is **ASYNC**, so reads are eventually consistent
* Replicas can be promoted to their own DB
* Applications must update the connection string to leverage read replicas
* Only 1 master which takes the writes

## RDS Multi AZ(Disaster Recovery)
* **SYNC** replication to 1 DB instance which is the Standby in different AZ
* One DNS name - automatic app failover to standby
* Failover in case of loss of AZ, loss of network, instance or storage failure
* No manual intervention in apps
* Not used for scaling

## RDS Backups
* Backups are automatically enabled in RDS
* Automated backups:
	* Daily full snapshot of the database
	* Capture transaction logs in real time
	* Ability to restore to any point in time
	* 7 days retention(can be increased to 35 days)
* DB Snapshots
	* Manually triggered by the user
	* Retention of backup for as long as you want

## RDS Encryption
* Encryption at rest capability with AWS KMS - AES-256 encryption
* SSL certificates to encrypt data to RDS in flight
* To enforce SSL:
	* PostgreSQL : `rds.force_ssl=1 in the AWS RDS Console`
	* MySQL: Within the DB: `GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;`
* To connect using SSL:
	* Provide the SSL Trust certificate(can be downloaded from AWS)
	* Provide SSL options when connecting to database

## RDS Security
* RDS databases are usually deployed within a private subnet, not in a public one
* RDS Security works by leveraging security groups - it controls who can communicate with RDS
* IAM Policies help control who can manage AWS RDS
* Traditional Username and password can be used to login to the database
* IAM users can now be used too for MySQL and Aurora

## RDS vs Aurora
* Aurora is a proprietary technology from AWS(not open sourced)
* Postgres and MySQL are both supported as Aurora DB(that means your drivers will work as if Aurora was a Postgres or MySQL database)
* Aurora is `AWS cloud optimized` and claims 5x performance improvement over MySQL on RDS, over 3x the performance of Postgres on RDS
* Aurora storage automatically grows in increments of 10GB up to 64TB
* Aurora can have 15 replicas while MySQL has 5, and the replication process is faster(sub 10ms replica lag)
* Failover in Aurora is instantaneous. It's HA(High Availability) native.
* Aurora costs more than RDS(20% more) - but is more efficient

# ElastiCache
* ElastiCache is managed cache equivalent to RDS
* Either **Redis** or **Memcached**
* Caches are in-memory databases with really high performance, low latency
* Help reduce load off of databases for read intensive workloads
* Help make your application stateless
* Write Scaling using sharding
* Read Scaling using Read Replicas
* Multi AZ with Failover Capability
* AWS takes care of OS maintenance/patchin, optimizations, setup, configuration, monitoring, failure recovery and backups

## ElastiCache Solution Architecture

### DB Cache
* Application queries ElastiCache, if not available, get from RDS and store in ElastiCache
* Helps relieve load in RDS
* Cache must have an invalidation strategy to make sure only the most current and relevant data is in the Cache

### User Session Store
* User logs into any of the application instance
* The application writes the session data into ElastiCache
* When the user hits another instance of our application, the application retrieves session
* The instances retrieves the data and the user is already logged in

Common usecase is to relive load of the Database and to share some state such as user session store

## Redis Overview
* Redis is an in-memory key-value store
* Super low latency(sub ms)
* Cache survive reboots by default(it's called **persistence**
* Great to host:
	* User sessions
	* Leaderboards
	* Distributed states
	* Relieve pressure on databases
	* Pub/Sub capability for messaging
* Multi AZ with Automatic Failover for disaster recovery if you don't want to lose your cache data
* Support for Read Replicas

## Memcached Overview
* Memcached is an in-memory object store
* Cache doesn't survive reboots
* Use cases:
	* Quick retrieval of objects from memory
	* Cache often accessed objects
* Overall, Redis has largely grown in popularity and has better feature sets than Memcached

## ElastiCache patterns
* ElastiCache is helpful for _read-heavy_ application workloads
	* social networks, gaming, media sharing, Q&A portals
* Compute-intensive workloads (recommendation engines)
* There are two patterns/cache strategies for ElastiCache
	* Lazy Loading
	* Write Through
* Strategies may be different based on the kind of application you have

### Lazy Loading 
* Load only when necessary
* Pros
	* Only requested data is cached(the cache isn't filled up with unused data)
	* Node failures are not fatal(just increased latency to warm the cache)
* Cons
	* Cache miss penalty that results in 3 round trips, noticeable delay for that request
	* Stale data: data can be updated in the database and outdated in cache
		* To deal with this implement TTL, set timer for data to be considered stale

### Write Through 
* Add or Update cache when database is updated
* Write
	* write to DB
	* write to cache after 
* Pros
	* Data in cache is never stale
	* Write penalty vs Read penalty(each write requires 2 calls)
* Cons
	* Missing Data until it is added/updated in the DB.
		* Mitigation is to implement Lazy Loading strategy as well
	* Cache churn - a lot of the data will be read
		* Cache big will be possibly as big as database

# VPC
* Within a Region, you're able to create VPCs(Virtual Private Cloud)
* Each VPC contains subnets(networks)
* Each subnet must be mapped to an AZ
* It's common to have a public subnet(public IP) and a private subnet(private IP)
* It's common to have many subnet per AZ
* Public Subnets usually contains:
	* Load Balancers
	* Static Websites
	* Files
	* Public Authentication Layers
* Private Subnets usually contains
	* Web application servers
	* Databases
* Public and private subnets can communicate if they are in the same VPC
* All new accounts come with a default VPC
* It's possible to use a VPN to connect to a VPC
* VPC Flow Logs allow you to monitor the traffic within, in and out of your VPC(useful for security, performance, audit)
* VPC are per Account per Region
* Subnets are per VPC per AZ
* Some AWS resources can be deployed in VPC while others can't
* You can peer VPC(within or across accounts) to make it look like they're part of the same network
