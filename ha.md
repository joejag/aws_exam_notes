# High Availability

## ELBs

- Three types in AWS
  - Application Load Balancers
    - Layer 7. http(s). Can send specific requests to specific web servers
  - Network Load Balancers
    - Layer 4. tcp. Also extreme perf
    - Fixed ip address
  - Classic Load Balancer
    - Layer 4 or 7. X-Forward, sticky sessions
    - Cheaper, basic round robin
- 504 (gateway timeout) if your app is not responding within timeout
- X-Forwarded-For sends ip address of caller (ipv4)

## Classic Load Balancer

- Classic
  - Needs VPC. Can be "internal" which means private subnets. Can select subnets (adv VPC)
  - Needs port for listener.
  - Needs SecurityGroup
  - Needs HealthCheck
  - Add EC2 instances. Cross-zone means if the ELB should only target inside it's AZ or go across AZ
  - You never get an ip address for an ELB, only a DNS
- Application
  - Create Target Group first
    - Target Type: Instances, ip, lambda
    - After creation. Add targets via a tab. Add Instances
  - Create Application Load Balacner
    - Scheme: Internal vs internet
    - Choose AZs
    - Add SG
    - Choose Target Group
    - Create
    - Goto TargetGroup, choose targets
  - Gives you if/then on headers/path you can forward/returnFixedResponse/redirect
- Summary:
  - InService/OutOfService from ELB pov
  - READ FAQ FOR CLASSIC LOAD BALANCERS

## Advanced Load Balancer theory

- Sticky: session goes to the same host. Classic and ALB (though to TargetGroup)
- CrossZone goes cross AZ, Good if unbalanced EC2s in each AZ
- Path Patterns: Send path segments to different microservices. ALB feature

## Auto Scaling

- Three components
  - Groups (logical: web, databse, application)
  - Config Templates (for EC2, AMI, iType, SGs)
  - Scaling Options (dynamic or on schedule)
    - Maintain current level
      - min 10 at all times (kills unhealthy and relaunches)
    - Scale manually
      - set max, min and desired - it goes to desired amount
    - Scale on schedule
      - date and time, set amount
    - Scale on demand
      - most popular. use scaling policies (set CPU to 50%)
    - Use predictive scaling
      - magic history stuff

## Launch Configurations

- EC2 -> Auto Scaling -> Create Launch Config
  - Similar to making EC2 instances (AMI, iType etc)
  - Offers spot instances and CloudWatch monitoring
  - Once complete doesn't do anything yet
- Create AutoScaling Group
  - Sets group size. Choose subnets
  - Can create a Load balancer
  - Can change from maintain scaling policy
    - Metrics: request count, cpu, network in/out
  - Can get SNS notification on changes
- ActivityHistory shows you when things are recreated
- Recreated instances go into the same AZ, to keep things even
- Deleting an AutoScalingGroup auto deletes the instances associated with it

## Fault Tolerant WordPress site

- Create web distribution from media S3 bucket
- Create image of EC2, then use AMI for scaling group
- RDS reboot to move AZs
- CloudFormation has example stacks called QuickStart, including WordPress

## Elastic Beanstalk

- Set of envs like Java, Docker. Can upload an app or use a SampleApp
- S3 backed code.
- The idea is to not learn AWS. Upload code and it'll inspect it and figure out how to host it
- Summary:
  - BeanStack manages capactiy, LBs, scaling, health monitoring

## Basion Hosts

- Make two in different zones, add a LB in front (network, static ip). AutoScale group to keep it up
- Alt, just have auto scaling group, add a user script to assume an elastic ip on recreation (downtime, cheap)

## On prem strat with AWS

- Things you can use on-prem for HA
  - DMS (database migration service)
    - on-prem primary, AWS as a backup
  - Server Migration Service (SMS)
    - incremental replication of on-prem services onto AWS
  - AWS Application Discovery Service
    - gathers data about on-prem data centres. VMWare plugin. Estimate ToC on AWS and plan.
    - Track in AWS Migration Hub during move
  - VM Import/Export
    - Export AWS VMs to or from data centers
  - Download Amazon Linux 2 as an ISO and mount it locally
    - Can use in VMWare!
- Summary:
  - DMS, SES, AWS App Discovery, VM import/xport, ISO of Linux

## HA Summary

- ALBs (7), NLB (4), CLB (Cheap). 504 on timeout
- InService or OutOfService from health checks.
- ip addresses only on NLB. X-Forward-For on CLB for ipv4
- ALB (sticky). Cross-Zone LB (AZs), Path Patterns (go to diff EC2)
- QuickStart is CF templates built by AWS
- Elastic Beanstalk is for folks who don't care about AWS. CloudFormation really
- High level on-prem: DMS, SES, AWS App Discoryt, VM im/export, ISO of Linux

## Quiz

- S3 Standard is more cost effective than S3 Intelligent-Tiering if the data is updated frequently
