# 01 — Getting Started with AWS

← [Back to Main Index](../README.md)

This section walks you through creating an AWS account, securing it, setting up billing alerts, and navigating the AWS Management Console for the very first time.

---

## 📚 Table of Contents

1. [Create an AWS Account](#1-create-an-aws-account)
2. [Verify Your Account](#2-verify-your-account)
3. [Secure the Root Account](#3-secure-the-root-account)
4. [Set Up a Billing Alert](#4-set-up-a-billing-alert)
5. [Explore the AWS Management Console](#5-explore-the-aws-management-console)
6. [Choose an AWS Region](#6-choose-an-aws-region)
7. [References](#references)

---

## 1. Create an AWS Account

**Time:** ~10 minutes

1. Open your browser and go to [https://aws.amazon.com](https://aws.amazon.com).
2. Click **"Create an AWS Account"** (top-right corner).
3. Enter your **email address** and choose an **AWS account name** (e.g., `my-learning-account`).
4. Click **"Verify email address"** — AWS will send you a verification code.

   > ⚠️ **Easy to miss:** Check your **spam/junk folder** if you don't see the email within 2 minutes.

5. Enter the verification code and click **"Verify"**.
6. Create a **strong password** for your root account. Save it in a password manager.

   > ⚠️ **Easy to miss:** This password is for the **root account** — the most powerful account. Treat it like a master key.

7. Select **"Personal"** for account type (unless creating for a business).
8. Fill in your name, address, and phone number. Click **"Continue"**.

---

## 2. Verify Your Account

**Time:** ~5 minutes

1. Enter your **credit or debit card details**.

   > ⚠️ **Easy to miss:** AWS will charge a **$1 temporary authorization** to verify your card. This is refunded. You will **not** be billed as long as you stay within the [Free Tier](https://aws.amazon.com/free/).

2. Choose your **Support Plan** → Select **"Basic support - Free"**.

   > ⚠️ **Easy to miss:** The default selection may highlight a paid plan. Make sure to select **Basic (Free)** unless you specifically need paid support.

3. Click **"Complete sign up"**.
4. AWS will send a welcome email. Your account may take up to **24 hours** to be fully activated, but it usually activates within a few minutes.

---

## 3. Secure the Root Account

**Time:** ~10 minutes

The root account has unlimited access to everything. You should **only use it for initial setup**, then lock it away.

### Enable Multi-Factor Authentication (MFA)

1. Sign in at [https://console.aws.amazon.com](https://console.aws.amazon.com) with your root email and password.
2. Click on your **account name** (top-right corner) → **"Security credentials"**.
3. Under **"Multi-factor authentication (MFA)"**, click **"Assign MFA device"**.
4. Choose **"Authenticator app"** (recommended), then click **"Next"**.
5. Open an authenticator app (e.g., Google Authenticator, Authy) on your phone.
6. Scan the QR code shown on screen.
7. Enter two consecutive **6-digit codes** from your app to verify, then click **"Add MFA"**.

   > ⚠️ **Easy to miss:** You must enter **two consecutive codes** (not the same code twice). Wait for the code to refresh and enter the next one.

---

## 4. Set Up a Billing Alert

**Time:** ~5 minutes

Billing alerts protect you from unexpected charges.

1. In the AWS Console, search for **"Billing"** in the top search bar → click **"Billing and Cost Management"**.
2. In the left sidebar, click **"Budgets"** → **"Create budget"**.
3. Select **"Use a template"** → choose **"Zero spend budget"** (alerts you if any charges occur).

   > ⚠️ **Easy to miss:** The "Zero spend budget" template is the safest option for beginners. It notifies you the moment you incur any charge beyond the Free Tier.

4. Enter your email address for notifications.
5. Click **"Create budget"**.

---

## 5. Explore the AWS Management Console

**Time:** ~10 minutes

1. Sign in at [https://console.aws.amazon.com](https://console.aws.amazon.com).
2. The **search bar** at the top is your best friend — you can search for any AWS service by name.
3. Click the **grid icon** (top-left) to see all services organized by category.
4. The **"Recently visited"** panel on the home page shows services you've used recently.

Key console areas to know:
| Area | What It Does |
|------|--------------|
| Services Menu | Access any AWS service |
| Region Selector | Switch between geographic regions (top-right) |
| Account Menu | Billing, security settings, sign out (top-right) |
| CloudShell | Browser-based terminal for CLI commands |
| Cost Explorer | View and analyze your spending |

---

## 6. Choose an AWS Region

**Time:** 2 minutes

AWS services run in **Regions** around the world. Always choose a region before creating resources.

1. In the AWS Console, look for the **region name** in the top-right corner (e.g., `US East (N. Virginia)`).
2. Click it to open the region dropdown and select a region close to you.

   > ⚠️ **Easy to miss:** Resources you create are **region-specific**. If you create an EC2 instance in `us-east-1` and then switch to `eu-west-1`, you won't see it. Always confirm your region before looking for resources.

**Recommended region for beginners:** `us-east-1` (US East - N. Virginia) — it has the most services available and is used in most tutorials.

---

## References

### 📖 AWS Documentation
- [Create and activate an AWS account](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html)
- [Enable MFA for your AWS account root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/enable-virt-mfa-for-root.html)
- [Creating a billing alarm to monitor your estimated AWS charges](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)
- [AWS Free Tier](https://aws.amazon.com/free/)

### 🎬 YouTube Videos
- [AWS Tutorial For Beginners | AWS Full Course - freeCodeCamp](https://www.youtube.com/watch?v=ubCNZFXd4NQ)
- [AWS Account Setup - Step by Step - TechWorld with Nana](https://www.youtube.com/watch?v=CjKhQoYeR4Q)
- [AWS Console Walkthrough for Beginners](https://www.youtube.com/watch?v=IT1X42D1KeA)

---

**Next →** [02 — IAM: Identity and Access Management](../02-iam/README.md)
