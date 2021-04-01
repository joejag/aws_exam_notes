# Elastic Compute Cloud (EC2)

## Intro

- resizable compute capacity, scale in minutes up and down as requirements change
- Pricing models:
  - on demand: pay per hour/second, no commitment
  - reserved: capacity reservation, much cheaper (1-3 years)
  - spot: excess capacity. Dropped prices - movable. Can be terminated.
  - dedicated: physical servers, for server bound licenses
- on demand:
  - low cost, flexible without up front or long term commitments
  - short term, spiky, unpredictable workloads that cannot be interuppted
  - spikes
- reserved:
  - steady state, predictable usage
  - can make upfront payments to reduce their overall cost
  - types:
    - standard: 75% off, greater discount if you buy for longer
    - convertable: Allows you to change CPU, RAM.
    - scheduled: time windows. fraction of week/day/month
- spot:
  - flexible start and end times
  - can be terminated
  - only feasible for very low compute prices. Urgent computing needs
- dedicated:
  - regulatory: no multi-tenant
  - licensing that doesn't support mult-tenent or cloud
  - can be purchased on demand (hourly)
  - reservation gives 70% off
  - No real difference between Dedicated and Dedicated Host apart from visibility to the server. Slight perf gains from Dedicated Host for multiple apps.
- Instance types:
  - F1: Field Prgrammable I3: High Speed Storage, G3: Graphics Intensive, T3: Low cost/general, D2: Dense Storage, R5: Memory Optimised, M5: General, C5: Compute, P3: Graphics/General, X1: Memory, Z1D: High compute and memory, A1: ARM, U-6tb1: Bare metal
  - F: FPGA, I: IOPS, G: Graphics, H: High disk, T: Cheap, D: Density, R: RAM, M: Main choice, C: Compute, P: Gfx/Pics, X: Extreme Memory, Z: Extreme memory & compute, A: ARM, U: Bare Metal
- Summary:
  - EC2: VMs in the cloud
  - Pricing: On demand, reservered, spot (not charged for remainder of time is killed by AWS), dedicated

## EC - Launching instances

- AMI: Choose machine type (Ubuntu etc)
- Instance Type: Can choose Graphics optimised etc
- Configure instance:
  - AZs have randomised number at the end per account
  - Change 'stop' behaviour to hibernate or terminate the whole thing
  - "Enable CloudWatch detailed monitoring" is to change the time frame from every 5m to every 1m
  - "User Data" allows you to add a bootstrap script
- Storage:
  - root device has less options for volume type
- Tags exist
- Security Groups:
  - A virtual firewall
  - Source: ip range to open up to
- Launch image
  - Need key pair
- You can use "EC2 Instance Connect" to launch an ssh session in browser
- `chkconfig on` sets httpd to auto restart on reboot
- EC2 Console
  - Status Check: System (hypervisor), Instance (ec2 instance)
- You can encrypt your root hard disk on creation
- Additional voumnes aren't auto-marked for delete on temination
- Summary:
  - Termination Protection is off by default
  - On Terminated, the root EBS volume is deleted by default
  - Root EBS can be encrypted on creation. Can used 3rd party like BitLocker on Windows. Can do when creating AMIs
  - Additional volumnes can be encrypted

## Security Groups

- IPv6 rule looks like: `::/0`, v4 is `0.0.0.0/0`
- Rule changes take place immediately
- security groups are **stateful**, if you allow something in, it is implicityly allowed out (NACLs are different to this)
- You can only allow with Security Groups, you cannot Deny anything
- You can attach more than one Security Groups to an EC2 instance. Use via "Change Security Groups" in EC2 Console
- Summary:
  - All incoming traffic is blocked by default
  - All outbound traffic is allowed
  - Changes are immediate
  - Security Groups can be shared between EC2 instances.
  - An EC2 instance can have multiple Security Groups attached
  - Security Groups are stateful (inbound is auto allowed for outbound)
  - Cannot block specific ip addresses with SG. You must use NACL instead
  - You can set Allow rules, but not Deny rules

## EBS Volumes: 101

- Elastic Block Store: virtual hard disk, persistent block storage. Automatic replication within the AZ
- 5 types:
  - gp2: General purpose (SSD) - most things
  - io1: Provisioned IOPS (SSD): Fast input output per second (4x faster) - databases
  - st1: Throughput Optimised Hard Disk Drive (magnetic) - Data warehouse
  - sc1: Cold Hard Disk Drive (magnetic) - file servers
  - Standard: EBS Magnetic - IA

