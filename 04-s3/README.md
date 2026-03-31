# 04 — S3: Simple Storage Service

← [Back to Main Index](../README.md) | [← 03 EC2](../03-ec2/README.md)

Amazon S3 (Simple Storage Service) is an object storage service that lets you store and retrieve any amount of data at any time. It's one of the most widely used AWS services.

---

## 📚 Table of Contents

1. [What is S3?](#1-what-is-s3)
2. [Key Concepts](#2-key-concepts)
3. [Create an S3 Bucket](#3-create-an-s3-bucket)
4. [Upload Files to S3](#4-upload-files-to-s3)
5. [Make a File Publicly Accessible](#5-make-a-file-publicly-accessible)
6. [Host a Static Website on S3](#6-host-a-static-website-on-s3)
7. [Bucket Versioning](#7-bucket-versioning)
8. [S3 Storage Classes](#8-s3-storage-classes)
9. [References](#references)

---

## 1. What is S3?

S3 stores data as **objects** inside **buckets**. Unlike a file system, S3 doesn't use folders — but it uses **prefixes** (path-like names) to organize files.

**Real-world use cases:**
- Storing application files, images, videos, and documents
- Hosting static websites (HTML, CSS, JavaScript)
- Backup and disaster recovery
- Data lake for analytics
- Distribution source for CloudFront (CDN)

---

## 2. Key Concepts

| Term | Description |
|------|-------------|
| **Bucket** | A container for objects (like a top-level folder) |
| **Object** | A file + its metadata stored in a bucket |
| **Key** | The unique name/path of an object within a bucket (e.g., `images/logo.png`) |
| **Region** | The geographic location where your bucket is stored |
| **ACL** | Access Control List — controls object-level permissions |
| **Bucket Policy** | JSON policy controlling who can access the bucket |
| **Versioning** | Keeping multiple versions of the same object |

---

## 3. Create an S3 Bucket

**Time:** ~5 minutes

1. In the AWS Console, search for **"S3"** and open the S3 service.
2. Click **"Create bucket"**.
3. **Bucket name:** Enter a globally unique name (e.g., `my-learning-bucket-2024`).

   > ⚠️ **Easy to miss:** S3 bucket names must be **globally unique** across all AWS accounts. If the name is taken, try adding your name, date, or random numbers (e.g., `john-aws-learning-20240101`).

4. **Region:** Select the same region you're using for other services.

   > ⚠️ **Easy to miss:** Bucket region cannot be changed after creation. Choose it carefully — accessing data cross-region incurs extra costs.

5. **Block Public Access settings:** Keep **"Block all public access"** enabled for now (we'll change it in step 5 if needed).
6. Leave all other settings as default.
7. Click **"Create bucket"**.

---

## 4. Upload Files to S3

1. Click on your newly created bucket name.
2. Click **"Upload"** → **"Add files"**.
3. Select a file from your computer (e.g., an image or text file).
4. Click **"Upload"**.

   > ⚠️ **Easy to miss:** After clicking "Upload", wait for the green success banner before navigating away. Large files may take time to upload.

5. Click on the uploaded file to view its details and the **Object URL**.

---

## 5. Make a File Publicly Accessible

By default, all S3 objects are **private**. To share a file publicly:

### Step 1: Update Bucket Settings

1. Open your bucket → **"Permissions"** tab.
2. Click **"Edit"** under **"Block public access (bucket settings)"**.
3. Uncheck **"Block all public access"**.
4. Click **"Save changes"** → type `confirm` when prompted.

   > ⚠️ **Easy to miss:** You must **confirm** the dialog by typing `confirm` (lowercase) before the change takes effect.

### Step 2: Add a Bucket Policy

1. Still in the **"Permissions"** tab, scroll down to **"Bucket policy"**.
2. Click **"Edit"** and paste the following (replace `YOUR-BUCKET-NAME`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

3. Click **"Save changes"**.

Now the **Object URL** of any file in your bucket will be publicly accessible.

> ⚠️ **Easy to miss:** Making a bucket public exposes **all objects** in it to anyone on the internet. Never store private data (passwords, API keys, personal information) in a public bucket.

---

## 6. Host a Static Website on S3

You can host a static website (HTML/CSS/JS — no server-side code) directly from S3.

1. Create a file named `index.html` on your computer with this content:

```html
<!DOCTYPE html>
<html>
<head><title>My S3 Website</title></head>
<body>
  <h1>Hello from Amazon S3!</h1>
  <p>My first static website hosted on S3.</p>
</body>
</html>
```

2. Upload `index.html` to your bucket (follow [Upload Files](#4-upload-files-to-s3) steps).
3. Make the file public using the bucket policy from [Step 5](#5-make-a-file-publicly-accessible).
4. In your bucket, go to **"Properties"** tab.
5. Scroll down to **"Static website hosting"** → click **"Edit"**.
6. Select **"Enable"**.
7. Set **"Index document"** to `index.html`.
8. Click **"Save changes"**.
9. Back in the **"Properties"** tab, scroll to **"Static website hosting"** — copy the **Bucket website endpoint** URL.
10. Open the URL in your browser — you should see your webpage!

   > ⚠️ **Easy to miss:** The static website endpoint URL is different from the S3 Object URL. The website endpoint looks like: `http://bucket-name.s3-website-us-east-1.amazonaws.com`

---

## 7. Bucket Versioning

Versioning lets you keep multiple versions of an object, protecting against accidental deletes or overwrites.

1. Open your bucket → **"Properties"** tab.
2. Under **"Bucket Versioning"**, click **"Edit"** → **"Enable"** → **"Save changes"**.
3. Upload the same file again with different content.
4. In the bucket, click **"Show versions"** toggle to see all versions.

   > ⚠️ **Easy to miss:** Once versioning is enabled, it **cannot be fully disabled** — only suspended. Also, keeping multiple versions increases your storage costs.

---

## 8. S3 Storage Classes

Choose the right storage class to balance cost and access frequency:

| Storage Class | Use Case | Retrieval Time |
|---------------|----------|----------------|
| **S3 Standard** | Frequently accessed data | Milliseconds |
| **S3 Standard-IA** | Infrequently accessed, but needs fast retrieval | Milliseconds |
| **S3 One Zone-IA** | Infrequently accessed, no redundancy needed | Milliseconds |
| **S3 Glacier Instant** | Archive with occasional instant access | Milliseconds |
| **S3 Glacier Flexible** | Archive, retrieval in minutes to hours | Minutes–Hours |
| **S3 Glacier Deep Archive** | Long-term archive, cheapest option | 12+ hours |

> ⚠️ **Easy to miss:** Retrieving data from Glacier classes incurs additional **retrieval fees** beyond storage costs. Always check pricing before using Glacier for frequently accessed data.

---

## References

### 📖 AWS Documentation
- [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Creating a bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)
- [Hosting a static website using Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [Using bucket policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
- [Amazon S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)

### 🎬 YouTube Videos
- [Amazon S3 Tutorial For Beginners - TechWorld with Nana](https://www.youtube.com/watch?v=tfU0JEZjcsg)
- [AWS S3 Static Website Hosting - Step by Step](https://www.youtube.com/watch?v=BpFKnPae1oY)
- [AWS S3 Full Course - freeCodeCamp](https://www.youtube.com/watch?v=XGcoeEyt2UM)

---

**← Previous:** [03 — EC2](../03-ec2/README.md)  
**Next →** [05 — VPC: Virtual Private Cloud](../05-vpc/README.md)
