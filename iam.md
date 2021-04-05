# IAM

## Intro

- It's universal, not by region
- Use a root account just for billing, make a new user straight away
- New users have no perms. Can have Console or API key access
- You can configure MFA and password rotation policies

## Billing

- Cloudwatch > Alarms > Billing > Create (uses SNS to email)
- Shows alarm as "insuffcient" until 6 hours have passed

# Organisatio and Consolidated Billing

- Create multiple accounts off the Root.
- Root > OrgUnit (OU) > Account
- OU can have another OU under it. OU can have multiple Accounts
- Consolidated Billing: More you use, less you pay
  - one bill
  - easy to track charges and allocate
  - volume pricing discounts
- Can apply Service Control Polices (SCP) to OUs or Accounts

# Workshops

- Deny overrides Allow in IAM policies

# Exam

- PowerUser is a JobFunction that allows everything but IAM access

# Advanced IAM

## AWS Directory Service

- Bunch of services, let's you connect AWS Resources to on-prem AD
- Standalone directory in the cloud
- Use existing corporate creds, also including AWS Console
- SSO sign on for any domain-joined EC2 instance (ssh level)
- AD?
  - On-prem directory service. Trees/Forests of group policies for users and devices
  - LDAP and DNS. Kerberos, LDAP and NTLM auth
  - HA
- AWS Managed MS AD
  - AD Domain Controllers (DC) running on Win
  - Two DCs in 2 AZs
  - Reachable by things inside the VPC
  - Can add extra DC for Availability or transaction rates
  - Exclusive DCs - not shared
  - Can extend to on-prem using AD Trust
  - Responsibilities
    - AWS: Multi-Az, patching/recovery, snapshot/restore
    - Customer: user/groups, scaling by choice, AD Trusts, certs on LDAPS, federation
- AWS Simple AD
  - Standlone Directory
  - Small (500 users), Large (5000 users)
  - Easier to manage EC2 instances. No password management
  - No AD Trust - cannot join to on-prem
- AD Connector
  - Proxying on-prem AD
  - Avoid caching info, allows on-prem to use creds
  - Can scale across multiple AD Connectors
- Cloud Directory (just LDAP)
  - Directory store for developers
  - Use cases: org-chart apps, course catalogs, device registries
- Amazon Cognito User Pools
  - Managed user directory for SAAS applicaitons
  - Does sign-up and login for web and mobile
  - OAuth kinda, use Facbook etc to login
- AD vs not-AD
  - Yes: Managed MS AD (DS), AD Connector, Simple AD - for Worksite
  - No: Cognito User Pools, Cloud Directory

## IAM Polcies

- Amazon Resource Name (ARN): Unique in AWS.
  - Format: arn:partition:service:region:accout_id
    - partition: AMZ, China, US Gov
- Identity Policy for Humans, Resources Policies for Resources
- Police are attached to an idnentiy or resource
- IAM Policies are a list of statements with a version number
  - Sid: Identifier for humans
  - Effect: Allow / Deny
  - Action: List of service and action
  - Resrouce: ARN
- Usage: Create policy -> Add to role -> Add to real thing like EC2 (or identity)
- Summary
  - Not explicitly allowed == implicity denied
  - Explicty Deny overrides everything
  - Policies only matter when they are attached
- Permission Boundries

  - Set the maximum possible perms for a user - does not grant those perms
  - Used to delegate admin to other users
  - Prevents privilege escalation and unnecesary broad perms
  - Controls the maximum perms an IAM policy can grant
  - Use cases: Devs who create roles for Lambda functions, EC2 roles, creating ad-hoc users
  - Exist on user screen in IAM
  - SCP: Perm Boundires for Accounts

## AWS Resources Access Manager (RAM)

- Multi account stragegy. Reduce blast radius, but still need to share ARN between accounts. RAM enters
- Share Resources across accounts
  - not all services: App Mesh, Aurora, CodeBuild, EC2, EC2 Image Builder, License Manager, Resource Groupss, Route 53
- RAM Console app: Shared by me, shared with me
- Once shared, an invitation goes to the other account to accept
  - Looks like you can only clone rather than shared Aurora
- Can share outwith your whole AWS account

## AWS Single Sign-On

- Central management for access to AWS accounts and business apps
- Oauth stuff
- Proxies your identity stuff to use as an SSO for other places
- Can use to set policies for groups to accounts
- Works with identity provides using SAML (Azure AD, On prem AD)
- Access Github, access intenerl AWS Console
- Any SAML 2.0 enabled app
- All sign on acitivites go into CloudTrail
- SAML will always mean look for SSO

## Advanced IAM Summary

- Policies are EARS (Effect, Action, Resource)
- Existing identities is for SSO to GSuite etc.
