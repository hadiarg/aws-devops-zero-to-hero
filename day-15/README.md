# Lesson 7: VPC with Servers in Private Subnets and NAT

## What You Will Learn
- Create a production-ready VPC with servers deployed in **private subnets across two Availability Zones**.
- Use an **Auto Scaling group** and **Application Load Balancer** for high availability and scalability.
- Enable secure outbound internet access using **NAT gateways** (one per AZ).
- Provide private access to **Amazon S3** using a **gateway VPC endpoint**.

---

## Overview
This example demonstrates how to create a VPC for servers in a production environment. To improve resiliency, servers are deployed in two Availability Zones using an Auto Scaling group and an Application Load Balancer. For additional security, servers are deployed in private subnets. They receive requests through the load balancer, connect to the internet via NAT gateways, and access Amazon S3 through a gateway VPC endpoint.

![A VPC with subnets in two Availability Zones](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)

The VPC includes:
- Public and private subnets in **two Availability Zones**
- One **NAT gateway** and one **load balancer node** in each public subnet
- Application **servers in private subnets**, managed by an **Auto Scaling group**
- A **gateway VPC endpoint for Amazon S3**

---

## Routing

### Public Subnet Route Table
| Destination               | Target        |
|---------------------------|---------------|
| `10.0.0.0/16`             | local         |
| `2001:db8:1234:1a00::/56`| local         |
| `0.0.0.0/0`               | `igw-id`      |
| `::/0`                    | `igw-id`      |

> If you create IPv4-only subnets, only the IPv4 routes appear.

### Private Subnet Route Table
| Destination               | Target            |
|---------------------------|-------------------|
| `10.0.0.0/16`             | local             |
| `2001:db8:1234:1a00::/56`| local             |
| `0.0.0.0/0`               | `nat-gateway-id`  |
| `::/0`                    | `eigw-id`         |
| `s3-prefix-list-id`       | `s3-gateway-id`   |

> The last route sends traffic destined for Amazon S3 to the gateway VPC endpoint.

---

## Security

Security group rules for servers:

| Source                                | Protocol            | Port Range         | Comments                                               |
|---------------------------------------|---------------------|--------------------|--------------------------------------------------------|
| ID of the load balancer security group| listener protocol   | listener port      | Allows inbound traffic from the load balancer          |
| ID of the load balancer security group| health check protocol| health check port | Allows inbound health check traffic from the load balancer |

---

## 1. Create the VPC

- Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/).
- On the dashboard, choose **Create VPC**.
- For **Resources to create**, choose **VPC and more**.
- **Configure the VPC**:
  - Enter a name for the VPC.
  - Keep or customize the **IPv4 CIDR block**.
  - (Optional) Enable **Amazon-provided IPv6 CIDR block**.
- **Configure the subnets**:
  - **Number of Availability Zones**: `2`
  - **Number of public subnets**: `2`
  - **Number of private subnets**: `2`
- **NAT gateways**: Choose **1 per AZ**.
- **Egress-only internet gateway**: Choose **Yes** if using IPv6.
- **VPC endpoints**: Keep **S3 Gateway** enabled (no cost).
- **DNS options**: Clear **Enable DNS hostnames**.
- Choose **Create VPC**.

---

## 2. Deploy Your Application

- Create a **launch template** to specify configuration for EC2 instances.
- Create an **Auto Scaling group** across the private subnets.
- Create an **Application Load Balancer** in the public subnets.
- Attach the load balancer to the Auto Scaling group.

---

## 3. Test Your Configuration

- Access your application via the load balancer DNS name.
- Verify servers can reach the internet and Amazon S3.
- Use **Reachability Analyzer** to troubleshoot connectivity issues.

---

## 4. Clean Up

- Delete the Auto Scaling group.
- Delete the load balancer.
- Delete the NAT gateways.
- Delete the VPC.