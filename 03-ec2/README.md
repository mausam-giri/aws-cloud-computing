# 03 — EC2: Elastic Compute Cloud

← [Back to Main Index](../README.md) | [← 02 IAM](../02-iam/README.md)

EC2 (Elastic Compute Cloud) lets you rent virtual servers in the cloud. You can launch, stop, resize, and terminate instances on-demand.

---

## 📚 Table of Contents

1. [What is EC2?](#1-what-is-ec2)
2. [Key Concepts](#2-key-concepts)
3. [Launch Your First EC2 Instance](#3-launch-your-first-ec2-instance)
4. [Connect to Your EC2 Instance](#4-connect-to-your-ec2-instance)
5. [Install a Web Server (Optional)](#5-install-a-web-server-optional)
6. [Stop and Terminate an Instance](#6-stop-and-terminate-an-instance)
7. [Security Groups](#7-security-groups)
8. [References](#references)

---

## 1. What is EC2?

EC2 gives you **virtual machines (instances)** in AWS data centers. You choose the operating system, CPU, memory, and storage — and pay only for what you use.

**Real-world use cases:**
- Hosting web applications and APIs
- Running databases
- Batch data processing
- Development and testing environments

---

## 2. Key Concepts

| Term | Description |
|------|-------------|
| **Instance** | A running virtual machine |
| **AMI** (Amazon Machine Image) | A template (OS + software) used to launch instances |
| **Instance Type** | The hardware configuration (CPU, RAM, network) |
| **Key Pair** | SSH credentials used to connect to Linux instances |
| **Security Group** | A virtual firewall that controls inbound/outbound traffic |
| **Elastic IP** | A static public IP address you can attach to an instance |
| **EBS** (Elastic Block Store) | Persistent storage volumes attached to instances |

---

## 3. Launch Your First EC2 Instance

**Time:** ~10 minutes

1. In the AWS Console, search for **"EC2"** and open the EC2 Dashboard.
2. Click **"Launch instance"**.
3. **Name your instance** (e.g., `my-first-server`).
4. **Choose an AMI:** Select **"Amazon Linux 2023 AMI"** (Free Tier eligible — look for the orange "Free tier eligible" badge).

   > ⚠️ **Easy to miss:** Make sure the AMI you select shows **"Free tier eligible"** to avoid unexpected charges. Paid AMIs will show a per-hour cost.

5. **Choose an Instance Type:** Select **`t2.micro`** (Free Tier eligible — 1 vCPU, 1 GiB RAM).

   > ⚠️ **Easy to miss:** The default may not be `t2.micro`. Scroll through the list and select it explicitly.

6. **Key Pair (for SSH login):**
   - Click **"Create new key pair"**.
   - Enter a name (e.g., `my-ec2-key`).
   - Choose **RSA** format and **.pem** file format.
   - Click **"Create key pair"** — the `.pem` file will download automatically.

   > ⚠️ **Easy to miss:** **Download and save your `.pem` file immediately.** AWS will never show it again. If you lose it, you cannot connect to the instance and will need to create a new key pair.

7. **Network Settings:** Leave the defaults for now (a security group will be auto-created).
   - Make sure **"Allow SSH traffic"** is checked (port 22).
   - If you want to run a web server, also check **"Allow HTTP traffic"** (port 80).

8. **Storage:** The default 8 GiB is fine for this exercise (Free Tier allows up to 30 GiB).

9. Review the **Summary** on the right side, then click **"Launch instance"**.

10. Click **"View all instances"** to watch your instance move from `Pending` to `Running`.

    > ⚠️ **Easy to miss:** An instance takes 1–3 minutes to fully start. Wait until the **Instance State** column shows `Running` and the **Status Check** shows `2/2 checks passed` before trying to connect.

---

## 4. Connect to Your EC2 Instance

**Time:** ~5 minutes

### Option A: Connect via AWS Console (Browser — Easiest)

1. Select your instance in the EC2 dashboard.
2. Click **"Connect"** (top of the page).
3. Select the **"EC2 Instance Connect"** tab.
4. Click **"Connect"** — a terminal opens in your browser.

### Option B: Connect via SSH (Terminal)

**On Mac/Linux:**

```bash
# Move to the folder where your .pem file is downloaded
cd ~/Downloads

# Fix key file permissions (required — SSH will refuse the key otherwise)
chmod 400 my-ec2-key.pem

# Connect (replace <public-ip> with your instance's public IPv4 address)
ssh -i my-ec2-key.pem ec2-user@<public-ip>
```

> ⚠️ **Easy to miss:** If you get a `Permission denied (publickey)` error, check that:
> 1. You ran `chmod 400` on your `.pem` file
> 2. You're using the correct username (`ec2-user` for Amazon Linux, `ubuntu` for Ubuntu)
> 3. Port 22 is open in your instance's security group

**On Windows (using PuTTY):**
- Convert the `.pem` file to `.ppk` using **PuTTYgen**
- Open **PuTTY**, enter the public IP, and load the `.ppk` under Connection → SSH → Auth → Credentials

---

## 5. Install a Web Server (Optional)

Once connected via SSH, run these commands to install and start Apache:

```bash
# Update all packages
sudo yum update -y

# Install Apache web server
sudo yum install -y httpd

# Start Apache
sudo systemctl start httpd

# Enable Apache to start on reboot
sudo systemctl enable httpd

# Create a simple webpage
echo "<h1>Hello from my EC2 instance!</h1>" | sudo tee /var/www/html/index.html
```

Now open your browser and go to `http://<your-instance-public-ip>` — you should see your webpage.

> ⚠️ **Easy to miss:** Your security group must have **HTTP (port 80) open** from `0.0.0.0/0` for the web page to be accessible. Check Security Groups if the page doesn't load.

---

## 6. Stop and Terminate an Instance

| Action | Effect | Billing |
|--------|--------|---------|
| **Stop** | Instance is paused; data on EBS volumes is kept | No compute charge; EBS storage is still billed |
| **Terminate** | Instance is deleted permanently; data is lost (by default) | No further charges |

**To stop or terminate:**
1. In the EC2 dashboard, right-click your instance → **"Instance state"**.
2. Choose **"Stop"** or **"Terminate"**.

> ⚠️ **Easy to miss:** **Termination is permanent.** Once terminated, the instance and its storage (by default) cannot be recovered. Always double-check before terminating.

---

## 7. Security Groups

Security groups act as virtual firewalls. Each rule has:
- **Type** (e.g., SSH, HTTP, HTTPS, Custom TCP)
- **Protocol** (TCP, UDP, ICMP)
- **Port range** (e.g., 22 for SSH, 80 for HTTP)
- **Source** (IP address or range that is allowed)

**Common rules:**

| Purpose | Port | Protocol | Source |
|---------|------|----------|--------|
| SSH access | 22 | TCP | Your IP (`x.x.x.x/32`) |
| HTTP web traffic | 80 | TCP | `0.0.0.0/0` (everyone) |
| HTTPS web traffic | 443 | TCP | `0.0.0.0/0` (everyone) |
| Custom app | Any | TCP | Specific IP range |

> ⚠️ **Easy to miss:** Never open **SSH (port 22) to `0.0.0.0/0`** (all IP addresses) in a real environment — this exposes your server to brute-force attacks. Always restrict SSH to your IP address only.

---

## References

### 📖 AWS Documentation
- [What is Amazon EC2?](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [Launch an EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [Connect to your Linux instance using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)
- [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Amazon EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)

### 🎬 YouTube Videos
- [AWS EC2 Tutorial For Beginners - TechWorld with Nana](https://www.youtube.com/watch?v=8TlukLu11Yo)
- [Launch EC2 Instance Step by Step - AWS Tutorial](https://www.youtube.com/watch?v=0Gz-PUnEUF0)
- [AWS EC2 Full Course - freeCodeCamp](https://www.youtube.com/watch?v=a9__D53WsUs)

---

**← Previous:** [02 — IAM](../02-iam/README.md)  
**Next →** [04 — S3: Simple Storage Service](../04-s3/README.md)
