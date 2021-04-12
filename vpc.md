# VPC Overview

- Virtual Private Cloud lets you logically isolate a section of AWS
- Create public subnets for web servers, create private ones for databases and app servers
- Security: SecurityGroups, NACL to control EC2 in each subnet
- Can create a hardware Virtual Private Network (VPN) to extend your network with the VPC
- Region->VPN/Internet Gateway->Router->Route Table->NACL->Subnet->SecurityGroup->EC2
- VPC has 10.0.0.0/16, public 10.0.1.0/24, private 10.0.2.0/24
- Bastion host is a jump box
- AWS largest subset is 10/16 (not 10/8)
- https://cidr.xyz/
- Subnets largest to smallest: 8/16/24/32 (32 is one ip address)
- 10.0.0.0/16 is the most common used in corportate networks
- Smallset subnet is /28 which gives you 16 ip addresses
- Mini Summary:
  - Launch EC2 into a subnet we choose
  - Assign custom ip ranges to each subet
  - configure route tables between subnets
  - Create internet gateway and attach it to our VPC
  - Better security controls: SGs, Subnet Network Access Control Lists
- Default vs Custom VPC:
  - Default is user friendly
  - All subnets have a route to the Internet
  - Each EC2 has a public and private ip address
- VPC Pairing
  - Connect VPCs via a direct network route using private IP addresses
  - Instances behave as if they are on the same private network
  - Can peer VPCs in other AWS accounts
  - Peering is always a Star configuration. 1 central VPC peers with 4 others.
  - No transitive peering: Cannot hop from one network to another, all needs to be direct
  - Can peer across regions
- Summary:
  - VPC is a logical datacenter in AWS
  - Consists of IGW (or Virtual Private Gateways), Route Tables, NACL, Subnets and Security Groups
  - 1 subnet = 1 AZ. Can have multi-subbets in one AZ
  - SGs are stateful, NACL are stateless
  - no transitive peering, must be direct peering

# Create a custom VPC

- When creating VPC you can use dedicated hardware. You choose the IPv4 CIDR Block
  - does not create subnets, internet gateway
  - does create a route table, default NACL, SecurityGroup
  - We have: Region->No access->Router->RT->NACL->SG in the 10.0.0.0/16 subnet
- Create subnet
  - Choose AZ, give Subnet like 10.0.1.0/24 (251 addresses)
  - auto-assign public ipv4 is turned off by default
  - Some addresses are AWS reserved:
    - x.0: Network address
    - x.1: VPC Host
    - x.2: DNS Server
    - x.3: Reserved for future
    - x.255: Broadcast address - which is disabled
  - Modify to enable public ip addresses, which makes the subnet public
- Create Internet Gateway
  - Unattached at first. Attach it to a VPC
  - Only 1 gateway per VPC. IGWs are highly available
- Configure Route Table
  - default routes allows everything on the network to talk to each other
  - Do not make "main" route table public facing as everything becomes public facing
- Create Route Table
  - Create Route out to the Internet: 0.0.0.0/0, target IGW. Do ipv6 too ::/0
  - Edit Subnet Associations (explict subnet)
  - This moves the subnet here, the main route table gets any non-explictly added ones
- Launch EC2 into the VPC
  - SecurityGroups are gone, they are per VPC
  - By default, the SG does not allow access to each other
  - Create DB SG
    - Allow ICMP IPv4 from 10.0.1.0/24
    - Allow other ports
    - Reassign EC2 to this new SG
- Private subnet cannot contact the Internet to update

# NAT Instances vs NAT Gateways

- Network Address Translation. Allow private subnets to access the internet
- NAT Instances are the old style, uses EC2.
  - Launch EC2, Choose communitu images, type 'nat', launch into public subnet
  - Acts as a bridge from private subnet, onto the public Internet Gateway
  - Must disable "source and destination" checks under "Neworking"
  - Create a Route in Route Tables, use Main route table. Add 0.0.0.0/0 with target of the EC2 Instance
- Problem: Only one box, could get overwhelmed
  - If you terminate the NAT instance the status is "blackhole"
- NAT Gateway
  - Put in the public subnet
  - Add public elastic ip address
  - Create Route in Route Table as before
- Summary
  - NAT Instance
    - NAT Instance not used much as you have extra things to do like disable source/destination check
    - Must be in a public subnet
    - Must be a route out of the private subnet to the NAT instance
    - Bottlenecks based on size of the instance, can vertically scale
    - Can get HA by using Autoscaling groups, multiple subnets in different AZs, and a script to failover
    - Always behind an SG
  - NAT Gateway
    - Redundant inside a AZ.
    - Preferred. Start at 5Gbps, scales to 45Gbps
    - No need to patch
    - Not associated with SG
    - Auto assinged a public ip
    - Still need to update Route Table
    - No need to disabled source/destination checks
    - Best to have a Gateway in each AZ

# Network Access Control Lists vs Security Groups

