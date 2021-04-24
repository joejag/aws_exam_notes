# A Cloud Guru: Exam 1

- CloudWatch: cannot see EC2 memory as it cannot get onto the box
- DynamoDB: max size is 400kb name+value
- EBS: Enc at rest in: EBS, EFS, S3
- EC2: is dedicated or shared. It cannot change after creation
- EC2: spread placement is max 7 per AZ
- NAT: You can go to m4.xlarge to get more network bandwidth for NATs
- S3: "resource owner" is the AWS Account that creates buckets & objects
- S3: used to have account name as a path, it is now "virtual" by being a sub-domain
- SQS: Long Polling is set from 1-20 Seconds
- SQS: max visibility:12h, max retention:14d
- Storage Gateway: on prem for NFS or partial backup
- Support: Dev (nothing), Biz (1h prod), Enterprise (15m biz critical + TAM)
- VPC: subnets: biggest:16, smallest: 28

# A Cloud Guru: Exam 2

- DynamoDB: Already multi-AZ
- Spot: On termination: 120s, might miss the term notice, can get msg despite SpotBlocks in place (will kill)
- SQS: WaitTimeSeconds controls long polling
- VPC: Can set shared or dedicated hardware for EC2 at this level. This can be changed.

# Digital Cloud Training: Exam 1

- ALB: must be in a public subnet, but can target private subnet stuff
- AWS Inspector: cannot apply changes, just suggest
- Cloudfront: Block a country? use Cloudfront (or maybe a WAF) - not NACL
- Cloudfront: can choose regions by "price class"
- DynamoDB: You can autoscale DynamoDB throughput
- EBS: Cold/throughput cannot go to 3k iOPs, use SSD
- EBS: You can make a a 64k iops database with io1/2
- EC2: You can do "scheduled reserved instances"
- ECS: To add roles use taskRoleArn. Instances can have multiple tasks
- Endpoints: in-VPC? use Gateway (s3/dynamo), otherwise use Interface
- Global Accelerator -> NLB -> Instances
- IAM: SCP are a DENY all with an allow
- Kinesis: If it says "near real-time" use Kinesis
- RDS: For enc in transit, you can download an AWS root cert to use
- RDS: Read replicas cannot be enc if the host is not enc

# Compute notes

- ASG Health: Uses EC2 health check by default (can use ELB health checks instead), terminates if unhealthy, waits for inflight connections to finish (connection draining), health check for new instances has a 300 second grace period
- AutoScale: Can be suspended. Can put instances into a Standby state
- AutoScale: If you add existing EC2 instancds to a ASG target group and it exceeds the max for hte ASG then the request will fail
- AutoScale: To terminate, it chooses the the AZ with the most
- AWS Batch: Replace 3rd party stuff for analytics and spinning up EC2. Can do HPC
- Credentials: Store them in 'Systems Manager Parameter Store'
- Dedicated: Only choose 'host' if licensing comes into it
- EC2: If terminated it could be EBS snapshot corrupt or EBS volume limit reached
- EC2: OnDemand is quicker to spin up then Spot
- ECS: There is a 'Amazon ECS optimized AMI' which installs tools. When not working: Check the IAM instance profile has the right perms, check the agent is running.
- Lambda: Use KMS to encrypt passwords as ciphertext
- LaunchConfig: Are immutable
- Metadata: There is a 'Instance Metadata Query' tool that uses the 169.254 address to get similar info
- RunCommand: Run a command across Windows EC2 instances

# Storage notes

- EBS Backups: use 'EBS Data Lifecycle Manager'
- EBS: SSD goes to 16k iOps
- EFS: Security via SG and POSIX perms
- Glacier: Expediated is minutes, Standard is 3-5 hours, bulk is 5-12
- S3 home dir: Create IAM policy with folder perms, add users to that group
- S3: Glue is ETL, can be kicked off by Lambda

# Database notes

- DynamoDB: Streams allow you to kick off Lambdas
- DynamoDB: There is an S3 pointer type: S3 Object ID
- DynamoDB: You get a 400 if you use all your writers up (ProvisionedThroughputExceededException)
- EFS: Only the root user has access at first, you must create directories and assign them to people
- ElastiCache: Memcached does multi-thread. Redis does not
- ElastiCache: Redis cannot have an IAM role, needs a Redis AUTH to login
- ElastiCache: Redis persistence is good for high perf / persistence
- RDS: encryption is by the key in that region, so read replicas use a different one
- RDS: MySql hsa a AWSAuthenticationPlugin to handle requirements around login
- RDS: Restore works withing 5m and uses the default SG rather than a custom one
- RDS: To connect on prem: update SG and make instance available in a public subnet
- S3: BucketPolicy can specify a referer
- S3: To analyse in place you can use S3 Select or Redshift Spectrum

# Migrations and Transfer

- DirectConnect: Lead time is one month

# Networking & Content Delivery

- ALB: Can do Cognito auth
- ALB: Cannot do weighting. Can do on-prem via ip address
- DirectConnect: To bring multi-office online, DC to nearest region then create private VIFs in each region
- ELB: SG allow all in, outbound as ALL TCP goes to the SG of the webservers
- Gateway: On too many requests it returns a HTTP 429
- Lambda: To access another VPC ElastiCache you need to know VPC Subnet ID and VPC SG ID
- NACL: Default is block out and in.
- NLB: Share LB using PrivateLink. IAM is just authorization
- SG: Outbound is allowed by default. Inbound is not.

# Management & Governance

- CloudFormation: You associate EBS and EC2 Logical IDs together to make them work
- CloudFormation: You can directly apply or CreateChangeSet to see changes before deploying
- Cloudtrail: Uses data events and management events
- CloudWatch: To get swap space info you need to install an agent first on the EC2 instance
- OpsWorks Stacks: on-prem and AWS with chef-solo
- Organisations: Can be migrated via the AWS Orgs console
- TrustedAdvisor: Shows you when you are reaching AWS limits for resources

# Analytics

- Kineses: Data Streams doesn't have to go straight to a backing store
- Kinesis: Best to track user behaviour sequentially
- Kinesis: FireHose can store data into S3/Redshift.
- Kinesis: Kinesis Data Analytics is for using SQL

# Security, Identity and Compliance

- CloudFront: OAI is attached to the Bucket to only allow reads
- CloudTrail: To monitor everything, use CloudTrail in all regions and use a single KMS key
- Cognito: Can use MFA
- IAM: Groups cannot be prinicples
- IAM: Has a Query API
- SSO: Use STS for roles and a AWS Federation Endpoint.

# Cloud Architecture & Design

- DR: PilotLight is a warm standby of your critical services
- DR: WarmStandby is all services, but at lower resource levels

# Application Integration

- GuardDuty: Use CloudWatch to send via SNS to GuardDuty
- HighLoad: Use Dynamo OnDemand if available, SQS is better than Dynamo Provisioned
- SNS: Allows sending as email/json (whatever the hell that is)
