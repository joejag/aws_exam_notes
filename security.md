# Security

## Reducing Security threats

- Use NACL to DENY an ip address with a lower weight than other rules
- When using an ALB the connect terminates there, not on the host
- can introduce a WAF. WAF also does SQL injection stuff. Prefer WAF for public sites
- Can add WAF onto Cloudfront. NACL gets CLoudFront ip, so doesn't work there
- WAF can block an entire country
- NLB does not terminate traffic. Must use a NACL here

## Key Management Service (KMS)

- Regional service for enc and decry
- Manages Customer Master Keys (CMKs)
- use case: S3, DB Passwords, API Keys stored in Systems Manager Parameter Store
- Size limit is 4kb
- Pay per API call
- Audit via CloudTrail - logs in S3
- FIPS 140-2 Level 2 service
- Level 3 is CloudHSM
- Three types of CMKs
  - Customer Managed (view, manage, dedicated to account) - allows rotation, can enable/disable
  - AWS Managed CMK (view, dedicated to account) - default
  - AWS Owned CMK (manage) - rare
- Two types of crypto
  - Symmetric
    - CustomerManaged be default. AES-256. Most services. Can import your own
  - Asymmetric
    - RSA, eliptic-curve crypto (ECC). Can take public key outside of AWS - for users who cannot call the KMS API. No async CMKs
- Create custome key then : `aws kms encrypt <key> <file>` file:// fileb://
- Do not need to reference the key on decrypt as the key is in the metadata inside the file
- Bigger than 4kb? Need a Data Encryption Key (DEK). Sends just the enc key, so you don't have to send the entire file across the network. Need to store the CipherTextBlob with the thing enc. This is "Enveloper Encryption".

## CloudHSM

- Tamper resistent environemtn for keys
- Dedicated hardware security module (HSM)
- FIPS 140-2 Level 3 (US gov standard FIPS)
- Here you manage your own keys
- Single tenant, dedicated hardware, multi-AZ cluster arch
- Industry APIs, not AWS APIs
- Works with PKCS#11, Java Crypto (JCE), MS (CNG)
- If you lose the keys, they are gone
- Create the cluster in a new or existing VPC
  - Projects ENIs into other VPCs you choose
  - Not multi-AZ by default
- Summary:
  - Strict regulatory compliance or FIPS Level 3 == HSM

## Systems Manger Parameter Store

- Component of AWS Systems Manager (SSM)
- Serverless storage for secrets (passwords, db connection, license)
- Values are enc with KMS or plain text
- Store params in hierarchies
- Can track versions, rollback. TTL on passwords for rotation
- Stored in trees like /dev/db/pass
- Can give perms at levels of the tree
- SSM
  - Tiers dictate size of value
  - Can choose KMS key is using a SecureString

## SecretsManager

- Parameter store has no additional charges. Charge per secret and 10k calls
- SecretesManager has auto-rotate for secrets for RDS
- Can generate random secrets

## AWS Shield

- For dDoS attacks
- AWS Shield Standard
  - Comes for free with WAF. Protects against layer 3 and 4 attacks
    - SYN/UDP floods (open a connection)
    - Reflection attacks (hack the source of a connection)
- AWS Sheild Advanced
  - $3k a month per AWS org
  - Enhanced protection for EC2, ELB, CloudFront, GlobalAccelerator, Route 53
  - 24/7 access to a DDoS Response Team
  - DDos Cost Protection - insurance

## Web Application Firewall (WAF)

- Monitor HTTP(s) requests to CloudFront, ALB, API Gateway
  - control access to content
  - configure filtering rules for traffic
    - filter by: ip addy, query string params, SQL injection
    - Gives back 403 for these things when blocked
- Has three behaviours
  - Allow all requests, except ones you specify
  - Block all requests, except ones you specify
  - Count requests, that match properties you specify
    - originating ip address
    - origin country
    - request size
    - values in http headers
    - regex on strings in requests
    - SQL code injection
    - XSS prevention
- AWS Firewall Manager
  - Configure rules across an AWS Org
  - WAF rules for ALB, API Gateway, Cloudfront
  - AWS Shield Advanced: ALB, ELB, EIP, CloudFront (applies dDos protect to all resources or Tagged ones)
  - Configure SGs for ENIs and EC2s - and audit
