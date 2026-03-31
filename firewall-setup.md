# AWS Network Firewall Setup

AWS Network Firewall is a managed, stateful network firewall and intrusion detection/prevention service (IDS/IPS) that provides fine-grained control over network traffic flowing in and out of your VPCs.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Architecture](#architecture)
4. [Step-by-Step Setup](#step-by-step-setup)
   - [Step 1: Create a Firewall Policy](#step-1-create-a-firewall-policy)
   - [Step 2: Create Rule Groups](#step-2-create-rule-groups)
   - [Step 3: Deploy the Network Firewall](#step-3-deploy-the-network-firewall)
   - [Step 4: Configure Route Tables for Firewall Traffic](#step-4-configure-route-tables-for-firewall-traffic)
   - [Step 5: Configure Logging](#step-5-configure-logging)
   - [Step 6: Test the Firewall](#step-6-test-the-firewall)
5. [Rule Group Types](#rule-group-types)
6. [Common Rule Examples](#common-rule-examples)
7. [Pricing Considerations](#pricing-considerations)
8. [Cleanup](#cleanup)

---

## Overview

**AWS Network Firewall** is deployed inside a VPC and uses **Firewall Endpoints** to intercept traffic. It supports:

- **Stateful inspection:** Tracks connection state and applies rules to complete sessions
- **Stateless inspection:** Applies rules to individual packets for high-throughput filtering
- **Intrusion Prevention (IPS):** Uses Suricata-compatible rules to detect and block threats
- **Domain name filtering:** Block or allow traffic based on domain names (FQDN)

---

## Prerequisites

- An AWS account with IAM permissions for `network-firewall:*`, `ec2:*`, `logs:*`
- A VPC with at least one public and one private subnet
- An Internet Gateway attached to the VPC (for internet-facing scenarios)

---

## Architecture

```
Internet
    │
    ▼
┌──────────────────────────────────────────┐
│                   VPC                    │
│                                          │
│  ┌─────────────────────────────────────┐ │
│  │    Public Subnet (Firewall Subnet)  │ │
│  │   ┌──────────────────────────────┐  │ │
│  │   │  AWS Network Firewall        │  │ │
│  │   │  Endpoint (VPC Endpoint)     │  │ │
│  │   └──────────────────────────────┘  │ │
│  └─────────────────────────────────────┘ │
│                     │                    │
│  ┌─────────────────────────────────────┐ │
│  │    Private Subnet (Workload)        │ │
│  │   ┌──────────────────────────────┐  │ │
│  │   │         EC2 Instances        │  │ │
│  │   └──────────────────────────────┘  │ │
│  └─────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

Traffic flow: Internet → IGW → **Firewall Endpoint** (inspects) → Private subnet instances

---

## Step-by-Step Setup

### Step 1: Create a Firewall Policy

A **Firewall Policy** defines what rule groups to apply and the default actions for traffic.

1. Navigate to **VPC > Network Firewall > Firewall Policies > Create Firewall Policy**.
2. Configure:

| Setting | Value |
|---|---|
| **Name** | `my-firewall-policy` |
| **Stateless default actions** | Forward to stateful rules |
| **Stateless fragment default actions** | Forward to stateful rules |

3. Optionally add existing rule groups.
4. Click **Create Firewall Policy**.

> **CLI equivalent:**
> ```bash
> aws network-firewall create-firewall-policy \
>   --firewall-policy-name my-firewall-policy \
>   --firewall-policy '{
>     "StatelessDefaultActions": ["aws:forward_to_sfe"],
>     "StatelessFragmentDefaultActions": ["aws:forward_to_sfe"]
>   }'
> ```

---

### Step 2: Create Rule Groups

Rule groups define the actual traffic inspection rules.

#### Stateless Rule Group (example: block port 23/Telnet)

1. Navigate to **Network Firewall > Rule Groups > Create Rule Group**.
2. Select **Stateless**.
3. Configure:
   - **Name:** `block-telnet-stateless`
   - **Capacity:** `100`
4. Add a rule:
   - **Protocol:** TCP
   - **Destination Port:** 23
   - **Action:** Drop

> **CLI equivalent:**
> ```bash
> aws network-firewall create-rule-group \
>   --rule-group-name block-telnet-stateless \
>   --type STATELESS \
>   --capacity 100 \
>   --rule-group '{
>     "RulesSource": {
>       "StatelessRulesAndCustomActions": {
>         "StatelessRules": [{
>           "RuleDefinition": {
>             "MatchAttributes": {
>               "Protocols": [6],
>               "DestinationPorts": [{"FromPort": 23, "ToPort": 23}]
>             },
>             "Actions": ["aws:drop"]
>           },
>           "Priority": 1
>         }]
>       }
>     }
>   }'
> ```

#### Stateful Rule Group (example: block malicious domain)

1. Navigate to **Network Firewall > Rule Groups > Create Rule Group**.
2. Select **Stateful**.
3. Select **Domain List** rule type.
4. Configure:
   - **Name:** `block-bad-domains`
   - **Capacity:** `100`
   - **Action:** Deny
   - **Domains:** `example-malicious.com`, `badsite.net`
   - **Protocol:** HTTP, HTTPS

> **CLI equivalent (Suricata rules):**
> ```bash
> aws network-firewall create-rule-group \
>   --rule-group-name block-bad-domains \
>   --type STATEFUL \
>   --capacity 100 \
>   --rule-group '{
>     "RulesSource": {
>       "RulesSourceList": {
>         "Targets": ["example-malicious.com", "badsite.net"],
>         "TargetTypes": ["HTTP_HOST", "TLS_SNI"],
>         "GeneratedRulesType": "DENYLIST"
>       }
>     }
>   }'
> ```

#### Add Rule Groups to Firewall Policy

1. Go to **Firewall Policies** and select `my-firewall-policy`.
2. Click **Edit** and add your rule groups under **Stateless rule groups** or **Stateful rule groups**.

---

### Step 3: Deploy the Network Firewall

1. Navigate to **VPC > Network Firewall > Create Firewall**.
2. Configure:

| Setting | Value |
|---|---|
| **Name** | `my-network-firewall` |
| **VPC** | Select your VPC |
| **Availability Zone** | Select an AZ |
| **Subnet** | Select the **Firewall Subnet** (a dedicated subnet) |
| **Firewall Policy** | `my-firewall-policy` |

3. Click **Create Firewall**.
4. Wait for the firewall status to show **Ready** (can take 5–10 minutes).
5. Note the **Firewall Endpoint ID** (a VPC Endpoint ID starting with `vpce-`).

> **CLI equivalent:**
> ```bash
> aws network-firewall create-firewall \
>   --firewall-name my-network-firewall \
>   --firewall-policy-arn arn:aws:network-firewall:us-east-1:123456789012:firewall-policy/my-firewall-policy \
>   --vpc-id vpc-xxxxxxxxxxxxxxxxx \
>   --subnet-mappings SubnetId=subnet-xxxxxxxxxxxxxxxxx
> ```

> **Get the Firewall Endpoint:**
> ```bash
> aws network-firewall describe-firewall \
>   --firewall-name my-network-firewall \
>   --query 'FirewallStatus.SyncStates'
> ```

---

### Step 4: Configure Route Tables for Firewall Traffic

Traffic must be **redirected through the Firewall Endpoint**. This is done by modifying VPC route tables.

**Traffic flow for internet-bound traffic:**

```
Private Subnet → Private Route Table → Firewall Endpoint → IGW Route Table → Internet
Internet → IGW Ingress Route Table → Firewall Endpoint → Private Subnet
```

#### Private Subnet Route Table

Route all outbound traffic to the Firewall Endpoint:

| Destination | Target |
|---|---|
| `0.0.0.0/0` | Firewall Endpoint (`vpce-xxxxxxxxxxxxxxxxx`) |

#### Firewall Subnet Route Table

Route traffic to the Internet Gateway after inspection:

| Destination | Target |
|---|---|
| `0.0.0.0/0` | Internet Gateway (`igw-xxxxxxxxxxxxxxxxx`) |

#### IGW Ingress Route Table (Edge Association)

Route inbound traffic from the internet to the Firewall Endpoint:

1. Go to **Route Tables** and associate the route table with the Internet Gateway (Edge Association).
2. Add a route for the private subnet CIDR:

| Destination | Target |
|---|---|
| `10.0.1.0/24` (Private Subnet CIDR) | Firewall Endpoint (`vpce-xxxxxxxxxxxxxxxxx`) |

> **CLI equivalent:**
> ```bash
> # Private subnet → Firewall Endpoint
> aws ec2 create-route \
>   --route-table-id rtb-private \
>   --destination-cidr-block 0.0.0.0/0 \
>   --vpc-endpoint-id vpce-xxxxxxxxxxxxxxxxx
>
> # Firewall subnet → IGW
> aws ec2 create-route \
>   --route-table-id rtb-firewall \
>   --destination-cidr-block 0.0.0.0/0 \
>   --gateway-id igw-xxxxxxxxxxxxxxxxx
>
> # IGW ingress → Firewall Endpoint
> aws ec2 create-route \
>   --route-table-id rtb-ingress \
>   --destination-cidr-block 10.0.1.0/24 \
>   --vpc-endpoint-id vpce-xxxxxxxxxxxxxxxxx
> ```

---

### Step 5: Configure Logging

1. Navigate to **Network Firewall > Firewalls** and select your firewall.
2. Go to **Logging** tab > **Edit logging configuration**.
3. Configure log destinations:

| Log Type | Destination |
|---|---|
| **Alert logs** | CloudWatch Logs (e.g., `/aws/network-firewall/alert`) |
| **Flow logs** | Amazon S3 bucket or CloudWatch Logs |

> **CLI equivalent:**
> ```bash
> aws network-firewall update-logging-configuration \
>   --firewall-name my-network-firewall \
>   --logging-configuration '{
>     "LogDestinationConfigs": [
>       {
>         "LogType": "ALERT",
>         "LogDestinationType": "CloudWatchLogs",
>         "LogDestination": {"logGroup": "/aws/network-firewall/alert"}
>       },
>       {
>         "LogType": "FLOW",
>         "LogDestinationType": "CloudWatchLogs",
>         "LogDestination": {"logGroup": "/aws/network-firewall/flow"}
>       }
>     ]
>   }'
> ```

---

### Step 6: Test the Firewall

1. Launch an EC2 instance in the **private subnet**.
2. Try to connect to a domain that should be blocked:

```bash
# This should be blocked
curl http://example-malicious.com

# This should succeed (if allowed)
curl https://www.amazon.com
```

3. Check **CloudWatch Logs** for alert and flow log entries.

---

## Rule Group Types

| Type | Description | Use Case |
|---|---|---|
| **Stateless** | Inspects individual packets; no state tracking | High-performance basic filtering (allow/drop/forward) |
| **Stateful – 5-tuple** | Matches on protocol, source/dest IP, source/dest port | Simple connection-level rules |
| **Stateful – Domain List** | Allow or deny based on domain names | Web filtering, egress control |
| **Stateful – Suricata** | Full Suricata IDS/IPS rules | Advanced threat detection, custom rules |

---

## Common Rule Examples

### Allow only HTTPS and DNS (Suricata format)

```
pass tcp any any -> any 443 (msg:"Allow HTTPS"; sid:1; rev:1;)
pass udp any any -> any 53 (msg:"Allow DNS"; sid:2; rev:1;)
drop tcp any any -> any any (msg:"Drop all other TCP"; sid:3; rev:1;)
```

### Block a specific IP

```
drop ip 203.0.113.10 any -> any any (msg:"Block known bad IP"; sid:100; rev:1;)
```

---

## Pricing Considerations

- **Per firewall endpoint per hour:** Charged for each deployment
- **Per GB data processed:** Charged per GB inspected
- Logging to S3 or CloudWatch may incur additional storage/ingestion costs

Refer to [AWS Network Firewall Pricing](https://aws.amazon.com/network-firewall/pricing/) for current rates.

---

## Cleanup

Remove resources in this order:

1. Update route tables to remove Firewall Endpoint routes
2. Delete the Network Firewall
3. Delete the Firewall Policy
4. Delete Rule Groups

```bash
# Delete firewall
aws network-firewall delete-firewall \
  --firewall-name my-network-firewall

# Delete firewall policy
aws network-firewall delete-firewall-policy \
  --firewall-policy-name my-firewall-policy

# Delete rule group
aws network-firewall delete-rule-group \
  --rule-group-name block-telnet-stateless \
  --type STATELESS
```

---

## Related Topics

- [AWS Transit Gateway – How to Setup](./transit-gateway.md)
- [Transit Gateway and Firewall with Endpoint](./transit-gateway-firewall-endpoint.md)
- [VPC Endpoint](./vpc-endpoint.md)
