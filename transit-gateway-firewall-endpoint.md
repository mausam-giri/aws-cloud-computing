# AWS Transit Gateway and Firewall with Endpoint

This guide explains how to integrate **AWS Transit Gateway (TGW)** with **AWS Network Firewall** using **VPC Endpoints** to enable centralized, scalable traffic inspection across multiple VPCs.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Architecture](#architecture)
4. [Traffic Flow](#traffic-flow)
5. [Step-by-Step Setup](#step-by-step-setup)
   - [Step 1: Create VPCs](#step-1-create-vpcs)
   - [Step 2: Deploy AWS Network Firewall in the Inspection VPC](#step-2-deploy-aws-network-firewall-in-the-inspection-vpc)
   - [Step 3: Create the Transit Gateway](#step-3-create-the-transit-gateway)
   - [Step 4: Attach VPCs to the Transit Gateway](#step-4-attach-vpcs-to-the-transit-gateway)
   - [Step 5: Create TGW Route Tables](#step-5-create-tgw-route-tables)
   - [Step 6: Configure Associations and Propagations](#step-6-configure-associations-and-propagations)
   - [Step 7: Add Static Routes for Inspection](#step-7-add-static-routes-for-inspection)
   - [Step 8: Configure VPC Route Tables](#step-8-configure-vpc-route-tables)
   - [Step 9: Test Traffic Inspection](#step-9-test-traffic-inspection)
6. [Egress Traffic Inspection (Internet-Bound)](#egress-traffic-inspection-internet-bound)
7. [Key Considerations](#key-considerations)
8. [Cleanup](#cleanup)

---

## Overview

In enterprise environments, security teams often require **all inter-VPC and egress traffic** to pass through a centralized firewall before reaching its destination. The combination of:

- **AWS Transit Gateway** — as the central routing hub
- **AWS Network Firewall** — as the inspection engine
- **VPC Endpoints (Gateway Load Balancer Endpoints or Firewall Endpoints)** — to redirect traffic transparently

...enables a **hub-and-spoke centralized inspection model**.

---

## Prerequisites

- An AWS account with IAM permissions for `ec2:*`, `network-firewall:*`
- Understanding of VPC networking, CIDR blocks, and route tables
- AWS CLI configured (optional, for CLI-based setup)

---

## Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                          AWS Region                                │
│                                                                    │
│  ┌─────────────┐    ┌─────────────┐    ┌────────────────────────┐ │
│  │  Spoke VPC A│    │  Spoke VPC B│    │     Egress/Inspection  │ │
│  │ 10.0.0.0/16 │    │ 10.1.0.0/16 │    │          VPC           │ │
│  └──────┬──────┘    └──────┬──────┘    │      10.2.0.0/16       │ │
│         │                  │           │  ┌──────────────────┐  │ │
│         │ TGW Attach        │ TGW Attach│  │ Network Firewall │  │ │
│         │                  │           │  │   Endpoint (vpce)│  │ │
│  ┌──────┴──────────────────┴───────────┤  └────────┬─────────┘  │ │
│  │           Transit Gateway           │           │            │ │
│  │                                     │◄──────────┘            │ │
│  │  ┌─────────────┐  ┌─────────────┐   │  ┌──────────────────┐  │ │
│  │  │  Spoke RT   │  │ Inspection  │   │  │   Internet GW    │  │ │
│  │  │  (default:  │  │     RT      │   │  └──────────────────┘  │ │
│  │  │  0/0→FW)    │  │             │   └────────────────────────┘ │
│  │  └─────────────┘  └─────────────┘                             │
│  └─────────────────────────────────────                           │
└────────────────────────────────────────────────────────────────────┘
```

---

## Traffic Flow

### East-West (Spoke-to-Spoke) Traffic

```
Spoke VPC A Instance
        │
        ▼
Spoke A VPC Route Table (0.0.0.0/0 → TGW)
        │
        ▼
Transit Gateway (Spoke Route Table → static route → Inspection VPC attachment)
        │
        ▼
Inspection VPC – TGW subnet route table (→ Firewall Endpoint)
        │
        ▼
AWS Network Firewall (inspects & allows/denies)
        │
        ▼
Inspection VPC – Firewall subnet route table (→ TGW)
        │
        ▼
Transit Gateway (Post-Inspection Route Table → Spoke B attachment)
        │
        ▼
Spoke VPC B Instance
```

### Egress (Internet-Bound) Traffic

```
Spoke VPC Instance → TGW → Inspection VPC → Firewall → Internet Gateway → Internet
```

---

## Step-by-Step Setup

### Step 1: Create VPCs

Create the following VPCs with **non-overlapping CIDRs**:

| VPC | CIDR | Purpose |
|---|---|---|
| Spoke VPC A | `10.0.0.0/16` | Workload VPC (e.g., app servers) |
| Spoke VPC B | `10.1.0.0/16` | Workload VPC (e.g., databases) |
| Inspection VPC | `10.2.0.0/16` | Hosts Network Firewall + Internet Gateway |

For the **Inspection VPC**, create the following subnets:

| Subnet | CIDR | Purpose |
|---|---|---|
| TGW Subnet | `10.2.0.0/28` | TGW attachment subnet |
| Firewall Subnet | `10.2.1.0/28` | Network Firewall Endpoint subnet |
| Public Subnet | `10.2.2.0/24` | Internet Gateway access (for egress) |

---

### Step 2: Deploy AWS Network Firewall in the Inspection VPC

1. Create a **Firewall Policy** with appropriate stateful/stateless rule groups.
   - See [Firewall Setup](./firewall-setup.md) for detailed instructions.

2. Navigate to **VPC > Network Firewall > Create Firewall**.
3. Deploy the firewall in the **Firewall Subnet** of the Inspection VPC.
4. Wait for status **Ready** and note the **Firewall Endpoint ID** (`vpce-xxxxxxxxxxxxxxxxx`).

> **CLI equivalent:**
> ```bash
> aws network-firewall create-firewall \
>   --firewall-name central-inspection-firewall \
>   --firewall-policy-arn arn:aws:network-firewall:us-east-1:123456789012:firewall-policy/my-policy \
>   --vpc-id vpc-inspection \
>   --subnet-mappings SubnetId=subnet-firewall
> ```

---

### Step 3: Create the Transit Gateway

1. Navigate to **VPC > Transit Gateways > Create Transit Gateway**.
2. Configure:

| Setting | Value |
|---|---|
| **Name** | `central-tgw` |
| **Default route table association** | Disabled |
| **Default route table propagation** | Disabled |

> **CLI equivalent:**
> ```bash
> aws ec2 create-transit-gateway \
>   --description "Centralized inspection TGW" \
>   --options DefaultRouteTableAssociation=disable,DefaultRouteTablePropagation=disable
> ```

---

### Step 4: Attach VPCs to the Transit Gateway

Attach all three VPCs to the Transit Gateway.

> **Important:** For the Inspection VPC, attach using the **TGW Subnet** (not the Firewall Subnet or Public Subnet).

> **CLI equivalent:**
> ```bash
> # Spoke A
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-spoke-a \
>   --subnet-ids subnet-spoke-a
>
> # Spoke B
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-spoke-b \
>   --subnet-ids subnet-spoke-b
>
> # Inspection VPC (use TGW subnet)
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-inspection \
>   --subnet-ids subnet-tgw-inspection
> ```

---

### Step 5: Create TGW Route Tables

Create two route tables:

| Route Table | Purpose |
|---|---|
| `spoke-rt` | Associated with Spoke VPC attachments; directs all traffic to Inspection VPC |
| `inspection-rt` | Associated with Inspection VPC attachment; directs return traffic to correct spoke |

> **CLI equivalent:**
> ```bash
> aws ec2 create-transit-gateway-route-table \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=spoke-rt}]'
>
> aws ec2 create-transit-gateway-route-table \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=inspection-rt}]'
> ```

---

### Step 6: Configure Associations and Propagations

**Associations:**

| Attachment | Route Table |
|---|---|
| Spoke A VPC | `spoke-rt` |
| Spoke B VPC | `spoke-rt` |
| Inspection VPC | `inspection-rt` |

**Propagations:**

| Propagate into Route Table | From Attachment |
|---|---|
| `spoke-rt` | *(none — all traffic uses a static default route to inspection)* |
| `inspection-rt` | Spoke A VPC, Spoke B VPC |

> **CLI equivalent:**
> ```bash
> # Associations
> aws ec2 associate-transit-gateway-route-table \
>   --transit-gateway-route-table-id tgw-rtb-spoke \
>   --transit-gateway-attachment-id tgw-attach-spoke-a
>
> aws ec2 associate-transit-gateway-route-table \
>   --transit-gateway-route-table-id tgw-rtb-spoke \
>   --transit-gateway-attachment-id tgw-attach-spoke-b
>
> aws ec2 associate-transit-gateway-route-table \
>   --transit-gateway-route-table-id tgw-rtb-inspection \
>   --transit-gateway-attachment-id tgw-attach-inspection
>
> # Propagations into inspection-rt
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-inspection \
>   --transit-gateway-attachment-id tgw-attach-spoke-a
>
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-inspection \
>   --transit-gateway-attachment-id tgw-attach-spoke-b
> ```

---

### Step 7: Add Static Routes for Inspection

Add a **static default route** in `spoke-rt` that sends all traffic to the Inspection VPC attachment. This forces all inter-VPC and internet-bound traffic through the firewall.

> **CLI equivalent:**
> ```bash
> # Route all spoke traffic to Inspection VPC
> aws ec2 create-transit-gateway-route \
>   --transit-gateway-route-table-id tgw-rtb-spoke \
>   --destination-cidr-block 0.0.0.0/0 \
>   --transit-gateway-attachment-id tgw-attach-inspection
> ```

---

### Step 8: Configure VPC Route Tables

#### Spoke VPC A and B Route Tables

Direct all traffic to the Transit Gateway:

| Destination | Target |
|---|---|
| `0.0.0.0/0` | Transit Gateway |

```bash
aws ec2 create-route \
  --route-table-id rtb-spoke-a \
  --destination-cidr-block 0.0.0.0/0 \
  --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx
```

#### Inspection VPC – TGW Subnet Route Table

Route traffic to the Firewall Endpoint:

| Destination | Target |
|---|---|
| `10.0.0.0/8` (all VPC CIDRs) | Firewall Endpoint (`vpce-xxxxxxxxxxxxxxxxx`) |
| `0.0.0.0/0` | Firewall Endpoint (`vpce-xxxxxxxxxxxxxxxxx`) |

```bash
aws ec2 create-route \
  --route-table-id rtb-tgw-subnet \
  --destination-cidr-block 0.0.0.0/0 \
  --vpc-endpoint-id vpce-xxxxxxxxxxxxxxxxx
```

#### Inspection VPC – Firewall Subnet Route Table

After inspection, route traffic back to TGW (for east-west) or to IGW (for internet):

| Destination | Target |
|---|---|
| `10.0.0.0/16` (Spoke A) | Transit Gateway |
| `10.1.0.0/16` (Spoke B) | Transit Gateway |
| `0.0.0.0/0` | Internet Gateway (for egress) |

```bash
# Return traffic to Spoke VPCs via TGW
aws ec2 create-route \
  --route-table-id rtb-firewall-subnet \
  --destination-cidr-block 10.0.0.0/16 \
  --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx

aws ec2 create-route \
  --route-table-id rtb-firewall-subnet \
  --destination-cidr-block 10.1.0.0/16 \
  --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx

# Internet-bound egress
aws ec2 create-route \
  --route-table-id rtb-firewall-subnet \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxxxxxxxxxxxxxxxx
```

---

### Step 9: Test Traffic Inspection

1. Launch EC2 instances in Spoke VPC A and Spoke VPC B.
2. Test connectivity between the two spokes:

```bash
# From Spoke A instance
ping 10.1.x.x    # Should succeed if firewall allows ICMP
curl http://example-malicious.com   # Should be blocked by firewall
```

3. Check **CloudWatch Logs** for Network Firewall alerts and flow logs to confirm traffic is passing through the firewall.

---

## Egress Traffic Inspection (Internet-Bound)

For internet-bound traffic from spoke VPCs:

1. All spoke traffic goes to TGW via `spoke-rt`
2. TGW forwards to Inspection VPC via static route
3. Firewall inspects outbound traffic
4. Approved traffic exits via the Internet Gateway in the Inspection VPC
5. Return traffic enters via the Inspection VPC IGW → Firewall → TGW → Spoke VPC

Ensure the Inspection VPC has an Internet Gateway and the Firewall Subnet route table includes a default route (`0.0.0.0/0`) to the Internet Gateway.

---

## Key Considerations

| Consideration | Details |
|---|---|
| **Asymmetric routing** | AWS Network Firewall requires symmetric routing — both directions of a flow must pass through the same firewall endpoint |
| **Multiple AZs** | Deploy a firewall endpoint in each AZ used by your spoke VPCs to avoid cross-AZ traffic costs and single-AZ failures |
| **TGW attachment subnets** | Use small dedicated subnets (`/28`) for TGW attachments in the Inspection VPC |
| **Return traffic** | The Inspection VPC firewall subnet route table must have explicit routes for all spoke CIDRs back to the TGW |
| **Blackhole routes** | For isolation between spoke VPCs, use blackhole routes in `inspection-rt` for spoke CIDRs that should not communicate |

---

## Cleanup

Remove resources in this order:

1. Remove custom routes from VPC route tables
2. Delete TGW routes, propagations, and associations
3. Delete TGW attachments
4. Delete TGW route tables
5. Delete the Transit Gateway
6. Delete the Network Firewall
7. Delete the Firewall Policy and Rule Groups

---

## Related Topics

- [AWS Transit Gateway – How to Setup](./transit-gateway.md)
- [AWS Transit Gateway with Multiple Routing Tables](./transit-gateway-multiple-routing-tables.md)
- [Firewall Setup](./firewall-setup.md)
- [VPC Endpoint](./vpc-endpoint.md)