- NACL are associated with subnets, they have inbound and outbound rules
- Add new rules in increments of 100
- On creation the NACL has no ALLOW routes
- Can move subets to different NACLs
- Must add outband and inbound
  - Outband needs ephermeral ports like 1024-65535 (NAT Gateway uses this)
- Weighting is more important than DENY
- One subset can only be connected to one NACL
- Rule changes are immediate
- NACL are before SG
- Inbound also needs 1024-65535 to make calls to the Internet
- Summary
  - VPC comes with a default NACL, which is wide open by default
  - Can make custom ones, by default they deny everything
  - Each subnet must be associated with a NACL, or it gets the default one
  - Can block IP Addresses with a NCAL. Not with a SG
  - NACL is a one-to-many with Subnets. Subnets are a one-to-one with NACL
  - NACL has a weighted list of rules
  - Separate inbounc and outbound rules
  - Statless. Inbound does not mean outbound

# Custom VPCs and ELBs

- Three types of load balancer: app, network or classic
- Scheme: Internet facing or Internal
- It can detect if you add an Internet Facing into an Internal zone
- At least 2 public subnets are required for an ELB

# VPC FlowLogs

- Captures info about ip traffic going to and from your VPC. Uses CloudWatch or S3
- Can be created at three different levels: VPC, Subnet, Network Interface Level (ENI)
- Console: Choose VPC->Actions->Create Flow Log
  - Can choos to see accept/rejected traffic
  - Go to Cloudwatch to create a log group
  - It suggest an IAM role to allow logging
- Summary:
  - You cannot setup FlowLogs for peered VPCs unless they are in your account
  - Can tag flow logs
  - You cannot change a flow log after it has been created (i.e. change the IAM Role)
  - Not all IP Traffic is monitored:
    - DNS traffic is not
    - Windows license activiation is not logged
    - Traffic to 169.254 for metadata is not logged
    - DHCP traffic
    - Traffic going to the VPC router

# Bastions

