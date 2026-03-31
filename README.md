# ☁️ Cloud Computing with AWS — Beginner's Guide

A beginner-friendly, step-by-step guide to learning **Amazon Web Services (AWS)** — from account creation to deploying real cloud applications.

> ⚠️ **Easy-to-miss steps are marked throughout with this warning icon so you never get stuck.**

---

## 📁 Directory Structure

```
aws-cloud-computing/
├── README.md                              ← You are here (Start here!)
├── agent.rules.md                         ← Agent guiding principles and standards
├── 01-getting-started/
│   └── README.md                          ← AWS account setup & console walkthrough
├── 02-iam/
│   └── README.md                          ← Identity & Access Management (users, roles, policies)
├── 03-ec2/
│   └── README.md                          ← Elastic Compute Cloud (virtual servers)
├── 04-s3/
│   └── README.md                          ← Simple Storage Service (object storage)
├── 05-vpc/
│   └── README.md                          ← Virtual Private Cloud (networking)
├── 06-rds/
│   └── README.md                          ← Relational Database Service (managed databases)
├── 07-lambda/
│   └── README.md                          ← AWS Lambda (serverless functions)
├── 08-advanced-networking/
│   ├── README.md                          ← Advanced networking index
│   ├── transit-gateway.md                 ← Transit Gateway setup
│   ├── transit-gateway-multiple-routing-tables.md  ← TGW with multiple route tables
│   ├── transit-gateway-firewall-endpoint.md        ← TGW + Network Firewall integration
│   ├── firewall-setup.md                  ← AWS Network Firewall setup
│   └── vpc-endpoint.md                    ← VPC Endpoints (Gateway, Interface, GWLB)
└── 09-eks/
    └── README.md                          ← Elastic Kubernetes Service (EKS)
```

---

## 📚 Table of Contents

| # | Topic | Description |
|---|-------|-------------|
| 1 | [Getting Started](./01-getting-started/README.md) | Create your AWS account, set up billing alerts, and navigate the console |
| 2 | [IAM](./02-iam/README.md) | Create users, groups, roles, and policies to control access |
| 3 | [EC2](./03-ec2/README.md) | Launch, connect to, and manage virtual servers |
| 4 | [S3](./04-s3/README.md) | Create buckets, upload files, and host static websites |
| 5 | [VPC](./05-vpc/README.md) | Build isolated networks with subnets, routing, and security groups |
| 6 | [RDS](./06-rds/README.md) | Deploy and connect to managed relational databases |
| 7 | [Lambda](./07-lambda/README.md) | Write and deploy serverless functions triggered by events |
| 8 | [Advanced Networking](./08-advanced-networking/README.md) | Transit Gateway, Network Firewall, and VPC Endpoints |
| 9 | [EKS](./09-eks/README.md) | Elastic Kubernetes Service on AWS |

---

## 🔗 Official AWS Resources

- 📖 [AWS Documentation](https://docs.aws.amazon.com/)
- 🆓 [AWS Free Tier](https://aws.amazon.com/free/)
- 🎓 [AWS Skill Builder (Free Training)](https://skillbuilder.aws/)
- 📰 [AWS Getting Started Resource Center](https://aws.amazon.com/getting-started/)

---

## 🎬 Recommended YouTube Channels

| Channel | Description |
|---------|-------------|
| [AWS Official YouTube](https://www.youtube.com/@amazonwebservices) | Official tutorials and re:Invent talks |
| [freeCodeCamp.org](https://www.youtube.com/@freecodecamp) | Full AWS courses for beginners |
| [TechWorld with Nana](https://www.youtube.com/@TechWorldwithNana) | Practical DevOps and cloud tutorials |
| [Traversy Media](https://www.youtube.com/@TraversyMedia) | Project-based AWS walkthroughs |

---

*Contributions are welcome! Feel free to open issues or pull requests.*

---

## 🌐 Advanced Networking Guides

| Topic | Description |
|---|---|
| [Transit Gateway – How to Setup](./08-advanced-networking/transit-gateway.md) | Step-by-step guide to setting up an AWS Transit Gateway |
| [Transit Gateway with Multiple Routing Tables](./08-advanced-networking/transit-gateway-multiple-routing-tables.md) | Advanced TGW configuration with multiple route tables for traffic segmentation |
| [Firewall Setup](./08-advanced-networking/firewall-setup.md) | Setting up and configuring AWS Network Firewall |
| [Transit Gateway and Firewall with Endpoint](./08-advanced-networking/transit-gateway-firewall-endpoint.md) | Centralized traffic inspection using Transit Gateway and Network Firewall |
| [VPC Endpoint](./08-advanced-networking/vpc-endpoint.md) | Private connectivity to AWS services using VPC Endpoints |
