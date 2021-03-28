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
  - reservation givres 70% off
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
  - Securituy Groups are stateful (inbound is auto allowed for outbound)
  - Cannot block specific ip addresses with SG. You must use NACL instead
  - You can set Allow rules, nut not Deny rules

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
- When you terminate an EC2 instance is remove the EBS volume on termination
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
  - create snaphsot. Copy the snap, Create image, Launch instance
  - You cannot use an ecrypted image and try to have it unencrypted in the launcher
- Summary:
  - Snaps of encVolumes are enc automatically
  - Volumes created from encSnaps are env automatically
  - You can only share unencSnaps with other accouts or made public
  - You can enc the root volume when making new instances

## Spot Instances and Spot Fleets

- Use unused EC2 capacotu on the cloud. 90% cost reduction.
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
- OnDemand and Reserved instances only

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
- `yum install amazon-efs-tools -y` allows you to do some more things
- Need to open up Security Group inbound to allow access to EFS. The "source" is your other security group
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

-
