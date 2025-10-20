# Lesson 15: Security Groups vs Network ACLs (Best Practices)

## What You Will Learn
- Differences between Security Groups (SGs) and Network ACLs (NACLs)
- When to use SGs vs NACLs
- Practical rules for a web tier behind an ALB
- Quick verification steps

---

## Core Differences
- Security Groups
  - Stateful: return traffic is automatically allowed
  - Attached to ENIs/instances, evaluated on allow rules
  - Easier to reason about for application access control
- Network ACLs
  - Stateless: must allow both inbound and outbound explicitly
  - Attached to subnets, evaluated by numbered rules (lowest first)
  - Useful for coarse‑grained subnet‑level controls

---

## Recommended Pattern (ALB → Private Instances)
- ALB SG (public subnets)
  - Inbound: 80/443 from 0.0.0.0/0 (or your CIDR)
  - Outbound: 80/443 to instance SG
- Instance SG (private subnets)
  - Inbound: 80 from ALB SG only
  - Outbound: 443 to required AWS services/endpoints (or 0.0.0.0/0 if NAT + egress filter elsewhere)
- NACLs
  - Keep default allow unless you have specific compliance requirements
  - If restricting, ensure both directions are allowed for needed ports

---

## Example SG Rules
- ALB SG inbound: HTTP 80 from 0.0.0.0/0 (or your CIDR)
- ALB SG outbound: HTTP 80 to Instance SG
- Instance SG inbound: HTTP 80 from ALB SG
- Instance SG outbound: HTTPS 443 to 0.0.0.0/0 or to interface endpoint SGs

---

## Verification
1) From your browser, open the ALB DNS name → expect 200 OK
2) Stop one instance → ASG replaces it; health checks recover
3) Block ALB→Instance in Instance SG → targets become unhealthy immediately

---

## Tips
- Prefer SG references over CIDRs when possible (tighter coupling)
- Use VPC endpoints for AWS service access from private subnets
- Audit with VPC Reachability Analyzer and AWS Config

---

## Cleanup
- Revert temporary rule changes
- Remove unused SGs/NACL entries