## EBS Volumes and Snapshots

- EBS volume will be made in the same AZ as the EC2 instance. Otherwise you'd get huge lag
- When you terminate an EC2 instance it removes the EBS volume on termination
- You can modify disk allocations, but you need to perform a cmd line to use it
- The root partition has a snapshot as it comes from the AMI - it can also be resized on the fly
- you cannot decrease volume size
- Moving to another AZ
  - Create Snapshot of root EBS (can take a minute)
  - From snapshot -> Create Image
  - Choose Virtualisation Type: Hardware or Paravirtual (some OSes only support one). HV has more instance types.
  - Now available under "AMIs" -> Launch
  - Change Subnet to get a different AZ
- Moving to another Region
  - AMIs -> Copy AMI
- When you terminate, the non-root EBS volumes persist and are "available"
- You 'deregister' AMIs
- Summary:
  - EBS is a virtual hard disk
  - Snapshots exist on S3 - they are point in time copies of Volumes
  - Snapshots are incremental - first snapshot might take a while
  - For snapshots of root devices, it's best to stop the instance first - but can while running
  - Can create AMIs from snapshots
  - You can change EBS volume sizes on the fly including storage type
  - Volumes are always in the same AZ at the EC2 instance
  - Can migrate AZs via snapshot->AMI->Launch. Region too, by copying the AMI

## AMI types (EBS vs Instance Store)

- Select AMI by:
  - Region
  - OS
  - Arch (32, 64)
  - Launch Perms
  - Storage for Root device
    - Instance Store (ephermeral)
    - EBS Backed Volumes
- AMIs are categorised by backed by Intance or EBS
  - EBS: Root device is from a snapshot
  - Instance Store: Created from an instance store created from a template on S3
- Launch by community AMI: OS choices
  - Can cannot attach more Instance Store Volumes after creation
  - Only 1 Volume listed when two instance running
  - Cannot 'stop' an InstanceStore - only terminate. Stop is for writing to EBS
- Summary:
  - InstanceStore Volumes are Ephermeral Storage
  - Cannot be stopped, if the host fails you lose your data
  - EBS will not lose data if stopped
  - You can reboot both - it's just host failure that cannot be recovered from
  - By default, both delete root volumes, but with EBS you can opt to not delete on terminate

# ENI vs ENA vs EFA

- ENI: Elastic Network Interface - virtual network card
- EN: Enhanced Networking - Use a SingleRoot IOVirtualisaiton (SR-IOV) - high perf (on some instances)
  - EN skips the hypervisor emulation to get nearly non-virtualised speeds using a PF (Physical Function)
- EFA: Elastic Fabric Adatper - Network device for High Performance Computing & ML
- ENI:
  - EC2 comes with by default. Allows IPv4, plus secondary private IPv4.
  - One Elastic IP Address per private IPv4
  - IPv6, Security Group. Mac address. Source/Destination check flag
  - Scenarios:
    - Creating a Management Network separate from your production network
    - Use network and security appliances in your VPC
    - Create dual-homed instances (separate db and prod networks)
    - low budget, HA
- EN: Enhanced Networking:
  - Sometimes ENI can't handle the workload
  - Higher I/O, Lower CPU utilisation to ENI
  - Higher bandwidth, higher packet per second.
  - No extra charge - but instance needs to support it
  - Two ways of getting EN:
    - ENA: Elastic Network Adapter: 100Gbps - use this!
    - Intel Virtual Function (VF): 10Gbps - older!
- EFA: Elastic Fabric Adapter:
  - High Performance Computing & ML
  - Lower latency, high throughput
  - OS-bypass: Can bypass the linux kernel and talk directly to the EFA device (linux only)
- Summary:
  - ENI: Basic. Separating networks
  - Enchaned Network: Big speeds of 100Gbps
  - Elastic Fibre Adapter: HPC & ML

## Encryted Root Volumes & Snapshots

- To encrypt an unencrypted:
  - create snaphsot. Copy the snap (with enc), Create image, Launch instance
  - You cannot use an ecrypted image and try to have it unencrypted in the launcher
- Summary:
  - Snaps of encVolumes are enc automatically
  - Volumes created from encSnaps are env automatically
  - You can only share unencSnaps with other accouts or made public
  - You can enc the root volume when making new instances

## Spot Instances and Spot Fleets

