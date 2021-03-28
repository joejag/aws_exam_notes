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

## Spot Instances

-
