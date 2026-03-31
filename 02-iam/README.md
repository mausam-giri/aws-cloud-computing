# 02 — IAM: Identity and Access Management

← [Back to Main Index](../README.md) | [← 01 Getting Started](../01-getting-started/README.md)

IAM (Identity and Access Management) controls **who** can access your AWS account and **what** they can do. This is one of the most important services to understand before using anything else in AWS.

---

## 📚 Table of Contents

1. [What is IAM?](#1-what-is-iam)
2. [Create an IAM User](#2-create-an-iam-user)
3. [Create an IAM Group](#3-create-an-iam-group)
4. [Attach Policies to a Group](#4-attach-policies-to-a-group)
5. [Enable MFA for the IAM User](#5-enable-mfa-for-the-iam-user)
6. [Create an IAM Role](#6-create-an-iam-role)
7. [Understanding IAM Policies](#7-understanding-iam-policies)
8. [References](#references)

---

## 1. What is IAM?

IAM lets you create identities (users, groups, roles) and define what AWS actions they are allowed to perform.

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| **User** | A person or application that needs to access AWS |
| **Group** | A collection of users that share the same permissions |
| **Role** | A set of permissions that can be assumed temporarily (by users, services, etc.) |
| **Policy** | A JSON document that defines what actions are allowed or denied |

> ⚠️ **Easy to miss:** You should **never use your root account** for day-to-day tasks. Create an IAM admin user and use that instead. The root account should only be used for billing and account-level settings.

---

## 2. Create an IAM User

**Time:** ~5 minutes

1. Sign in to the AWS Console and search for **"IAM"** in the search bar.
2. In the left sidebar, click **"Users"** → **"Create user"**.
3. Enter a **username** (e.g., `admin-user`).
4. Check **"Provide user access to the AWS Management Console"**.
5. Select **"I want to create an IAM user"**.
6. Choose **"Custom password"** and enter a strong password.

   > ⚠️ **Easy to miss:** Uncheck **"Users must create a new password at next sign-in"** if you're setting this up for yourself and don't want to reset the password on first login.

7. Click **"Next"**.
8. On the **"Set permissions"** page — skip for now (we'll add the user to a group in the next step).
9. Click **"Next"** → **"Create user"**.
10. On the confirmation screen, copy the **Console sign-in URL** and save it somewhere safe.

    > ⚠️ **Easy to miss:** The console sign-in link contains your **Account ID**. Bookmark it — you'll need it every time you sign in as an IAM user.

---

## 3. Create an IAM Group

Groups make it easy to manage permissions for multiple users at once.

1. In the IAM left sidebar, click **"User groups"** → **"Create group"**.
2. Enter a group name (e.g., `Administrators`).
3. In the **"Add users to the group"** section, check the user you just created.
4. In the **"Attach permissions policies"** section, search for `AdministratorAccess`.
5. Check the **AdministratorAccess** policy.

   > ⚠️ **Easy to miss:** `AdministratorAccess` gives **full access** to all AWS services. This is fine for your personal admin user, but **never assign this to application users or service accounts** — always use least-privilege policies.

6. Click **"Create group"**.

---

## 4. Attach Policies to a Group

1. In the left sidebar, click **"User groups"**.
2. Click on a group name to open it.
3. Go to the **"Permissions"** tab → **"Add permissions"** → **"Attach policies"**.
4. Search for and select the policies you want to attach.
5. Click **"Attach policies"**.

**Common policies for beginners:**

| Policy Name | What it allows |
|-------------|----------------|
| `AdministratorAccess` | Full access to all AWS services |
| `ReadOnlyAccess` | View-only access to all services |
| `AmazonS3FullAccess` | Full access to S3 only |
| `AmazonEC2FullAccess` | Full access to EC2 only |
| `AmazonRDSFullAccess` | Full access to RDS only |

---

## 5. Enable MFA for the IAM User

1. In the IAM left sidebar, click **"Users"** → click your username.
2. Go to the **"Security credentials"** tab.
3. Under **"Multi-factor authentication (MFA)"**, click **"Assign MFA device"**.
4. Enter a device name, select **"Authenticator app"**, and click **"Next"**.
5. Scan the QR code with your authenticator app (Google Authenticator, Authy, etc.).
6. Enter **two consecutive codes** from the app, then click **"Add MFA"**.

   > ⚠️ **Easy to miss:** You need to enter **two different 6-digit codes** from the authenticator app (wait for the code to cycle before entering the second one). Entering the same code twice will fail.

---

## 6. Create an IAM Role

Roles are used when an AWS service (like EC2 or Lambda) needs to access another AWS service.

1. In the IAM left sidebar, click **"Roles"** → **"Create role"**.
2. Under **"Trusted entity type"**, select **"AWS service"**.
3. Choose the service that will use this role (e.g., **EC2**).
4. Click **"Next"**.
5. Search for and attach the required policies (e.g., `AmazonS3ReadOnlyAccess`).
6. Click **"Next"**.
7. Enter a **Role name** (e.g., `ec2-s3-read-role`) and an optional description.
8. Click **"Create role"**.

   > ⚠️ **Easy to miss:** The **trust policy** (set automatically in step 2-3) defines which service can *assume* this role. If EC2 can't access S3 after attaching the role, verify the trust policy allows `ec2.amazonaws.com`.

---

## 7. Understanding IAM Policies

Policies are JSON documents. Here's a simple example that allows listing S3 buckets:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    }
  ]
}
```

**Policy components:**

| Field | Description |
|-------|-------------|
| `Version` | Always use `"2012-10-17"` (the latest policy language version) |
| `Effect` | `Allow` or `Deny` |
| `Action` | The AWS API action(s) to allow/deny (e.g., `s3:GetObject`) |
| `Resource` | The specific resource ARN (or `*` for all) |

> ⚠️ **Easy to miss:** IAM follows **least privilege** — only grant the exact permissions needed. Avoid `"Action": "*"` or `"Resource": "*"` in production environments.

---

## References

### 📖 AWS Documentation
- [What is IAM?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [Creating your first IAM admin user and user group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html)
- [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### 🎬 YouTube Videos
- [AWS IAM Tutorial For Beginners - TechWorld with Nana](https://www.youtube.com/watch?v=iF9fs8Rw4Uo)
- [AWS IAM Full Tutorial - freeCodeCamp](https://www.youtube.com/watch?v=3y596T3YM8I)
- [AWS IAM Crash Course - Traversy Media](https://www.youtube.com/watch?v=Ul6FW4UANGc)

---

**← Previous:** [01 — Getting Started](../01-getting-started/README.md)  
**Next →** [03 — EC2: Elastic Compute Cloud](../03-ec2/README.md)
