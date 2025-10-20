# Lesson 10: VPC with Private Subnets, NAT, and ALB (2 AZ)

## What You Will Learn
- Build a production‑ready VPC with public and private subnets across two AZs
- Place application servers in private subnets behind an ALB
- Provide outbound internet via NAT gateways
- Add an S3 gateway endpoint for private S3 access

---

## Architecture Overview
Servers run in private subnets across two AZs; an Application Load Balancer in public subnets routes traffic to them. NAT gateways in public subnets provide outbound internet for patching/dependency downloads. An S3 gateway endpoint enables private S3 access.

![VPC with two AZs](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)

---

## Hands‑On (Console)
1) Create VPC (VPC and more):
- AZs: 2; Public subnets: 2; Private subnets: 2
- NAT gateways: 1 per AZ; S3 gateway endpoint: enabled
- DNS hostnames: disabled for this demo (enable if you need internal DNS)

2) Launch Template + Auto Scaling Group:
- AMI: Amazon Linux 2/2023; Security group: allow from ALB only
- ASG subnets: the two private subnets

3) Application Load Balancer:
- Subnets: the two public subnets
- Target group: the ASG instances (health checks enabled)

4) Route Tables:
- Public: `0.0.0.0/0 -> igw`
- Private: `0.0.0.0/0 -> nat-gateway`, add S3 prefix list -> S3 gateway endpoint

5) Test:
- Access the ALB DNS name; verify app works and instances can reach internet/S3

---

## Cleanup
- Delete ASG and launch template
- Delete ALB and target group
- Delete NAT gateways, then the VPC