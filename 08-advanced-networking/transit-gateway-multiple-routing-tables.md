# AWS Transit Gateway with Multiple Routing Tables

> 📺 **Reference video:** [AWS Transit Gateway with Multiple Routing Tables](https://youtu.be/iKNWFxOEVDI)

This guide explains how to use **multiple Transit Gateway (TGW) route tables** to implement advanced traffic segmentation — for example, to isolate shared services, development, and production environments or to enforce traffic through an inspection VPC.

---

## Table of Contents

1. [Overview](#overview)
2. [Use Cases](#use-cases)
3. [Architecture](#architecture)
4. [Key Concepts: Association vs Propagation](#key-concepts-association-vs-propagation)
5. [Step-by-Step Setup](#step-by-step-setup)
   - [Step 1: Create the Transit Gateway](#step-1-create-the-transit-gateway)
   - [Step 2: Attach VPCs to the Transit Gateway](#step-2-attach-vpcs-to-the-transit-gateway)
   - [Step 3: Create Multiple TGW Route Tables](#step-3-create-multiple-tgw-route-tables)
   - [Step 4: Configure Associations](#step-4-configure-associations)
   - [Step 5: Configure Propagations](#step-5-configure-propagations)
   - [Step 6: Add Static Routes (if needed)](#step-6-add-static-routes-if-needed)
   - [Step 7: Update VPC Route Tables](#step-7-update-vpc-route-tables)
   - [Step 8: Test Connectivity](#step-8-test-connectivity)
6. [Common Patterns](#common-patterns)
7. [Cleanup](#cleanup)

---

## Overview

By default, a Transit Gateway uses a **single route table** for all attachments. While this works for simple hub-and-spoke connectivity, more complex environments require **traffic segmentation**:

- Prevent spoke VPCs from communicating with each other (isolation)
- Allow all spokes to reach a shared services VPC
- Route traffic through a security/inspection VPC before reaching the destination

Multiple TGW route tables make this possible through granular **association** and **propagation** control.

---

## Use Cases

| Scenario | Solution |
|---|---|
| Dev/Test VPCs must not reach Production VPCs | Separate route tables per environment; no cross-propagation |
| All VPCs must access a shared DNS/logging VPC | Propagate shared services routes into all route tables |
| All traffic must be inspected by a firewall VPC | Route traffic through a security VPC using static routes |
| Segment partner VPC from internal VPCs | Dedicated route table for partner with limited propagation |

---

## Architecture

This example uses three environments:

```
┌──────────────────────────────────────────────────────┐
│                  Transit Gateway                     │
│                                                      │
│  ┌─────────────────────┐  ┌────────────────────────┐ │
│  │  Prod Route Table   │  │  Dev Route Table       │ │
│  │  - Prod VPC CIDR    │  │  - Dev VPC CIDR        │ │
│  │  - Shared Svc CIDR  │  │  - Shared Svc CIDR     │ │
│  └──────────┬──────────┘  └────────────┬───────────┘ │
│             │ associated               │ associated   │
└─────────────┼───────────────────────────┼─────────────┘
              │                           │
      ┌───────┘                   ┌───────┘
      │                           │
 ┌────┴──────┐              ┌─────┴─────┐
 │ Prod VPC  │              │  Dev VPC  │
 │10.0.0.0/16│              │10.1.0.0/16│
 └───────────┘              └───────────┘
                   ┌──────────────────────┐
                   │   Shared Svc VPC     │
                   │     10.2.0.0/16      │
                   │ (propagated to both) │
                   └──────────────────────┘
```

---

## Key Concepts: Association vs Propagation

| Concept | Description |
|---|---|
| **Association** | Links an attachment to a route table. Determines **which route table** is used to route traffic *coming from* that attachment |
| **Propagation** | Automatically adds the attachment's CIDR as a route *into* a route table. Controls where the attachment's CIDR is reachable |

**Rule:** Each attachment can be **associated with only one** route table, but can **propagate its routes into multiple** route tables.

---

## Step-by-Step Setup

### Step 1: Create the Transit Gateway

1. Navigate to **VPC > Transit Gateways > Create Transit Gateway**.
2. Configure:
   - **Name:** `my-tgw`
   - **Default route table association:** **Disabled** *(important — we will manage associations manually)*
   - **Default route table propagation:** **Disabled** *(important — we will manage propagations manually)*
3. Click **Create Transit Gateway** and wait for `available` state.

> **CLI equivalent:**
> ```bash
> aws ec2 create-transit-gateway \
>   --description "TGW with multiple route tables" \
>   --options DefaultRouteTableAssociation=disable,DefaultRouteTablePropagation=disable
> ```

---

### Step 2: Attach VPCs to the Transit Gateway

Create a Transit Gateway VPC attachment for each VPC (Prod, Dev, and Shared Services).

1. Navigate to **VPC > Transit Gateway Attachments > Create Transit Gateway Attachment**.
2. Select your Transit Gateway and the VPC, then choose subnets.
3. Repeat for each VPC.

> **CLI equivalent:**
> ```bash
> # Prod VPC
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-prod-xxxxxxxx \
>   --subnet-ids subnet-prod-xxxxxxxx
>
> # Dev VPC
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-dev-xxxxxxxx \
>   --subnet-ids subnet-dev-xxxxxxxx
>
> # Shared Services VPC
> aws ec2 create-transit-gateway-vpc-attachment \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --vpc-id vpc-shared-xxxxxxxx \
>   --subnet-ids subnet-shared-xxxxxxxx
> ```

---

### Step 3: Create Multiple TGW Route Tables

Create a separate route table for each segment.

1. Navigate to **VPC > Transit Gateway Route Tables > Create Transit Gateway Route Table**.
2. Select your Transit Gateway.
3. Add a **Name tag** to identify the route table (e.g., `prod-rt`, `dev-rt`, `shared-rt`).
4. Repeat to create route tables for Dev and Shared Services.

> **CLI equivalent:**
> ```bash
> # Production route table
> aws ec2 create-transit-gateway-route-table \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=prod-rt}]'
>
> # Development route table
> aws ec2 create-transit-gateway-route-table \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=dev-rt}]'
>
> # Shared Services route table
> aws ec2 create-transit-gateway-route-table \
>   --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx \
>   --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=shared-rt}]'
> ```

---

### Step 4: Configure Associations

Associate each VPC attachment with its corresponding route table. This controls which route table is consulted when traffic **leaves** that VPC.

| VPC Attachment | Associated Route Table |
|---|---|
| Prod VPC attachment | `prod-rt` |
| Dev VPC attachment | `dev-rt` |
| Shared Services attachment | `shared-rt` |

1. Go to **Transit Gateway Route Tables**.
2. Select `prod-rt` > **Associations** tab > **Create association**.
3. Select the Prod VPC attachment.
4. Repeat for `dev-rt` → Dev VPC attachment and `shared-rt` → Shared Services attachment.

> **CLI equivalent:**
> ```bash
> # Associate Prod VPC with prod-rt
> aws ec2 associate-transit-gateway-route-table \
>   --transit-gateway-route-table-id tgw-rtb-prod \
>   --transit-gateway-attachment-id tgw-attach-prod
>
> # Associate Dev VPC with dev-rt
> aws ec2 associate-transit-gateway-route-table \
>   --transit-gateway-route-table-id tgw-rtb-dev \
>   --transit-gateway-attachment-id tgw-attach-dev
>
> # Associate Shared Services VPC with shared-rt
> aws ec2 associate-transit-gateway-route-table \
>   --transit-gateway-route-table-id tgw-rtb-shared \
>   --transit-gateway-attachment-id tgw-attach-shared
> ```

---

### Step 5: Configure Propagations

Propagation adds a VPC's CIDR as a **route entry** in a route table. This determines which VPCs can **receive** traffic from the associated VPC.

**Desired routing behavior:**

| Route Table | Propagate From | Result |
|---|---|---|
| `prod-rt` | Prod VPC + Shared Services VPC | Prod can reach itself and shared services only |
| `dev-rt` | Dev VPC + Shared Services VPC | Dev can reach itself and shared services only |
| `shared-rt` | Prod VPC + Dev VPC + Shared Services VPC | Shared services can reach all VPCs |

1. Go to **Transit Gateway Route Tables** > select `prod-rt`.
2. Click **Propagations** tab > **Enable propagation**.
3. Select the **Prod VPC attachment** and confirm.
4. Repeat to propagate **Shared Services VPC attachment** into `prod-rt`.
5. Repeat the process for `dev-rt` and `shared-rt`.

> **CLI equivalent:**
> ```bash
> # prod-rt: propagate Prod and Shared Services
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-prod \
>   --transit-gateway-attachment-id tgw-attach-prod
>
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-prod \
>   --transit-gateway-attachment-id tgw-attach-shared
>
> # dev-rt: propagate Dev and Shared Services
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-dev \
>   --transit-gateway-attachment-id tgw-attach-dev
>
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-dev \
>   --transit-gateway-attachment-id tgw-attach-shared
>
> # shared-rt: propagate all three
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-shared \
>   --transit-gateway-attachment-id tgw-attach-prod
>
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-shared \
>   --transit-gateway-attachment-id tgw-attach-dev
>
> aws ec2 enable-transit-gateway-route-table-propagation \
>   --transit-gateway-route-table-id tgw-rtb-shared \
>   --transit-gateway-attachment-id tgw-attach-shared
> ```

---

### Step 6: Add Static Routes (if needed)

If you need to route traffic through an inspection VPC (e.g., for firewall inspection), add static routes instead of relying solely on propagations.

> **CLI equivalent:**
> ```bash
> # Route all traffic from Prod through a firewall/inspection attachment
> aws ec2 create-transit-gateway-route \
>   --transit-gateway-route-table-id tgw-rtb-prod \
>   --destination-cidr-block 0.0.0.0/0 \
>   --transit-gateway-attachment-id tgw-attach-firewall
> ```

---

### Step 7: Update VPC Route Tables

Each VPC's subnet route table must route traffic destined for other VPCs toward the Transit Gateway.

```bash
# In Prod VPC: route to Dev and Shared Services via TGW
aws ec2 create-route \
  --route-table-id rtb-prod \
  --destination-cidr-block 10.1.0.0/16 \  # Dev VPC
  --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx

aws ec2 create-route \
  --route-table-id rtb-prod \
  --destination-cidr-block 10.2.0.0/16 \  # Shared Services VPC
  --transit-gateway-id tgw-xxxxxxxxxxxxxxxxx
```

> Repeat for Dev and Shared Services VPCs as appropriate.

---

### Step 8: Test Connectivity

| Test | Expected Result |
|---|---|
| Prod instance → Shared Services instance | ✅ Success |
| Dev instance → Shared Services instance | ✅ Success |
| Prod instance → Dev instance | ❌ Blocked (no route) |
| Dev instance → Prod instance | ❌ Blocked (no route) |
| Shared Services instance → Prod instance | ✅ Success |
| Shared Services instance → Dev instance | ✅ Success |

```bash
# From Prod instance
ping 10.2.x.x  # Should succeed (Shared Services)
ping 10.1.x.x  # Should fail (Dev - no route)
```

---

## Common Patterns

### Pattern 1: Isolated Spokes with Shared Services

All spoke VPCs can reach a central shared services VPC, but not each other.

```
Spoke Route Tables → propagate Shared Services CIDR only
Shared Services Route Table → propagate all Spoke CIDRs
```

### Pattern 2: Centralized Inspection (East-West)

Route all inter-VPC traffic through a security/firewall appliance.

```
All Spoke Route Tables → static default route → Inspection VPC attachment
Inspection VPC → static routes back to each spoke
Post-inspection Route Table → propagate all Spoke CIDRs
```

### Pattern 3: Environment Segmentation

Full isolation between Dev, Test, and Prod with shared access to services.

```
Dev RT → propagate Dev + Shared Services
Test RT → propagate Test + Shared Services
Prod RT → propagate Prod + Shared Services
Shared RT → propagate Dev + Test + Prod + Shared
```

---

## Cleanup

Remove resources in this order:

1. Disable propagations
2. Delete associations
3. Delete TGW route table entries
4. Delete Transit Gateway attachments
5. Delete Transit Gateway route tables
6. Delete Transit Gateway

```bash
# Example: disable propagation
aws ec2 disable-transit-gateway-route-table-propagation \
  --transit-gateway-route-table-id tgw-rtb-prod \
  --transit-gateway-attachment-id tgw-attach-prod

# Delete route table
aws ec2 delete-transit-gateway-route-table \
  --transit-gateway-route-table-id tgw-rtb-prod
```

---

## Related Topics

- [AWS Transit Gateway – How to Setup](./transit-gateway.md)
- [Transit Gateway and Firewall with Endpoint](./transit-gateway-firewall-endpoint.md)
- [Firewall Setup](./firewall-setup.md)
