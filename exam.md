# A Cloud Guru: Exam 1

- S3 used to have account name as a path, it is now "virtual" by being a sub-domain
- Enc at rest in: EBS, EFS, S3
- spread placement is max 7 per AZ
- You can go to m4.xlarge to get more network bandwidth for NATs
- "resource owner" is the AWS Account that creates buckets & objects
- CloudWatch cannot see EC2 memory as it cannot get onto the box
- SQS visibility:12h, retention:14d
- Support: Dev (nothing), Biz (1h prod), Enterprise (15m biz critical + TAM)
- EC2 is dedicated or shared. It cannot change after creation
- SQS Long Polling is set in Seconds
- Storage Gateway: on prem for NFS or partial backup
- DynamoDB max size is 400kb name+value
- subnets: biggest:16, smallest: 28

# Digital Cloud Training: Exam 1

- Cloudfront can choose regions by "price class"
- given two valid options go for the cheaper one
- You can autoscale DynamoDB throughput
- Global Accelerator -> NLB -> Instances
- Block a country? use Cloudfront (or maybe a WAF) - not NACL
- You can makea a 64k iops database with io1/2
- Endpoints: in-VPC? use Gateway (s3/dynamo), otherwise use Interface
- You can do "scheduled reserved instances"
- AWS Inspector cannot apply changes, just suggest
- RDS: Read replicas cannot be enc if the host is not enc
- RDS: For enc in transit, you can download an AWS root cert to use
- Kinesis: If it says "near real-time" use Kinesis
- SCP are a DENY all with an allow
- ALB must be in a public subnet, but can target private subnet stuff
- EBS: Cold/throughput cannot go to 3k iOPs, use SSD
- ECS: To add roles use taskRoleArn. Instances can have multiple tasks

# Compute notes

- Lambda: Use KMS to encrypt passwords as ciphertext
- AutoScale: Can be suspended. Can put instances into a Standby state
- EC2: If terminated it could be EBS snapshot corrupt or EBS volume limit reached

# Storage notes

- S3: Glue is ETL, can be kicked off by Lambda
- Glacier: Expediated is minutes, Standard is 3-5 hours, bulk is 5-12
- EBS: SSD goes to 16k iOps

# Database notes

- DynamoDB: Streams allow you to kick off Lambdas
- ElastiCache: Redis persistence is good for high perf / persistence
- RDS: encryption is by the key in that region, so read replicas use a different one
- DynamoDB: You get a 400 if you use all your writers up (ProvisionedThroughputExceededException)
- RDS: To connect on prem: update SG and make instance available in a public subnet
- DynamoDB: There is an S3 pointer type: S3 Object ID

# Migrations and Transfer

- DirectConnect: Lead time is one month

# Networking & Content Delivery

-
