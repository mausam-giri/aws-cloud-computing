# AWS VPC Endpoint

A **VPC Endpoint** enables you to privately connect your VPC to supported AWS services and VPC endpoint services without requiring an Internet Gateway, NAT device, VPN connection, or AWS Direct Connect. Traffic between your VPC and the service does not leave the Amazon network.

---

## Table of Contents

1. [Overview](#overview)
2. [Types of VPC Endpoints](#types-of-vpc-endpoints)
3. [Prerequisites](#prerequisites)
4. [Gateway Endpoints](#gateway-endpoints)
   - [Create a Gateway Endpoint (S3 or DynamoDB)](#create-a-gateway-endpoint-s3-or-dynamodb)
5. [Interface Endpoints](#interface-endpoints)
   - [Create an Interface Endpoint](#create-an-interface-endpoint)
6. [Gateway Load Balancer Endpoints](#gateway-load-balancer-endpoints)
7. [VPC Endpoint Policies](#vpc-endpoint-policies)
8. [DNS Resolution for Interface Endpoints](#dns-resolution-for-interface-endpoints)
9. [Common Use Cases](#common-use-cases)
10. [Pricing Considerations](#pricing-considerations)
11. [Cleanup](#cleanup)

---

## Overview

Without VPC Endpoints, traffic from a private subnet to AWS services (like S3 or SSM) must travel via a NAT Gateway or Internet Gateway, which adds cost and exposes traffic to the public internet. VPC Endpoints solve this by keeping traffic private within the AWS network.

**Benefits:**
- Improved security (traffic stays within AWS network)
- Reduced data transfer costs (no NAT Gateway charges for qualifying traffic)
- Lower latency
- Works with VPC security controls (Security Groups, Endpoint Policies)

---

## Types of VPC Endpoints

| Type | Description | Supported Services |
|---|---|---|
| **Gateway Endpoint** | Free endpoint added as a route in your route table | Amazon S3, Amazon DynamoDB |
| **Interface Endpoint** | Elastic Network Interface (ENI) with a private IP in your subnet; powered by AWS PrivateLink | Most AWS services (EC2, SSM, Secrets Manager, KMS, SNS, SQS, etc.) |
| **Gateway Load Balancer Endpoint** | Routes traffic to a Gateway Load Balancer for inspection | Third-party security appliances |

---

## Prerequisites

- An AWS account with IAM permissions for `ec2:*`
- A VPC with at least one subnet
- For Interface Endpoints: subnets in the desired Availability Zones

---

## Gateway Endpoints

Gateway Endpoints are **free** and work by adding entries to your VPC route tables. They are supported for **Amazon S3** and **Amazon DynamoDB**.

### Create a Gateway Endpoint (S3 or DynamoDB)

#### Via AWS Console

1. Navigate to **VPC > Endpoints > Create Endpoint**.
2. Configure:

| Setting | Value |
|---|---|
| **Service category** | AWS Services |
| **Service name** | Search for `com.amazonaws.<region>.s3` (or `dynamodb`) |
| **VPC** | Select your VPC |
| **Route Tables** | Select route tables where you want the endpoint route added |
| **Policy** | Full access (or custom) |

3. Click **Create Endpoint**.
4. Verify the endpoint route is now present in the selected route tables:
   - Destination: `pl-xxxxxxxx` (S3/DynamoDB managed prefix list)
   - Target: `vpce-xxxxxxxxxxxxxxxxx`

#### Via AWS CLI

```bash
# Create a Gateway Endpoint for S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxxxxxxxxxxxxxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-xxxxxxxxxxxxxxxxx rtb-yyyyyyyyyyyyyyyyy \
  --vpc-endpoint-type Gateway

# Create a Gateway Endpoint for DynamoDB
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxxxxxxxxxxxxxx \
  --service-name com.amazonaws.us-east-1.dynamodb \
  --route-table-ids rtb-xxxxxxxxxxxxxxxxx \
  --vpc-endpoint-type Gateway
```

#### Verify

```bash
# List all VPC endpoints
aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=vpc-xxxxxxxxxxxxxxxxx

# Test from an EC2 instance in the VPC (no internet access needed)
aws s3 ls s3://my-bucket
```

---

## Interface Endpoints

Interface Endpoints create an **Elastic Network Interface (ENI)** in your subnet with a private IP address. Requests to the AWS service are directed to this ENI.

They support almost all AWS services including:
- AWS Systems Manager (SSM)
- AWS Secrets Manager
- Amazon EC2 API
- AWS KMS
- Amazon SNS / SQS
- Amazon CloudWatch
- AWS CodeBuild / CodeDeploy
- And many more

### Create an Interface Endpoint

#### Via AWS Console

1. Navigate to **VPC > Endpoints > Create Endpoint**.
2. Configure:

| Setting | Value |
|---|---|
| **Service category** | AWS Services |
| **Service name** | e.g., `com.amazonaws.us-east-1.ssm` |
| **VPC** | Select your VPC |
| **Subnets** | Select subnets (one per AZ recommended) |
| **Enable DNS name** | ✅ Enabled (recommended) |
| **Security Group** | Select/create a Security Group allowing HTTPS (port 443) inbound from your VPC CIDR |
| **Policy** | Full access (or custom) |

3. Click **Create Endpoint**.
4. Note the endpoint's **DNS names** (private hosted zone will resolve service hostnames to the private IP).

#### Via AWS CLI

```bash
# Create Interface Endpoint for SSM
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxxxxxxxxxxxxxx \
  --service-name com.amazonaws.us-east-1.ssm \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-xxxxxxxxxxxxxxxxx subnet-yyyyyyyyyyyyyyyyy \
  --security-group-ids sg-xxxxxxxxxxxxxxxxx \
  --private-dns-enabled

# Create Interface Endpoint for Secrets Manager
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxxxxxxxxxxxxxx \
  --service-name com.amazonaws.us-east-1.secretsmanager \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-xxxxxxxxxxxxxxxxx \
  --security-group-ids sg-xxxxxxxxxxxxxxxxx \
  --private-dns-enabled
```

#### Security Group Requirements

The Security Group attached to an Interface Endpoint must allow inbound **HTTPS (port 443)** traffic from the VPC CIDR (or specific source):

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxxxxxxxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 10.0.0.0/16
```

#### Verify

```bash
# From an EC2 instance with no internet access (no NAT)
# If private DNS is enabled, the standard service hostname resolves to the private endpoint IP
aws ssm describe-instance-information
aws secretsmanager list-secrets
```

---

## Gateway Load Balancer Endpoints

**Gateway Load Balancer (GWLB) Endpoints** work with a Gateway Load Balancer to transparently redirect traffic to a fleet of security/inspection appliances (e.g., firewalls, IDS/IPS).

> This is the mechanism used by **AWS Network Firewall** under the hood.

**Traffic flow:**

```
EC2 Instance
     │
     ▼
Subnet Route Table (→ GWLB Endpoint)
     │
     ▼
Gateway Load Balancer Endpoint (vpce-xxx)
     │
     ▼
Gateway Load Balancer
     │
     ▼
Security Appliance (EC2 or third-party)
```

Creating GWLB Endpoints requires an existing Gateway Load Balancer and endpoint service. Refer to the [AWS GWLB documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/getting-started.html) for detailed steps.

---

## VPC Endpoint Policies

All VPC Endpoints support **resource-based policies** that control which principals can access which resources through the endpoint.

### Example: Restrict S3 Gateway Endpoint to a specific bucket

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-allowed-bucket",
        "arn:aws:s3:::my-allowed-bucket/*"
      ]
    }
  ]
}
```

### Example: Restrict Interface Endpoint to specific IAM role

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyAppRole"
      },
      "Action": "ssm:*",
      "Resource": "*"
    }
  ]
}
```

Apply the policy via CLI:

```bash
aws ec2 modify-vpc-endpoint \
  --vpc-endpoint-id vpce-xxxxxxxxxxxxxxxxx \
  --policy-document file://endpoint-policy.json
```

---

## DNS Resolution for Interface Endpoints

When **private DNS** is enabled on an Interface Endpoint, the standard AWS service DNS name (e.g., `ssm.us-east-1.amazonaws.com`) automatically resolves to the **private IP** of the endpoint ENI when queried from within the VPC.

**Requirements:**
- VPC must have **DNS hostnames** and **DNS resolution** enabled
- `enableDnsHostnames = true`
- `enableDnsSupport = true`

```bash
# Enable DNS settings on the VPC
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-xxxxxxxxxxxxxxxxx \
  --enable-dns-support '{"Value": true}'

aws ec2 modify-vpc-attribute \
  --vpc-id vpc-xxxxxxxxxxxxxxxxx \
  --enable-dns-hostnames '{"Value": true}'
```

**Verify DNS resolution inside the VPC:**

```bash
# Should resolve to a private IP (e.g., 10.x.x.x), not a public IP
nslookup ssm.us-east-1.amazonaws.com
```

---

## Common Use Cases

| Use Case | Endpoint Type | Service |
|---|---|---|
| Private EC2 instances accessing S3 without NAT | Gateway | `com.amazonaws.<region>.s3` |
| Private EC2 instances using SSM Session Manager | Interface | `com.amazonaws.<region>.ssm`, `ssmmessages`, `ec2messages` |
| Lambda functions accessing Secrets Manager privately | Interface | `com.amazonaws.<region>.secretsmanager` |
| EC2 instances calling KMS for encryption without internet | Interface | `com.amazonaws.<region>.kms` |
| Centralized security inspection via firewall appliance | GWLB Endpoint | Custom endpoint service |
| DynamoDB access from private subnet | Gateway | `com.amazonaws.<region>.dynamodb` |

### Required Endpoints for SSM Session Manager (no internet access)

To use AWS Systems Manager Session Manager with instances in a fully private VPC (no NAT, no IGW), you need **all three** of the following Interface Endpoints:

| Service | Endpoint Name |
|---|---|
| SSM | `com.amazonaws.<region>.ssm` |
| SSM Messages | `com.amazonaws.<region>.ssmmessages` |
| EC2 Messages | `com.amazonaws.<region>.ec2messages` |

---

## Pricing Considerations

| Endpoint Type | Cost |
|---|---|
| **Gateway Endpoint** | Free |
| **Interface Endpoint** | Per endpoint per AZ per hour + per GB data processed |
| **GWLB Endpoint** | Per endpoint per AZ per hour + per GB data processed |

> Interface Endpoints can still reduce costs compared to NAT Gateway for high-volume AWS API traffic.

Refer to [AWS PrivateLink Pricing](https://aws.amazon.com/privatelink/pricing/) for current rates.

---

## Cleanup

```bash
# Delete a VPC endpoint
aws ec2 delete-vpc-endpoints \
  --vpc-endpoint-ids vpce-xxxxxxxxxxxxxxxxx

# List all endpoints to confirm deletion
aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=vpc-xxxxxxxxxxxxxxxxx \
  --query 'VpcEndpoints[*].{ID:VpcEndpointId,Service:ServiceName,State:State}'
```

> **Note:** Deleting a Gateway Endpoint automatically removes the endpoint routes from all associated route tables.

---

## Related Topics

- [AWS Transit Gateway – How to Setup](./transit-gateway.md)
- [Firewall Setup](./firewall-setup.md)
- [Transit Gateway and Firewall with Endpoint](./transit-gateway-firewall-endpoint.md)
