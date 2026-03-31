# 06 — RDS: Relational Database Service

← [Back to Main Index](../README.md) | [← 05 VPC](../05-vpc/README.md)

Amazon RDS (Relational Database Service) makes it easy to set up, operate, and scale a relational database in the cloud. It handles routine tasks like backups, patching, and replication automatically.

---

## 📚 Table of Contents

1. [What is RDS?](#1-what-is-rds)
2. [Supported Database Engines](#2-supported-database-engines)
3. [Create a DB Subnet Group](#3-create-a-db-subnet-group)
4. [Launch an RDS Instance](#4-launch-an-rds-instance)
5. [Connect to Your RDS Database](#5-connect-to-your-rds-database)
6. [Create a Database and Table](#6-create-a-database-and-table)
7. [Enable Automated Backups](#7-enable-automated-backups)
8. [Delete an RDS Instance](#8-delete-an-rds-instance)
9. [References](#references)

---

## 1. What is RDS?

RDS provides managed relational databases. AWS handles the infrastructure (servers, storage, OS) — you focus on your database and data.

**Benefits over self-managed databases:**
- Automated backups and point-in-time recovery
- Automatic software patching
- Easy scaling (vertically and horizontally)
- Multi-AZ deployments for high availability
- Read replicas for performance

---

## 2. Supported Database Engines

| Engine | Description |
|--------|-------------|
| **Amazon Aurora** | AWS-optimized MySQL/PostgreSQL compatible DB |
| **MySQL** | Most popular open-source relational DB |
| **PostgreSQL** | Advanced open-source relational DB |
| **MariaDB** | Community-developed MySQL fork |
| **Oracle** | Enterprise database (license required) |
| **SQL Server** | Microsoft's relational database |

> ⚠️ **Easy to miss:** **Amazon Aurora** is not available in the Free Tier. For learning, use **MySQL** or **PostgreSQL** with a `db.t3.micro` instance type, which is Free Tier eligible.

---

## 3. Create a DB Subnet Group

A DB Subnet Group tells RDS which subnets to use for your database. You need at least **two subnets in different Availability Zones**.

1. In the AWS Console, search for **"RDS"** and open the RDS Dashboard.
2. In the left sidebar, click **"Subnet groups"** → **"Create DB subnet group"**.
3. Enter a **Name** (e.g., `my-db-subnet-group`) and description.
4. Select your **VPC**.
5. Under **"Add subnets"**, choose at least 2 different Availability Zones and add subnets from each.

   > ⚠️ **Easy to miss:** RDS requires at least **two subnets in two different Availability Zones** for a subnet group, even if you're not using Multi-AZ. If you only created subnets in one AZ, create a second subnet in a different AZ first (see [05 - VPC](../05-vpc/README.md)).

6. Click **"Create"**.

---

## 4. Launch an RDS Instance

**Time:** ~10 minutes (plus 5–10 minutes for the DB to become available)

1. In the RDS Dashboard, click **"Create database"**.
2. **Database creation method:** Select **"Standard create"** (gives you more control).
3. **Engine type:** Select **"MySQL"**.
4. **Edition:** Select **"MySQL Community"**.
5. **Version:** Keep the default (latest recommended version).
6. **Templates:** Select **"Free tier"**.

   > ⚠️ **Easy to miss:** Always select the **"Free tier"** template when learning. This pre-selects settings to keep costs at zero. Using "Production" or "Dev/Test" templates can select instance types that are NOT Free Tier eligible.

7. **DB instance identifier:** Enter a name (e.g., `my-first-db`).
8. **Master username:** Enter `admin` (or any username you'll remember).
9. **Master password:** Enter a strong password and write it down.

   > ⚠️ **Easy to miss:** Write down your **master username and password** now. You will need them every time you connect to the database. There is no "forgot password" — you would need to reset it through the RDS console if lost.

10. **DB instance class:** Ensure `db.t3.micro` is selected (Free Tier).
11. **Storage:** Keep `20 GiB` gp2 storage (Free Tier allows up to 20 GiB).
12. **Uncheck "Enable storage autoscaling"** to prevent unexpected charges.

    > ⚠️ **Easy to miss:** Storage autoscaling can increase your storage beyond the Free Tier limits and incur charges. Disable it when learning.

13. **Connectivity:**
    - Select your VPC
    - Select your DB subnet group
    - **Public access:** Select **"Yes"** if you want to connect from your local computer (less secure but easier for learning). Select **"No"** if only EC2 instances will access it.
    - Choose an existing security group or create a new one
14. Under **"Additional configuration"**, enter an **Initial database name** (e.g., `mydb`).

    > ⚠️ **Easy to miss:** If you skip the **"Initial database name"**, RDS creates the instance but doesn't create a database inside it. You'd then need to connect and run `CREATE DATABASE` manually.

15. Uncheck **"Enable automated backups"** for cost savings during learning (or keep it for practice).
16. Uncheck **"Enable Enhanced monitoring"** (costs extra).
17. Click **"Create database"**.

Wait for the **Status** to change to **"Available"** (usually 5–10 minutes).

---

## 5. Connect to Your RDS Database

### Get the Connection Endpoint

1. In the RDS Dashboard, click on your database name.
2. In the **"Connectivity & security"** tab, copy the **Endpoint** URL (e.g., `my-first-db.abc123.us-east-1.rds.amazonaws.com`).
3. Note the **Port** (default: `3306` for MySQL).

### Update Security Group

Your RDS security group must allow inbound traffic on the database port from your IP.

1. Click on the security group linked to your RDS instance.
2. Add an **Inbound rule:**
   - **Type:** MySQL/Aurora (or the appropriate type for your engine)
   - **Port:** 3306
   - **Source:** My IP (for local connection) or the EC2 security group ID (if connecting from EC2)

   > ⚠️ **Easy to miss:** If your RDS has "Public access: Yes" but the security group doesn't allow port 3306 from your IP, you still can't connect. Both conditions must be true.

### Connect Using MySQL CLI

```bash
# Install MySQL client (if not already installed)
# macOS:
brew install mysql-client

# Ubuntu/Debian:
sudo apt install mysql-client

# Connect to the database
mysql -h <endpoint> -P 3306 -u admin -p
# Enter your password when prompted
```

### Connect Using MySQL Workbench (GUI)

1. Download [MySQL Workbench](https://www.mysql.com/products/workbench/).
2. Click **"+"** to create a new connection.
3. Fill in:
   - **Hostname:** Your RDS endpoint
   - **Port:** 3306
   - **Username:** admin
4. Click **"Test Connection"** and enter your password.
5. Click **"OK"** → double-click the connection to open it.

---

## 6. Create a Database and Table

Once connected, run these SQL commands:

```sql
-- Use your database
USE mydb;

-- Create a sample table
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO users (name, email) VALUES
  ('Alice Smith', 'alice@example.com'),
  ('Bob Jones', 'bob@example.com');

-- Query the data
SELECT * FROM users;
```

---

## 7. Enable Automated Backups

1. In the RDS Dashboard, select your database.
2. Click **"Modify"**.
3. Under **"Additional configuration"** → **"Backup"**:
   - Set **Backup retention period** to `7 days`.
   - Choose a **Backup window** (or leave as "No preference").
4. Click **"Continue"** → **"Apply immediately"** → **"Modify DB instance"**.

> ⚠️ **Easy to miss:** Automated backups are stored in S3 and incur storage costs beyond the Free Tier. For learning, keep the retention period short (1–7 days) to minimize costs.

---

## 8. Delete an RDS Instance

When you're done learning, delete the instance to avoid ongoing charges.

1. In the RDS Dashboard, select your database.
2. Click **"Actions"** → **"Delete"**.
3. Uncheck **"Create final snapshot"** (since this is just for learning).
4. Check **"I acknowledge..."** and type `delete me` to confirm.
5. Click **"Delete"**.

   > ⚠️ **Easy to miss:** Deleting an RDS instance is **permanent and irreversible**. Any data not backed up will be lost. Also, a **final snapshot** (if created) incurs storage costs until you manually delete it from RDS → Snapshots.

---

## References

### 📖 AWS Documentation
- [What is Amazon RDS?](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [Getting started with Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.html)
- [Connecting to a DB instance running the MySQL database engine](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html)
- [Working with DB instance read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)

### 🎬 YouTube Videos
- [AWS RDS Tutorial For Beginners - TechWorld with Nana](https://www.youtube.com/watch?v=vw5EO5Jz8-8)
- [AWS RDS MySQL Setup Step by Step](https://www.youtube.com/watch?v=hAvF93BHmCc)
- [AWS RDS Full Course - freeCodeCamp](https://www.youtube.com/watch?v=GmN2RN4F8c8)

---

**← Previous:** [05 — VPC](../05-vpc/README.md)  
**Next →** [07 — Lambda: Serverless Functions](../07-lambda/README.md)