- Use unused EC2 capacity on the cloud. 90% cost reduction.
- Use cases: stateless workloads, test and dev instances. Anywhere with flexible apps
- Anywhere you can take a 1-2 minute notification of termination. Not for critical apps
- Spot Price: Use set a maximum spot price. If Spot price is below max, then you get it
  - varies hourly depending on capacity and region
  - if the price goes above you have 2 minutes to decide to stop or terminate your instance
- Spot Blocks prevent termination: 1-6 hours extra time
- Check Spot instance pricing history to figure out an AZ and price to use
- Spots are useful for:
  - Big data & Analytics, Containerised workloads, CD/CD and testing, Web services, image and media rendering, HPC
- Spot instances are bad for:
  - Persistent workloads, Critical Jobs, Databases
- Terminating spot instances:
  - First you make a request with: max price, number, launch spec, type: one-time|persistent, from-to
  - persistent: ec2 is stopped and restarted when prices falls again. one-time is terminated
- Spot Fleets:
  - Spot instances and on-demand instances. If not enough spot instances to match number then it launches on-demand
  - Setup different launch pools (instance type, AZ, OS), multiple pools available, stops when you meet price or capacity desire
- Strategies:
  - capacityOptimised: from the pool with the optimal amount
  - diverisifed: spot instances are distributed across pools
  - lowestPrice; pool with the lowest price (DEFAULT)
  - InstancePoolsToUseCount: used with lowestPrice to distrubute
- Summary:
  - Save 90% compared to OnDemand
  - Useful when you don't need persistent storage
  - You can block terminatation using Spot Block
  - Fleet is a combo of Spot and OnDemand

## EC2 Hibernate

- Needs to be set when you create the instance
- terminate: deletes things, stop: keeps them on an EBS
- When you start an EC2 instance: OS boots up, bootstrap script runs, apps start
- Hibernate: Store RAM on disk, plus normal EBS stop behaviour
- Difference from stopping: You retain your instanceId for the EC2 instance
- Useful for:
  - Long running processes. Services that take a long time to start
- to use: in launch config you have to choose "enable hibernation as an additional stop behaviour"
- MUST be encrypted to use this feature
- Summary:
  - Hibernate preserves RAM in EBS
  - Faster to boot up - no OS to load again
  - Instance RAM must be less than 150Gb
  - Instance families include: C3/4/5, M3/4/5, R3/4/5 (compute, general, ram)
  - Available on Win, Azn, Ubuntu
  - Limit of 60 days for hibernation
  - Not available on Spot instances

# Cloudwatch

- Monitoring service for AWS resources and applications
- Monitors:
  - compute: EC2, Autoscaling, ELBs, Route53
  - storage & content delivery: EBS, Storage Gateways, Cloudfront
- CloudWatch with EC2:
  - metrics: cpu, network, disk, status check (hypervisor, instance)
- Similar to CloudTrail which monitors AWS console and API calls (ip address, time, user)
- Clouwatch: performance. CloudTrail: audit of calls to AWS

## Cloudwatch - Lab

- CPU & Network are monitored. RAM usage and disk usage aren't in by default
- Dashboards can be region or global
- Events are for when resources change
- Summary:
  - Standard Monitoring is 5m
  - Detailed monitoring is 1m
  - can create dashboards, alarms, events (AWS resources changed), logs (aggreate, monitor)
  - cloudtrail is api within platform, cloudwatch is about monitoring perf

# AWS Command Line

- `aws` command line is preinstalled on EC2 instances, but you will still need to configure it
- not on the exam

# IAM Roles

