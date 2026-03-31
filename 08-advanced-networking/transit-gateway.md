# AWS Transit Gateway – How to Setup

AWS Transit Gateway acts as a **central hub** that connects multiple VPCs, VPNs, and on-premises networks through a single gateway. This guide walks through setting up an AWS Transit Gateway from scratch.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Architecture](#architecture)
4. [Step-by-Step Setup](#step-by-step-setup)
   - [Step 1: Create a Transit Gateway](#step-1-create-a-transit-gateway)
   - [Step 2: Create Transit Gateway Attachments](#step-2-create-transit-gateway-attachments)
   - [Step 3: Configure Route Tables](#step-3-configure-route-tables)
   - [Step 4: Update VPC Route Tables](#step-4-update-vpc-route-tables)
   - [Step 5: Test Connectivity](#step-5-test-connectivity)
5. [Key Concepts](#key-concepts)
6. [Pricing Considerations](#pricing-considerations)
7. [Cleanup](#cleanup)

---

## Overview

AWS Transit Gateway simplifies network architecture by eliminating the need for complex VPC peering meshes. Instead of peer-to-peer connections, all VPCs connect to a single Transit Gateway.

**Benefits:**
- Centralized routing between VPCs and on-premises networks
- Supports thousands of VPCs and on-premises connections
- Simplifies network management
- Supports multicast (optional)

---

## Prerequisites

- An AWS account with appropriate IAM permissions (`ec2:*`, `sts:AssumeRole`)
- At least two VPCs with non-overlapping CIDR blocks
- Basic understanding of AWS VPC networking

---

## Architecture

```
          ┌─────────────┐
          │    VPC A    │
          │ 10.0.0.0/16 │
          └──────┬──────┘
                 │ TGW Attachment
          ┌──────┴──────────────┐
          │  Transit Gateway    │
          │  (Central Hub)      │
          └──────┬──────────────┘
                 │ TGW Attachment
          ┌──────┴──────┐
          │    VPC B    │
          │ 10.1.0.0/16 │
          └─────────────┘
```

---

## Step-by-Step Setup

### Step 1: Create a Transit Gateway

1. Open the **AWS Management Console** and navigate to **VPC > Transit Gateways**.
2. Click **Create Transit Gateway**.
3. Configure the following settings:

| Setting | Recommended Value |
|---|---|
| **Name** | `my-transit-gateway` |
| **Description** | Central hub for VPC connectivity |
| **Amazon side ASN** | `64512` (default, or custom private ASN) |
| **DNS Support** | Enabled |
| **VPN ECMP Support** | Enabled |
| **Default route table association** | Enabled |
| **Default route table propagation** | Enabled |
| **Multicast support** | Disabled (enable only if needed) |

4. Click **Create Transit Gateway**.
5. Wait for the state to change from `pending` to `available` (usually 2–5 minutes).

> **CLI equivalent:**
> ```bash
> aws ec2 create-transit-gateway \
>   --description "Central hub for VPC connectivity" \
>   --options AmazonSideAsn=64512,DnsSupport=enable,VpnEcmpSupport=enable,DefaultRouteTableAssociation=enable,DefaultRouteTablePropagation=enable
> ```

---

### Step 2: Create Transit Gateway Attachments

Each VPC must be attached to the Transit Gateway via a **TGW VPC Attachment**.

1. Navigate to **VPC > Transit Gateway Attachments**.
2. Click **Create Transit Gateway Attachment**.
3. Fill in the details:

| Setting | Value |
|---|---|
| **Transit Gateway ID** | Select your Transit Gateway |
| **Attachment type** | VPC |
| **VPC ID** | Select VPC A |
| **Subnet IDs** | Select one subnet per Availability Zone |

4. Click **Create Transit Gateway Attachment**.
5. Repeat steps 2–4 for **VPC B** (and any additional VPCs).

> **CLI equivalent:**
> ```bash
> # Attach VPC A
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-xxxxxxxxxxxxxxxxx \
>   --subnet-ids subnet-xxxxxxxxxxxxxxxxx
>
> # Attach VPC B
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-yyyyyyyyyyyyyyyyy \
>   --subnet-ids subnet-yyyyyyyyyyyyyyyyy
> ```

> **Note:** Wait for attachments to show state `available` before proceeding.

---

### Step 3: Configure Route Tables

If **Default Route Table Association** is enabled (recommended for basic setup), VPC attachments are automatically associated with the default Transit Gateway route table and routes are automatically propagated.

**To verify the default route table:**
1. Navigate to **VPC > Transit Gateway Route Tables**.
2. Select the default route table and review the **Routes** tab.
3. Confirm that routes for both VPC CIDRs appear (e.g., `10.0.0.0/16` and `10.1.0.0/16`).

**To manually add a static route (if needed):**
1. Go to **Transit Gateway Route Tables > Routes > Create route**.
2. Enter the destination CIDR and select the attachment.

> **CLI equivalent:**
> ```bash
> aws ec2 create-transit-gateway-route \
>   --transit-gateway-route-table-id tgw-rtb-xxxxxxxxxxxxxxxxx \
>   --destination-cidr-block 10.1.0.0/16 \
>   --transit-gateway-attachment-id tgw-attach-xxxxxxxxxxxxxxxxx
> ```

---

### Step 4: Update VPC Route Tables

Each VPC's route table must have a route directing traffic destined for the other VPC's CIDR to the Transit Gateway.

1. Navigate to **VPC > Route Tables**.
2. Select the route table associated with the subnets in **VPC A**.
3. Click **Edit routes > Add route**.

| Destination | Target |
|---|---|
| `10.1.0.0/16` (VPC B CIDR) | Transit Gateway ID |

4. Click **Save changes**.
5. Repeat for **VPC B**, adding a route to `10.0.0.0/16` via the Transit Gateway.

> **CLI equivalent:**
> ```bash
> # Add route in VPC A route table pointing to Transit Gateway for VPC B traffic
> aws ec2 create-route \
>   --route-table-id rtb-xxxxxxxxxxxxxxxxx \
>   --destination-cidr-block 10.1.0.0/16 \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx
> ```

---

### Step 5: Test Connectivity

1. Launch an EC2 instance in a subnet in **VPC A**.
2. Launch an EC2 instance in a subnet in **VPC B**.
3. Ensure **Security Groups** allow ICMP (ping) inbound traffic.
4. SSH into the instance in VPC A and ping the private IP of the instance in VPC B:

```bash
ping 10.1.x.x
```

You should see successful replies, confirming Transit Gateway routing is working.

---

## Key Concepts

| Term | Description |
|---|---|
| **Transit Gateway (TGW)** | Central routing hub connecting VPCs and on-premises networks |
| **TGW Attachment** | A connection between the TGW and a VPC, VPN, or Direct Connect |
| **TGW Route Table** | Defines how traffic is routed between attachments |
| **Propagation** | Automatically adds attachment routes to a TGW route table |
| **Association** | Links an attachment to a specific TGW route table for routing decisions |

---

## Pricing Considerations

- **Per attachment:** Charged per Transit Gateway attachment per hour
- **Per GB processed:** Charged per GB of data processed by the Transit Gateway
- **Inter-Region peering:** Additional charges apply for cross-region TGW peering

Refer to [AWS Transit Gateway Pricing](https://aws.amazon.com/transit-gateway/pricing/) for current rates.

---

## Cleanup

To avoid unnecessary charges, clean up resources in this order:

1. Delete VPC route table entries pointing to the TGW
2. Delete Transit Gateway Attachments
3. Delete the Transit Gateway
4. Terminate EC2 instances (if created for testing)

```bash
# Delete attachment
aws ec2 delete-transit-gateway-vpc-attachment \
  --transit-gateway-attachment-id tgw-attach-xxxxxxxxxxxxxxxxx

# Delete Transit Gateway
aws ec2 delete-transit-gateway \
  --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx
```

---

## Related Topics

- [AWS Transit Gateway with Multiple Routing Tables](./transit-gateway-multiple-routing-tables.md)
- [Transit Gateway and Firewall with Endpoint](./transit-gateway-firewall-endpoint.md)
- [VPC Endpoint](./vpc-endpoint.md)
