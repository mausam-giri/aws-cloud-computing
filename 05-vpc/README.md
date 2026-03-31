# 05 — VPC: Virtual Private Cloud

← [Back to Main Index](../README.md) | [← 04 S3](../04-s3/README.md)

Amazon VPC (Virtual Private Cloud) lets you launch AWS resources in a logically isolated virtual network that you define. It gives you complete control over your networking environment.

---

## 📚 Table of Contents

1. [What is VPC?](#1-what-is-vpc)
2. [Key Concepts](#2-key-concepts)
3. [Default VPC vs Custom VPC](#3-default-vpc-vs-custom-vpc)
4. [Create a Custom VPC](#4-create-a-custom-vpc)
5. [Create Subnets](#5-create-subnets)
6. [Set Up an Internet Gateway](#6-set-up-an-internet-gateway)
7. [Configure Route Tables](#7-configure-route-tables)
8. [Create Security Groups](#8-create-security-groups)
9. [Launch an EC2 Instance in Your VPC](#9-launch-an-ec2-instance-in-your-vpc)
10. [References](#references)

---

## 1. What is VPC?

A VPC is your own private section of the AWS cloud. Think of it as a virtual data center — you define the IP address space, subnets, routing, and security rules.

**Real-world use cases:**
- Isolating production and development environments
- Building multi-tier applications (web, app, database layers)
- Connecting your on-premises network to AWS
- Restricting access to sensitive databases

---

## 2. Key Concepts

| Term | Description |
|------|-------------|
| **VPC** | An isolated virtual network in AWS |
| **Subnet** | A range of IP addresses within a VPC (can be public or private) |
| **Internet Gateway (IGW)** | Allows communication between your VPC and the internet |
| **Route Table** | Rules that determine where network traffic is directed |
| **NAT Gateway** | Allows instances in private subnets to access the internet (outbound only) |
| **Security Group** | Instance-level virtual firewall (stateful) |
| **Network ACL** | Subnet-level firewall (stateless) |
| **CIDR Block** | IP address range notation (e.g., `10.0.0.0/16`) |
| **Availability Zone (AZ)** | A physically separate data center within a region |

---

## 3. Default VPC vs Custom VPC

AWS automatically creates a **default VPC** in every region. For most beginner exercises, you can use it. However, for real applications, you should create a custom VPC.

| | Default VPC | Custom VPC |
|--|-------------|------------|
| Created automatically | ✅ Yes | ❌ No — you create it |
| Public subnets | ✅ All subnets are public | 🔧 You decide |
| Internet access | ✅ Ready to use | 🔧 You configure it |
| Best for | Quick testing, learning | Production workloads |

> ⚠️ **Easy to miss:** If you accidentally delete the default VPC, you can recreate it from the VPC dashboard by clicking **"Actions"** → **"Create default VPC"**. However, you should avoid modifying the default VPC for production use.

---

## 4. Create a Custom VPC

**Time:** ~5 minutes

1. In the AWS Console, search for **"VPC"** and open the VPC Dashboard.
2. Click **"Create VPC"**.
3. Select **"VPC only"** (not "VPC and more" — we'll set things up manually to learn each component).
4. Enter a **Name** (e.g., `my-custom-vpc`).
5. **IPv4 CIDR block:** Enter `10.0.0.0/16` (this gives you 65,536 IP addresses).

   > ⚠️ **Easy to miss:** The CIDR block you choose here cannot be changed after the VPC is created. `/16` is a common choice for a VPC as it allows many subnets.

6. Leave IPv6 as **"No IPv6 CIDR block"** for now.
7. **Tenancy:** Select **"Default"** (dedicated tenancy costs significantly more).
8. Click **"Create VPC"**.

---

## 5. Create Subnets

Subnets divide your VPC's IP range into smaller blocks. Create at least one **public** and one **private** subnet.

**Create a Public Subnet:**

1. In the VPC Dashboard sidebar, click **"Subnets"** → **"Create subnet"**.
2. Select your **VPC** from the dropdown.
3. **Subnet name:** `public-subnet-1`
4. **Availability Zone:** Choose one (e.g., `us-east-1a`).
5. **IPv4 CIDR block:** Enter `10.0.1.0/24` (256 IP addresses).
6. Click **"Add new subnet"** to also create a private subnet.
7. **Subnet name:** `private-subnet-1`
8. **Availability Zone:** Choose the same (or different) AZ.
9. **IPv4 CIDR block:** Enter `10.0.2.0/24`.
10. Click **"Create subnet"**.

   > ⚠️ **Easy to miss:** Creating a subnet in a VPC doesn't make it "public" or "private" automatically. A subnet is public only if it's connected to an **Internet Gateway** via a route table. We do that in the next steps.

---

## 6. Set Up an Internet Gateway

An Internet Gateway enables internet traffic to reach your VPC.

1. In the sidebar, click **"Internet gateways"** → **"Create internet gateway"**.
2. Enter a **Name** (e.g., `my-igw`).
3. Click **"Create internet gateway"**.
4. On the next page, click **"Attach to a VPC"**.
5. Select your VPC and click **"Attach internet gateway"**.

   > ⚠️ **Easy to miss:** An Internet Gateway must be **attached to your VPC** before it works. Creating it alone has no effect — the attachment step is critical.

> ⚠️ **Easy to miss:** Each VPC can only have **one Internet Gateway** attached at a time.

---

## 7. Configure Route Tables

Route tables control how traffic flows within and outside your VPC.

**Create a public route table:**

1. In the sidebar, click **"Route tables"** → **"Create route table"**.
2. Enter a **Name** (e.g., `public-route-table`).
3. Select your VPC and click **"Create route table"**.
4. Select the new route table → click the **"Routes"** tab → **"Edit routes"**.
5. Click **"Add route"**:
   - **Destination:** `0.0.0.0/0`
   - **Target:** Select **"Internet Gateway"** → choose your IGW
6. Click **"Save changes"**.
7. Go to the **"Subnet associations"** tab → **"Edit subnet associations"**.
8. Check **`public-subnet-1`** and click **"Save associations"**.

   > ⚠️ **Easy to miss:** Without associating the route table to a subnet, the routing rules have no effect. The subnet will continue using the **main route table** (which has no internet route) unless you explicitly associate it.

---

## 8. Create Security Groups

Security groups control traffic at the instance level.

1. In the sidebar, click **"Security groups"** → **"Create security group"**.
2. Enter a **Name** (e.g., `web-sg`) and description.
3. Select your VPC.
4. Add **Inbound rules:**

   | Type | Protocol | Port | Source |
   |------|----------|------|--------|
   | HTTP | TCP | 80 | `0.0.0.0/0` |
   | HTTPS | TCP | 443 | `0.0.0.0/0` |
   | SSH | TCP | 22 | My IP |

5. Leave **Outbound rules** as default (allow all outbound traffic).
6. Click **"Create security group"**.

---

## 9. Launch an EC2 Instance in Your VPC

1. Go to EC2 → **"Launch instance"**.
2. In the **"Network settings"** section, click **"Edit"**.
3. **VPC:** Select `my-custom-vpc`.
4. **Subnet:** Select `public-subnet-1`.
5. **Auto-assign public IP:** Set to **"Enable"**.

   > ⚠️ **Easy to miss:** Without enabling **"Auto-assign public IP"**, your instance in the public subnet will not have a public IP and won't be reachable from the internet.

6. **Security group:** Select the `web-sg` you created.
7. Complete the remaining launch steps and click **"Launch instance"**.

---

## References

### 📖 AWS Documentation
- [What is Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [VPCs and subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
- [Internet gateways](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [Route tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

### 🎬 YouTube Videos
- [AWS VPC Beginner Tutorial - TechWorld with Nana](https://www.youtube.com/watch?v=g2JOHLHh4rI)
- [AWS VPC Tutorial Step by Step](https://www.youtube.com/watch?v=fpxDGU2KdkA)
- [AWS VPC Full Course - freeCodeCamp](https://www.youtube.com/watch?v=2doSoMN2xvI)

---

**← Previous:** [04 — S3](../04-s3/README.md)  
**Next →** [06 — RDS: Relational Database Service](../06-rds/README.md)
