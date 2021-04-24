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

# A Cloud Guru: Exam 2

- Spot: On termination: 120s, might miss the term notice, can get msg despite SpotBlocks in place (will kill)
- DynamoDB: Already multi-AZ
- VPC: Can set shared or dedicated hardware for EC2 at this level. This can be changed.
- SQS: WaitTimeSeconds controls long polling

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
- AutoScale: To terminate, it chooses the the AZ with the most
- EC2: OnDemand is quicker to spin up then Spot
- Dedicated: Only choose 'host' if licensing comes into it
- AutoScale: If you add existing EC2 instancds to a ASG target group and it exceeds the max for hte ASG then the request will fail
- Metadata: There is a 'Instance Metadata Query' that uses the 169.254 address to get similar info
- ECS: There is a 'Amazon ECS optimized AMI' which installs tools. When not working: Check the IAM instance profile has the right perms, check the agent is running.
- LaunchConfig: Are immutable
- RunCommand: Run a command across Windows EC2 instances
- ASG Health: Uses EC2 health check by default (can use ELB health checks instead), terminates if unhealthy, waits for inflight connections to finish (connection draining), health check for new instances has a 300 second grace period
- AWS Batch: Replace 3rd party stuff for analytics and spinning up EC2. Can do HPC
- Credentials: Store them in 'Systems Manager Parameter Store'

# Storage notes

- S3: Glue is ETL, can be kicked off by Lambda
- Glacier: Expediated is minutes, Standard is 3-5 hours, bulk is 5-12
- EBS: SSD goes to 16k iOps
- EBS Backups: use 'EBS Data Lifecycle Manager'
- S3 home dir: Create IAM policy with folder perms, add users to that group
- EFS: Security via SG and POSIX perms

# Database notes

- DynamoDB: Streams allow you to kick off Lambdas
- ElastiCache: Redis persistence is good for high perf / persistence
- RDS: encryption is by the key in that region, so read replicas use a different one
- DynamoDB: You get a 400 if you use all your writers up (ProvisionedThroughputExceededException)
- RDS: To connect on prem: update SG and make instance available in a public subnet
- DynamoDB: There is an S3 pointer type: S3 Object ID
- S3: To analyse in place you can use S3 Select or Redshift Spectrum
- RDS: Restore works withing 5m and uses the default SG rather than a custom one
- ElastiCache: Redis cannot have an IAM role, needs a Redis AUTH to login
- RDS: MySql hsa a AWSAuthenticationPlugin to handle requirements around login
- ElastiCache: Memcached does multi-thread. Redis does not
- EFS: Only the root user has access at first, you must create directories and assign them to people
- S3: BucketPolicy can specify a referer

# Migrations and Transfer

- DirectConnect: Lead time is one month

# Networking & Content Delivery

- NLB: Share LB using PrivateLink. IAM is just authorization
- Gateway: On too many requests it returns a HTTP 429
- Lambda: To access another VPC ElastiCache you need to know VPC Subnet ID and VPC SG ID
- DirectConnect: To bring multi-office online, DC to nearest region then create private VIFs in each region
- ALB: Can do Cognito auth
- ALB: Cannot do weighting. Can do on-prem via ip address
- SG: Outbound is allowed by default. Inbound is not.
- NACL: Default is block out and in.
- ELB: SG allow all in, outbound as ALL TCP goes to the SG of the webservers

# Management & Governance

- CloudFormation: You associate EBS and EC2 Logical IDs together to make them work
- OpsWorks Stacks: on-prem and AWS with chef-solo
- Cloudtrail: Uses data events and management events
- CloudFormation: You can directly apply or CreateChangeSet to see changes before deploying
- TrustedAdvisor: Shows you when you are reaching AWS limits for resources
- CloudWatch: To get swap space info you need to install an agent first on the EC2 instance
- Organisations: Can be migrated via the AWS Orgs console

# Analytics

- Kinesis: Best to track user behaviour sequentially
- Kinesis: Kinesis Data Analytics is for using SQL
- Kinesis: FireHose can store data into S3/Redshift.
- Kineses: Data Streams doesn't have to go straight to a backing store

# Security, Identity and Compliance

- CloudTrail: To monitor everything, use CloudTrail in all regions and use a single KMS key
- Cognito: Can use MFA
- IAM: Has a Query API
- SSO: Use STS for roles and a AWS Federation Endpoint.
- IAM: Groups cannot be prinicples
- CloudFront: OAI is attached to the Bucket to only allow reads

# Cloud Architecture & Design

- DR: PilotLight is a warm standby of your critical services
- DR: WarmStandby is all services, but at lower resource levels

# Application Integration

- HighLoad: Use Dynamo OnDemand if available, SQS is better than Dynamo Provisioned
- SNS: Allows sending as email/json (whatever the hell that is)
- GuardDuty: Use CloudWatch to send via SNS to GuardDuty
