# Lesson 11: Highly Available Web Tier with ALB + Auto Scaling

## What You Will Learn
- Create an ALB across two AZs
- Deploy an Auto Scaling Group in private subnets
- Lock down instance access using security groups
- Validate health checks and failover

---

## Lab Steps

### 1) Networking Prereqs
- Use the VPC from Lesson 10 (2 public + 2 private subnets)
- Ensure NAT gateways exist in both AZs

### 2) Launch Template
- AMI: Amazon Linux 2/2023
- User data: simple HTTP server on port 80 (for example, install httpd and start service)
- Security group: allow inbound 80 only from ALB security group

### 3) Target Group
- Type: Instance, Protocol: HTTP:80
- Health check path: `/` (or your app path)

### 4) Application Load Balancer
- Scheme: Internet‑facing
- Subnets: the two public subnets
- Listener: HTTP:80 → forward to target group

### 5) Auto Scaling Group
- Subnets: the two private subnets
- Desired/Min/Max: 2/2/4 (example)
- Attach to target group

### 6) Test & Observe
- Open ALB DNS name in browser; verify round‑robin responses
- Stop one instance; ASG should replace it
- Observe health checks turning healthy/unhealthy appropriately

---

## Security Best Practices
- Only ALB SG should reach instance SG (no 0.0.0.0/0 to instances)
- Restrict SSH via SSM Session Manager instead of opening port 22
- Use IAM roles on instances (no static credentials)

## Advanced ALB Patterns (Exam Focus)
- **Lambda Targets**: Register Lambda functions as targets in target groups
- **Path‑Based Routing**: Create listener rules with path conditions for different endpoints
- **Weighted Target Groups**: Gradually shift traffic between EC2 and Lambda targets
- **Blue/Green Testing**: Use weighted routing to test new Lambda functions with limited traffic

## Blue/Green Deployment Pattern
- Create separate ALBs for blue and green environments
- Use Route 53 weighted routing to gradually shift traffic (10% → 50% → 100%)
- Both environments can share the same RDS Multi‑AZ database during transition
- **Exam Tip**: ALB + ASG + Route 53 weighted routing = gradual traffic shift

---

## Cleanup
- Delete ASG and launch template
- Delete ALB and target group