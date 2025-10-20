# Lesson 6: Amazon Route 53 – DNS for Highly Available Architectures (Exam‑Ready)

## What You Will Learn
- Create and manage public and private hosted zones
- Configure A/AAAA/CNAME/ALIAS and health‑checked failover records
- Use weighted, latency‑based, and geolocation routing policies
- Integrate Route 53 with ALB/CloudFront/API Gateway and on‑prem DNS

---

## Core Concepts
- Hosted Zones: Containers for DNS records (public Internet or VPC‑scoped private)
- Record Types:
  - A / AAAA: IPv4/IPv6 addresses
  - CNAME: Canonical name (alias to another name; not for zone apex)
  - ALIAS: AWS‑specific pointer usable at zone apex (ALB, CloudFront, S3 website, API Gateway)
- Health Checks: Monitor endpoints (HTTP/HTTPS/TCP) and integrate with DNS failover
- Routing Policies: Simple, Weighted, Latency‑based, Failover, Geolocation, Geoproximity, Multivalue answer

---

## Hands‑On: Public Web App with ALB and Failover

### 1) Create a Public Hosted Zone
- Domain: `example.com` (registered in Route 53 or another registrar)
- Note the NS and SOA records; update registrar to use Route 53 name servers if external

### 2) Add ALIAS Record to ALB
- Create an ALB for your app (see Lesson 11)
- In Route 53 → Hosted zone → Create record:
  - Name: `app.example.com`
  - Type: `A – IPv4 address`
  - Alias: Yes → Target: your ALB DNS

### 3) Configure Health‑Checked Failover
- Create a Route 53 health check pointing to your primary endpoint (ALB or HTTP URL)
- Records:
  - Primary: `app.example.com` ALIAS to primary ALB with Failover: Primary and attach health check
  - Secondary: `app.example.com` ALIAS to backup endpoint (e.g., static S3 website/another ALB) with Failover: Secondary
- Test: Stop primary target; DNS should serve the secondary after health check fails

---

## Advanced Policies (Exam Focus)
- Weighted: Split traffic for blue/green or canary (e.g., 90/10). Useful with CodeDeploy/CodePipeline.
- Latency‑based: Route to region with lowest latency for the resolver’s location
- Geolocation: Route by user location (e.g., country/continent)
- Multivalue Answer: Return multiple healthy IPs (basic load distribution)

Tip: You can combine health checks with weighted records to shift traffic progressively during deployments.

---

## Private Hosted Zones (PHZ)
- Scope: One or more VPCs
- Use cases: Service discovery within VPC, split‑horizon DNS
- Steps:
  - Create PHZ: `corp.local`
  - Associate with target VPC(s)
  - Create records: `api.corp.local` → NLB/EC2 private IP

---

## Integrations
- CloudFront: Use ALIAS at zone apex to your distribution; supports global CDN
- API Gateway: ALIAS records for custom domains; attach ACM cert in us‑east‑1 for edge‑optimized
- S3 Static Website: Use S3 website endpoint with ALIAS; bucket name must match record name
- Hybrid DNS: Use Route 53 Resolver endpoints to query/forward to on‑premises DNS

---

## Troubleshooting
- Propagation: TTL and resolver caching delay updates; use `dig`/`nslookup`
- Health Check flapping: Increase thresholds, verify security groups/firewalls allow probes
- ALIAS vs CNAME: Use ALIAS at zone apex; CNAME not allowed at apex
- Mixed IPv4/IPv6: Create both A and AAAA (and enable IPv6 on your ALB if required)

---

## Cleanup
- Remove test records and health checks
- Delete unused hosted zones

---

## Exam Tips
- Prefer ALIAS over CNAME for zone apex and AWS targets
- Use health‑checked failover for DR; combine with S3 static site or multi‑region ALB
- Weighted routing for canary/blue‑green with pipelines
- Private hosted zones for intra‑VPC service discovery; use Resolver for hybrid