- Computer designed to withstand attacks, usually just hosts a proxy server
- Usually in a public zone (DMZ) and is contactable by untrusted computers
- Nat Instance is behind a SG, NAT Gateway is not
- Reduces the surface area which we need to harden (it's a jump box)
- ssh or RDP access. Images also available via AWS Community
- Summary
  - NAT gateway/instance is for giving internet access to private subnet instances
  - Bastion is used to admin instances using SSH (jump box)
  - You cannot use a NAT Gateway as a Bastion Host

# Direct Connect

- Connect on-prem to AWS. Reduce network costs, increase bandwidth than over the Net
- Example:
  - AWS Region (Public and Private) | AWS Backbone Network
  - Direct Connect (DX) Location - Partner Site. Add AWS routers and customer ones
    - X-Connect goes between routers (a cable)
  - Customer Site | Customer Provided Network
- "Last mile" solution
- Summary:
  - Directly connects your datacenter to AWS
  - High throughput workloads
  - Need stable and reliable, secure connection
  - Useful if you have an unreliable connection or need high throughput

# Setting up Direct Connect

- How to create in the AWS Console: Like InternetGateway but called VirtualPrivateGateway instead
  - Create a PUBLIC Virtual Interface
  - VPN Connection -> Create a Customer Gateway
  - Create Virtual Private Gateway
  - Attach VPG to VPC
  - VPN Connections -> Create new VPN Connection
  - Select VPG and CustomerGateway
  - Once VPN is available, set up the VPN on the customer gateway or firewall
- Looks like the VPN requires a CustomerGateway and VirtualPrivateGateway to work

# Global Accelarator

- Sends users to Global endpoints near them. Availability and performance
- Gives you two static ip addresses. Can bring your own ip addresses
- Works by saving your traffic going across multiple hops
- Things involved: ip addresses, DNS Name, Endpoint Group, Endpoint
  - Accelartor: directs traffic to optional endpoints. Has 1 or more listeners
  - DNS Name: Each Accelator gets a domain name, or use your own
  - Network Zone: Kinda like an AZ. Two ips -> Two zones
  - Listener: UDP/TCP ports. Associated with Endpoints by Regions
  - Endpoint Group: Per AWS Region. 1 or more per Region. Can adjust using a traffic dial. Let's you do blue/green deployment
  - Endpoints: LBs, EC2 instances or IP addresses. Can weight endpoints
- Lab making one:
  - Give name (choose ip address), posts tcp/udp sticky, endpoint group (region/traffic dial) health checks, add endpoint (EC2, ALB)
  - Disable then Delete Accelartor
- Summary
  - Improve availability and perf for local and global users
  - Get two static ips or bring your own
  - Traffic dials control where traffic goes

# VPC Endpoints

- Privately connect your VPC to supported AWS services via PowerLink with needing NAT/iGW,VPN or public ips. Does not leave AWS network
- Virtual devices, horizontally scaled, HA no bandwidth risk
- Two types:
  - Interface endpoints
    - ENI with private IP going to an AWS service (gateway, ec2, ELBs, SQS etc)
    - Attach an ENI to an EC2 instance
  - Gateway endpoints
    - NAT based, S3, DynamoDB
- Lab (adding S3 without a NAT gateway on a private network)
  - VPC -> Create Endpoint -> Service Category (AWS services) -> Huge list -> Choose VPC, Choose RouteTable
  - Not immediate
  - Need to specify region on command line to see things (bug?)
- Summary
  - Connects privately to AWS services
  - Also connects privagtely via PowerLink to things
  - No other equipment required
  - Two types: Interface and Gateway

# AWS PrivateLink

- Open your VPC to another VPC
  - Could open to Internet (via iGW) - security concerns, need to manage SGs and NACLS
  - Could VPC Peer - loads of peers, opens the whole network up rather than a host
- Powerlink
  - Powerlink exposes a service to 1000s of other VPCs
  - No VPC Peering, route tables, NAT, iGWs
  - Requires a NLB on the service and an ENI on the customer VPC
- Summary
  - 10s,100s,1000s of VPCs peering? Use PrivateLink
  - Does not require VPC peering of network things
  - Needs a NLB and ENI

# AWS Transit Gateway

- Network can get complicated. Transit Gateway simplifies network topologies
- All connections go via this. VPNs, VPCs, Direct Connect. Hub and spoke model
- Summary
  - Transitive peering between 1000s of VPCs, on-prem
  - hub and spoke model
  - can have across multiple regions
  - can go across accounts using RAM
  - can use RouteTables to limit how VPCs talk to one another
  - Works with DirectConnect and VPNs
  - Supports IP Multicast (only thing that does this on AWS)

# AWS VPN CloudHub

- Hub and spoke VPN via your VPC and other on-prem sites
- Over the Internet but encrypted

# AWS Network Costs

- Traffic going in is free
- Internal to AZ: Via private ip address is free
- Private IP: Internal to another AZ: Charged $0.01 per gig
- Public IP: External to the Internet back into AWS: Twice as expensive. As using AWS Backbone network
- Inter-Region: Same and Public IP transfer
- Summary
  - Use private over public ip addresses to save costs. This uses AWS backbone network
  - Save the most by putting all EC2 in one AZ and use private IPs. Cheap, but single point of failure

# VPC Summary

- TODO: You must be able to build a VPC from memory
- VPC
  - VPCs are not peered transitive (through another) has to be 1-1 basis. Can peer between regions
  - Create a VPC gives you a RouteTable, NACL, SG. DOES NOT do subnets or iGW
  - AZs are randomised between accounts
  - AWS reservers 5 address in your subnet
  - Only one iGW per VPC
  - SG does not span VPCs
- NAT Instances
  - Disable Source/Destination Check. Public subnet, route from private subnet to the NAT instance for it to work
  - Instance Size bottlenecks the throughput. Can do autoscaling groups, behind an SG
- NAT Gateway
  - MAnaged version. Not behind an SG.
  - REdundant, no patching, gets public ip, you need to update the ROuteTables to get a route out
  - Need multiple to cover AZ going down
- NACLS
  - VPC gets a default one that allows everything
  - Custom NACLS deny everything
  - Each subnet must be associated with a NACL or it gets the default one
  - Can block ip addys, cannot do this with SG
  - Use a weighting rules with lowest number first. DENY rules must come first to trump ALLOW rules
- ELBs
  - Need two public subnets to use an internet facing load balancer
- VPC Flow Logs
  - Cannot turn on for other accounts which are peered
  - Can tag them
  - Config is set at creation - change IAM role for instance
  - Not all IP traffic is monitored (DNS, license activation MS, 169.254, DHCP, router)
- Bastion
  - Allows ssh/rdp in to EC2, NAT Gateway/Instance allow traffic out
- Direct Connect
  - datacentre to AWS. Good for high throughput network work loads
  - Stable and secure use cases
  - TODO: Remember the steps to create a DC connection
    - Create a virtual interface in DC Console (public virtual interface)
    - VPC->VPN connections create a new Customer Gateway
    - Create a Virtual Private Gateway
    - Attach VPG to VPC
    - Select VPN Connections and create new VPN Connection
    - Select VPG and the Customer Gateway
    - Once available, setup VPN on Customer Gateway or firewall
- Global Accelerator
  - Perf and availablity. 2 static ips, traffic dials, weighting
- VPC Endpoints
  - Connect VPCs privately without NAT stuff. Does not require public ip addresses
  - Two types: Interface (ENI) and Gateway (S3/Dynamo)
- AWS PrivateLink
  - Connecting to loads of Customer VPCs via a Network Load Balancer to an ENI on the customer
- Transit Gateway
  - Simplify network, hub and spoke, gives transitive peering, aws & onprem
  - regional, but can be cross-region
  - multi AWS accounts via RAM
  - RouteTables limit how the VPCs talk to each other
  - Works with Direct Connect and VPN connections
  - Supports Multicast - unique
- VPN Cloudhub
  - Simplify VPN connections. On-prem and AWS talking across VPNs. Hub and spoke
- AWS Network Costs
  - private > public for cost = use AWS backbone
  - groups EC2 in AZ and use private for cheapness