- Use AWS platform without access keys. Can put IAM role onto EC2 instances to do so
- Reduces breadth of attack if attacker can only use this machine to do things
- Summary:
  - Roles are more secure than access keys on EC2 instances
  - Roles are easier to manage than keys (think rotation or loss)
  - Roles can be applied after creation via console of CLI
  - Roles are universal (it's IAM)

# Bootstrap scripts

- Runs as root when provisoning EC2 instance

# Instance Metadata

- See metadata inside EC2: `curl http://169.254.169.254/latest/user-data`
- get ip address: `curl http://169.254.169.254/latest/meta-data/public-ipv4`
- You could punt the ip address into S3 then have a lambda kick off to process it
- Summary:
  - Metadata is used to get info about an instance

# EFS: Elastic File System

- Think NFS, but the filesize auto increase and decreases (so can be used on multiple machines)
- Has LifeCycle policies like S3. Can move to EFS IA which is cheaper
- `yum install amazon-efs-tools -y` allows you to mount EFS devices
- Need to open up Security Group inbound on EFS to allow access from EC2. The "source" is your other security group
- EC2 SG needs to allow Outbound to EFS
- "mount target state" needs to be "available" before you can use it - which can take a few minutes
- The attach menu in EFS gives you `sudo mount -t efs -o tls fs-eb2cdfdf:/ efs` to run - if it doesn't work check the SGs
- Summary:
  - EFS supports NFSv4
  - Only pay for what you use, no pre-provisioning required like EBS
  - Scales to Petabytes, can support 1000s of connections
  - Data is stored across multiple AZs
  - Read after Write consistency

# Amazon FSx for Windows and Lustre

- managed fully native Windows filesystem. FSx is built on Windows server
- For use with Sharepoint, IIS etc
- Samba (well Server Message Block (SMB)) instead of NFS. Supports AD users, security policies and DFS namespaces and replication
- EFS does not support Windows. So use FSx there
- FSx for Lustre: Compute intensive, High Performance. Insane speeds
- Summary:
  - EFS: Distributed, resilent for Linux
  - FSx for Windows: Centralised storage for Windows apps
  - FSx for Lustre: High speed, HPC. Can store onto S3

# EC2 Placement Groups

- Three types:
  - Clustered Placement Group
    - Instances within an AZ - High throughput, low latency - only certain instances can use this
  - Spread Placement Group
    - As many different AZs as possible. Small number of critical instances that should be far apart
  - Partitioned
    - Logical segments, Like spread, but some instanes are in the same partition
    - For Hadoop, HBase, Cassandra
  - Summary
    - Clustered cannot span multi AZ
    - Spread and Partitioned can go multi AZ
    - Placement group names must be unique within you account
    - Only certain instances can use groups (general, compute, memory, storage)
    - AWS recommends you use the same instance types in a clustered group
    - You can't merge placement groups
    - You can move a stopped instance into a placement group via the CLI

# HPC on AWS

- Requires: data trasnfer, compute&networking, storage, orchestration&automation
- Data transfer:
  - Snowball, Snowmobile (T/P worth of data)
  - AWS Datasync to store on S3, EFS, FSx
  - Direct Connect
- Compute and Networking services:
  - EC2 with GPU or CPU
  - EC2 fleets (spot instances or spot fleets)
  - Placement Groups (cluster)
  - Enchaned Networking (no extra cost!)
  - Elastic Network Adapters (older version is VF)
  - Elastic Fibre Adapters (OS-Bypass, lower consistent latency - only Linux)
- Storage services:
  - Instance Attached:
    - EBS (64k IOPS with provisioned IOPS)
    - Instance Store (ephermeral, millions of IOPS)
  - Network Storage
    - S3
    - EfS (IOPS scales to the tatal size, or use Proisioned IOPS)
    - FSx for Lustre: Millions of IOPS, backed by S3
- Orchestration and Automation
  - AWS Batch:
    - Run batch jobs. parallel single job that runs on multi EC2 instances, schedule jobs
  - AWS ParralelCluster:
    - OpenSource for doing the same thing. Simple text file to model your needs, automates creeation of resources

# AWS WAF: Web Application Firewall

- HTTP or HTTPS protection for CloudFront, ALB or API Gateway.
- Layer 7, so it can see query strings, bodies
- Configure it by IP Addresses, by query strings. Allow or give back HTTP 403 response
- Three different behaviours:
  - Allow all, expect ones you specify
  - Block all, expect ones you specify
  - Count the requests that match the proeprties you specify (passive mode)
- Protects using conditions you specify. You can use:
  - ip range, country, request headers, regex, length of requets, SQL injection, XSS
- Summary:
  - Use WAF to block SQL injection, XSS
  - use Network ACLS to block ip addresses

# EC2 Summary

- EC2: Pricing models: on demand, reserved, spot, dedicated
- spot termination: you pay per hour, unless AWS terminate it
- EBS: Hard disk termination is off by default. Additioanl volumes are not deleted
- EBS: Can be encyrpted on creation, can use 3rd party tools. Additional volumes can be encrypted
- Security Groups: Deny Inbound by default. Allow Outbound by default. Changes are instant. EC2 is a many-to-many with SG
- Security Groups: Stateful (in implies out allowed.). Cannot block ip addresses (Network ACL instead)
- EBS types: SSD: gp2, io1 (dbs). HDD: st1 (warehouses), sc1 (file servers), Standard (IA stuff)
- EBS: Snapshots go to S3. Point in time copies, incremental copying. First snapshot takes a while.
- EBS: Stop the instance before making the root snapshot (optional). Create AMIs from Snapshosts.
- EBS: Can change volume size and storage type. Always same AZ as EC2 instance
- Migrating EBS: In region: Take snapshot, Create image, Launch Instance
- Migrating EBS: Between region: Take snapshot, Create image, Copy Image, Launch Instance
- Snaphosts: If encrypted, will be encrupted. Can share unencrypted snapshots. Can share with other accounts
- Enc: Root devices can now be enc on provisioning. To enc: Create Snap, Copy Snap with Enc, Create Image, Launch Instance
- Instance Store: Epehermeral. Cannot be stopped. EBS saves data. Reboot both without losing. EBS can be saved on termination
- Enhanced Networking: ENI (basic), EN/A (high speed), EFA (bypass)
- Cloudwatch: Used for performance monitoring. Monitors AWS and your apps. Every 5m by default (1m with detailed). Create Alarms
- Cloudwatch: Dashboards. Threshold Alarms. Events from AWS resource changes. Logs.
- CLI: IAM access required.
- Roles: Better than using keys on Ec2 instances. Can be assigned after creation. Universal.
- Bootstrap scripts: Run on first boot
- Instance Meta Data and User data: Http call to get public ips 169.254.169.254
- EFS: Uses NFS. pay per storage used. Scales to Petabytes. 1000s connections. Stored in multi AZ in region. Read after Write.
- EFS v FSx v FSx 4 Lustre: EFS: Linux. FSx: Windows apps (SMB), FSx Lustre: HPC, direct on S3
- Placement Groups: Clustered (same rack), Spread (no where near), Partitioned (Spread with some together)
- Placement Groups: Only certain types of instance work there. Name needs to be unique in account. Homogenus instances rec by AWs
- WAF: Layer 7 (sql, xss, block countries, block query params). But also NACL available.

# EC2 - Resource Groups and Tags

- AWS Config can show you a count of your resources
- You can right click on the EC2 Console and Create an Image directly from a running instance
- AWS Resource Groups lets you search by resource type
  - TagEditor: You can select a subset of resources and edit their tags (apply a tag to multi resources)
  - CreateResourceGroup: Can select by Tag value
  - SavedResourceGroups: Anything new with the Tag will appear here
- AWSConfig: Can create rules for approved AMIs.
  - Only fires on change. You can reboot EC2 instances to get the Rule to fire

# EC2 - Working with EC2

- VPC is a software networking device. No physical hardware, all virtual
- If the default VPC is missing, under Actions you can create a new VPC
- Default VPC is set up for you: InternetGateway, SecurityGroups, NACL, SubNets
- When creating a new EC2 instance, Tags are applied to ENI, EBS, EC2
- You can set an EC2 up with no ssh key, then Connect and add you public key
- Restarting an EC2 keeps the same private IP but gets a different public IP
- You can stop an instance and change it's instance type for vertical scaling

# EC2 - Roles and Instance profiles

- InstanceProfiles are for applying roles to EC2 instances
- Roles exist as it would be burdensome to apply them to each box, including Spot instances
- High level workflow
  - Create an IAM Role
  - Define which Accounts or Services can assume the Role
  - Define what API actions and Resources the app can use after assuming the Role
  - Specify the Role when launching the instance or attach the Role to a running instance
  - Have the app retrieve a set of temporary creds and use them
- Policy has "Effect" like Allow, "Action" like Put, "Resource" which is an ARN
- Policy optionally can had a "Statement ID": Sid
- AWS STS: Security Token Service
- Steps:
  - Create a Role
  - Create a Permission so EC2 can assume that role
  - Create Policy allowing that role to access an S3 bucket
  - Associate the Policy with the Role
  - Create a Development InstanceProfile
  - Assoiate InstanceProfile with the Role
  - Attached InstanceProfile to EC2 Instance
- EC2 Intance can only have one role

# Quiz wrong answers / extra info

- Underlying Hypervisor on EC2 is Nitro or Xen
- http://169.254.169.254 is a local-link-address and can be disabled
- EBS Multi Attach is for connecting an EBS to 16 EC2 instances in the same AZ
