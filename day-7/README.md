# Lesson 7: VPC with Servers in Private Subnets and NAT

## What You Will Learn
- Design a secure, highly available VPC for a 2‑tier app
- Place application servers in private subnets across two AZs
- Use ALB for ingress and NAT gateways for egress
- Enable private S3 access with a gateway endpoint

---

## Architecture Overview
- Public subnets: ALB + NAT gateways
- Private subnets: application instances managed by an Auto Scaling Group
- One NAT gateway per AZ; S3 gateway endpoint for private S3 access

![VPC Architecture](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)

---

## Network Paths and Route Tables
- Public: `0.0.0.0/0 -> igw` (and `::/0 -> igw` if IPv6)
- Private: `0.0.0.0/0 -> nat-gateway`, `::/0 -> eigw` (IPv6), S3 prefix list -> S3 gateway endpoint

Why:
- Internet → ALB → Private instances
- Private instances → NAT → Internet; S3 via endpoint (no internet)

Quick checks:
- Browse ALB DNS; expect 200 OK
- From a private instance: `curl https://s3.amazonaws.com`, `aws s3 ls`

---

## Steps (Console)
1) Create VPC (VPC and more): 2 AZs, 2 public + 2 private, 1 NAT per AZ, S3 endpoint
2) Launch template + ASG in private subnets
3) ALB in public subnets → target group → attach ASG

---

## Security
- ALB SG: inbound 80/443 from Internet; outbound to instance SG
- Instance SG: inbound 80 from ALB SG only; outbound 443

---

## Cleanup
- Delete ASG, ALB/target group, NAT gateways, VPC
