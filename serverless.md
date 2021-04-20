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
-
