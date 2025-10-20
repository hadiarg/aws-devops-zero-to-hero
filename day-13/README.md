# Lesson 13: VPC Endpoints (Gateway and Interface) Hands‑On

## What You Will Learn
- Difference between Gateway and Interface VPC endpoints
- Add an S3 Gateway endpoint to private subnets
- Add an Interface endpoint (for example, Systems Manager or ECR) with security group controls
- Test private connectivity without using an Internet Gateway

---

## Concepts
- Gateway endpoint: Route‑table entry to S3/DynamoDB using AWS prefix lists
- Interface endpoint: ENI with private IPs in your subnets; controlled by SGs; billed hourly + data

---

## Lab: Configure Endpoints

### 1) S3 Gateway Endpoint
- VPC console → Endpoints → Create endpoint
- Service: com.amazonaws.<region>.s3 (Gateway)
- Route tables: select the two private subnet route tables
- Save

Verify routes added:
- Private RT adds: `pl-xxxx (S3 prefix list) -> vpce‑gateway`

### 2) Interface Endpoint (example: SSM)
- Service: com.amazonaws.<region>.ssm (Interface)
- Subnets: the two private subnets
- Security group: allow HTTPS 443 from your instances' SG
- Enable private DNS (recommended)

### 3) Test from a Private Instance
```bash
# S3 via gateway endpoint
aws s3 ls s3://aws-cli/ --region <region>

# SSM via interface endpoint (if SSM agent/role configured)
sudo systemctl status amazon-ssm-agent
```
- Confirm no egress to Internet is required for these services

---

## Cost & Security Notes
- Gateway endpoints for S3/DynamoDB have no hourly cost
- Interface endpoints incur hourly and data processing charges
- Restrict interface endpoint SGs to least privilege (only necessary sources, 443)

---

## Cleanup
- Delete interface endpoints not needed
- Remove gateway endpoint route table associations if no longer used