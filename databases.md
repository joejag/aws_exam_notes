# Databases 101

- Relational databases on AWS
  - MS SQL, Oracle, MySQL, PostgreSQL, Aurora, MariaDB
- RDS has two key features: Multi-AZ (disaster recovery) and Read Replicas (perf)
  - Multi-AZ: AWS auto detects failure and updates DNS to point to secondary
  - Read Replicas: Scale out, but connect via different DNS name
    - Up to 5 copies of the read replica
- Non-Relational databases on AWS
  - Theory from RDS
    - Collection == Table
    - Documents == Row
    - Key/Value Pairs == Columns
- Data Warehousing
  - BI, large data sets, usually for management
  - OTLP (Online Transaction Processing): Single records
  - OLAP (Online Analytics Analutics Processing): Aggregates of large amount of records
  - Data Warehousing use different architecture from a DB and infra perspective
  - Use one type of DB for customers and another for analytics
  - AWS Redshift is Amazon's solution (OLAP)
- Elasticache
  - In-memory caching service. Cache common complex queires to reduce load of dbs
  - Redis and Memcached
- Summary
  - RDS (OLTP): Six types
  - DynamodDB (NoSQL)
  - Redshidt (OLAP): BI or DW
  - Elasticache (in memory cache): Caches frequent identical queries

# RDS Lab

- The 't' series are 'burstable' which incure CPU credit cost under load
- Databases can be optimised for prod, dev, or free
- The storage can autoscale when a threshold is met
- Databases cannot move VPC
- You can select a maintainance window
- Creating an RDS server generates a SecurityGroup
- You need to add the connecting SG into the RDS SG as a custom source
- Summary:
  - RDS runs on virtual machines which you cannnot access
  - Patching RDS is Amazon's responsibility
  - RDS is not Serverless
  - Aurora Serverless is
  - RDS instances do not appear in the EC2 dashboard

# RDS Backups, Multi-AZ, Read Replicas

- Two types of backups: Automated, Snapshots
- Automated:
  - "retention period" 1-35 days. Full daily backup with transaction logs down to within a second
  - Enabled by default. Stored in S3. Free S3 up to your RDS instance size. 10Gb == 10Gb
  - Backups are during a window. During this storage I/O may be suspended, so you can experience latency
- Snapshots:
  - Manually. Stored even if you delete the RDS instance. Automated backups do not do this.
  - When you delete the DB it will prompt you to make a final snapshot
  - When you restore from manual/auto it creates a new RDS instance with a new DNS endpoint
- Encryption at rest:
  - Available on all RDS types. Uses Key Management Service (KMS)
  - Applies to backups, read replicas and snapshots
- Multi AZ
  - RDS auto copies to another AZ. Auto fail over.
  - Writes are synchronous to standby database
  - In the event of maintaince window, DB failure, AZ failure it auto failsover
  - For failover, not for performance
  - All DBs expect Aurora
- Read Replica
  - Async to read replicas
  - Configure EC2 to write to one and read from the other
  - You can have read replicas of read replicas (latency though)
  - Can promote a read replica to become their own standalone database
  - UseCase: Very heavy read workloads
  - Two ways to improve perd: read replicas and Elasticache
  - Available for all DB types
  - Not for DR. Just Scaling. Must have auto backups turned on to work. Up to 5 read replicas
  - Can have Multi-AZ read replicas. Each read replica gets a DNS name
  - Can have read replica in a second region

# RDS Backups, Multi-AZ, Read Replicas - Lab

- Can create Aurora read replicas of an existing database
- Question? Is it better to share an RDS to reduce costs?
- Adding a Standby AZ takes a little while to modify - you can schedule it during a window
- You can check "Multi-AZ" in the Configuration tab
- Reboot allows you to change AZs
- You need backups turned on to create a read replica
- You create read replica from "Actions" rather than "Configure"
- read replica can be in any region
- DB Instance identifier is mandatory
- DBs have a "Role" of "Primary" or "Replica"
- You can "promote" a read replica
  - this stops replication
  - this becomes it's own database
- Summary

  - Two backups: Automated and Database Snapshots
  - Read Replicas:
    - Can be Multi-AZ, increase perf, requires backups, diff regions, any DB type, can be promoted
  - MultiAZ:
    - DR, Reboot to test
  - Encryption is available using KMS and applies to Snapshots, replicas, backups

  # DynamoDB
