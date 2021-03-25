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
