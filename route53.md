# DNS 101

- Route 53: 53 is the port number for DNS
- SOA: Start of Authority: name of the server who answerd, TTL, admin name
- Top level domain has a nameserver address for each domain
- A: Address! Domain -> ipv4
- CName: Cannoncial Name! Domain -> Domain
- Alias records: Map DNS to AWS resrouce (S3, Cloudfront, ELBs)
- CName cannot be used for naked domain names (apex: joejag.com) MUST be A or ALIAS
- Summary:
  - ELBs do not get pre-defined IPv4, you must use DNS to resolve them
  - Always choice ALIAS over CNAME
  - PTR is a reverse lookup from an ipv4 to an A record

# Register a daomin name

- Summary:
  - You can buy domain names with AWS, can take 3 days though

# Routing Policies

- Simple: One record to multiple IP addresses (order is randomised)
  - AWS NS is on four different TLDs
- Weighted: Give percentages for destinations
  - Add A records with one IP at a time, add weight and a "SetId" (name)
  - Can add health check which removes records
  - HealthCheck: Monitor by ip address. Can configure behaviour. Can SNS failure notification
  - Weighting is done by adding up the numbers then checking the percentage
- Latency: Based on end user connection latency to an AWS zone
- Failover: Active/Passive. Primary/Secondary. TTL should really be 60 seconds
- Geolocation Routing: If in Europe, use Europe servers (regardless of latency). Continent and Country level
  - You need to add a Default record too to handle unmapped and other calls
- Geoproximity Routing: Mix of customer and resource location
  - Create policy in a UI
  - Takes coordinates (long, lat) and a bias (weighting)
- Multivalue answers: Same as simple but with health checks
- Summary
  - ELB do not have IPv4 addresses
