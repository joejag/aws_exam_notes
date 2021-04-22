# Serverless

## Lambda

- Progression: Data Centres, IAAS (EC2), PAAS (BeanStalk), Containers (EKS), Serverless (Lambda)
- don't worry about patching, scaling
- Can use as event driven (S3 or Dynamo). Compute service using Gateway
- Traditional arch vs Serverless arch
- Priced on requests and duration
- Summary
  - Lambda Scales out (not up)
  - 1 event = 1 function (I doubt it)
  - AWS X-RAY helps you debug what's going on

## Building a service

- AWS Serverless Application Repository is pre-made apps
- Can attach to a VPC
- Polly is a text-to-speach service that stores mp3s onto S3

## Serverless Application Model (SAM)

- CloudFormation extension optimised for servless apps
- Adds: Functions, APIs, Tables
- Run Serverless locally using Docker
- Package and deploy using CodeDeploy
- Many things can trigger: Alexa, Cognito, Gateway, CloudWatch, EventBridge, DynamoDB, S3, SQS, SNS, Kinesis, IoT
- sam does local and deloyment

## Elastic Container Services (ECS)

- Container Orchestration into clusters optimially placed
- ECS run on EC2 or Fargate
- Can set CPU, mem and monitor
- ECS components
  - Cluster
    - Logical collection of EC2 or Fargate instances
  - Task Definition
    - Defines app. similar to Dockerfile of Dockerfiles
  - Container Definition
    - cpu, mem, ports, which container to use
  - Task
    - A single running copy of a container
  - Service
    - min/max for Task Definitions
  - Registry
    - Storage for container images (ECR or DockerHub)
- Fargate
  - Serverless container engine. Works with ECS and EKS
  - Runs in own kernel. Only pay for the containers not the supporting stuff
  - Choose EC2 instead if:
    - strict compliance rules
    - apps require customisation
    - require GPUs
- Elastic Kubernetes Service (EKS)
  - ECS lets you use the same toolset onprem and cloud
  - Containers are grouped in pods
  - Why use?
    - Already on K8s. Want to migrate to AWS
- Elastic Container Registry (ECR)
  - Managed Docker container repo. Works with ECS and EKS and on-prem. HA and IAM
- ECS + ELB
  - Supports all LBs
  - ALB is recommended
- ECS Security
  - Instances Roles
    - Policy applies to all tasks in EC2 instance
  - Task Roles
    - Policy applies to tasks alone
- Demo
  - Structure is Cluster->Service->Task Definition->Container Definition
  - IF you stop a task, the service will bring a new one online

## Serverless Summary / quiz

- If cost is a concern, go for serverless arch over traditional
- S3 does async calls to Lambda. Kinesis, Gateway and Lex do sync
