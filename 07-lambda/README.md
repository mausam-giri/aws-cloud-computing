# 07 — Lambda: Serverless Functions

← [Back to Main Index](../README.md) | [← 06 RDS](../06-rds/README.md)

AWS Lambda lets you run code **without provisioning or managing servers**. You pay only for the compute time your code consumes — no charge when it's not running.

---

## 📚 Table of Contents

1. [What is AWS Lambda?](#1-what-is-aws-lambda)
2. [Key Concepts](#2-key-concepts)
3. [Create Your First Lambda Function](#3-create-your-first-lambda-function)
4. [Test Your Lambda Function](#4-test-your-lambda-function)
5. [Add Environment Variables](#5-add-environment-variables)
6. [Trigger Lambda from S3](#6-trigger-lambda-from-s3)
7. [Trigger Lambda with API Gateway](#7-trigger-lambda-with-api-gateway)
8. [Monitor Lambda with CloudWatch](#8-monitor-lambda-with-cloudwatch)
9. [Lambda Pricing and Free Tier](#9-lambda-pricing-and-free-tier)
10. [References](#references)

---

## 1. What is AWS Lambda?

Lambda is a **serverless compute service** — you upload your code and Lambda runs it in response to events. You don't manage any servers.

**Real-world use cases:**
- REST API backends (with API Gateway)
- Processing S3 uploads (e.g., resize images, parse CSV files)
- Responding to database changes (e.g., DynamoDB Streams)
- Scheduled jobs (e.g., cleanup tasks, reports via EventBridge)
- IoT event processing

---

## 2. Key Concepts

| Term | Description |
|------|-------------|
| **Function** | Your code + configuration that Lambda runs |
| **Trigger** | An event source that invokes your function (S3, API Gateway, etc.) |
| **Handler** | The entry point function that Lambda calls |
| **Runtime** | The language environment (Python, Node.js, Java, Go, etc.) |
| **Execution Role** | IAM role that grants your function permissions to access AWS services |
| **Timeout** | Max duration your function can run (default: 3 seconds, max: 15 minutes) |
| **Memory** | RAM allocated to your function (128 MB to 10,240 MB) |
| **Layer** | A reusable package of libraries/dependencies shared across functions |
| **Cold Start** | The first invocation delay when Lambda initializes a new container |

---

## 3. Create Your First Lambda Function

**Time:** ~10 minutes

1. In the AWS Console, search for **"Lambda"** and open the Lambda service.
2. Click **"Create function"**.
3. Select **"Author from scratch"**.
4. **Function name:** Enter `hello-world-function`.
5. **Runtime:** Select **"Python 3.12"** (or your preferred language).
6. **Architecture:** Leave as `x86_64`.
7. Under **"Permissions"**, select **"Create a new role with basic Lambda permissions"**.

   > ⚠️ **Easy to miss:** The execution role controls what AWS services your function can access. The default basic role only allows writing logs to CloudWatch. If your function needs to access S3 or DynamoDB, you must **add those permissions to the role** — Lambda will silently fail or throw an access denied error without them.

8. Click **"Create function"**.
9. In the **Code** tab, replace the existing code with:

```python
import json

def lambda_handler(event, context):
    name = event.get('name', 'World')
    message = f"Hello, {name}! This is my first Lambda function."
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': message
        })
    }
```

10. Click **"Deploy"** to save and deploy your code.

    > ⚠️ **Easy to miss:** Changes to your code are **not live until you click "Deploy"**. The console shows an orange "Changes not deployed" indicator when there are unsaved changes. Test events will use the **last deployed version** of your code.

---

## 4. Test Your Lambda Function

1. Click the **"Test"** tab (next to Code tab).
2. Click **"Create new event"**.
3. Enter an event name (e.g., `TestEvent`).
4. Replace the default JSON with:

```json
{
  "name": "AWS Learner"
}
```

5. Click **"Save"** → **"Test"**.
6. The **Execution results** panel will appear, showing:
   - **Response:** The value your function returned
   - **Function Logs:** CloudWatch log output
   - **Duration:** How long your function ran

Expected output:
```json
{
  "statusCode": 200,
  "body": "{\"message\": \"Hello, AWS Learner! This is my first Lambda function.\"}"
}
```

> ⚠️ **Easy to miss:** The test event is only used in the console for manual testing. It does **not** represent how real triggers (S3, API Gateway) invoke your function — those have different event structures. Check the AWS docs for each trigger's event format.

---

## 5. Add Environment Variables

Environment variables let you store configuration without hardcoding it in your code.

1. In your function, go to the **"Configuration"** tab → **"Environment variables"**.
2. Click **"Edit"** → **"Add environment variable"**.
3. Add:
   - **Key:** `GREETING`
   - **Value:** `Welcome to AWS Lambda!`
4. Click **"Save"**.

Update your code to use the environment variable:

```python
import json
import os

def lambda_handler(event, context):
    name = event.get('name', 'World')
    greeting = os.environ.get('GREETING', 'Hello')
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f"{greeting}, {name}!"
        })
    }
```

> ⚠️ **Easy to miss:** Never store **sensitive secrets** (API keys, passwords) as plain-text environment variables in Lambda. Use **AWS Secrets Manager** or **AWS Systems Manager Parameter Store** for sensitive data.

---

## 6. Trigger Lambda from S3

You can automatically invoke a Lambda function whenever a file is uploaded to S3.

**Set up permissions first:**

1. Go to your function → **"Configuration"** → **"Permissions"** → click the Execution role name.
2. Click **"Add permissions"** → **"Attach policies"**.
3. Add `AmazonS3ReadOnlyAccess` (or a more specific policy).

**Add the S3 trigger:**

1. In your function, go to the **"Function overview"** section and click **"Add trigger"**.
2. Select **"S3"** from the dropdown.
3. Choose your S3 bucket.
4. **Event type:** Select **"All object create events"** (or a specific event).
5. (Optional) Add a **Prefix** (e.g., `uploads/`) to only trigger on files in a specific folder.
6. Check the acknowledgment checkbox.
7. Click **"Add"**.

   > ⚠️ **Easy to miss:** If the S3 bucket and Lambda function are in **different regions**, triggers won't work. Keep them in the same region.

Update your function to process S3 events:

```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get the uploaded file details from the S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    print(f"New file uploaded: s3://{bucket}/{key}")
    
    return {
        'statusCode': 200,
        'body': json.dumps(f"Processed: {key}")
    }
```

---

## 7. Trigger Lambda with API Gateway

Combine Lambda with API Gateway to create a REST API.

1. In your function → **"Function overview"** → **"Add trigger"**.
2. Select **"API Gateway"**.
3. Select **"Create a new API"**.
4. **API type:** Select **"HTTP API"** (cheaper and simpler than REST API for basic use).
5. **Security:** Select **"Open"** (for testing — add authentication in production).
6. Click **"Add"**.
7. Copy the **API endpoint URL** shown in the trigger details.
8. Test it in your browser or with curl:

```bash
curl "https://your-api-id.execute-api.us-east-1.amazonaws.com/default/hello-world-function"
```

   > ⚠️ **Easy to miss:** "Open" security means **anyone with the URL can invoke your function**. This is fine for learning, but in production always add authentication (API keys, Cognito, or IAM authorization).

---

## 8. Monitor Lambda with CloudWatch

1. In your function, click the **"Monitor"** tab.
2. You'll see metrics for:
   - **Invocations:** How many times the function was called
   - **Duration:** Average execution time
   - **Errors:** Number of failed invocations
   - **Throttles:** Times the function was rate-limited
3. Click **"View CloudWatch logs"** to see detailed log output from your function.

   > ⚠️ **Easy to miss:** Lambda logs are **not free** — CloudWatch Logs storage and data ingestion have their own costs. However, the Free Tier includes 5 GB of log storage per month, which is usually more than enough for learning.

---

## 9. Lambda Pricing and Free Tier

**Free Tier (per month, every month — does not expire):**
- 1 million function requests
- 400,000 GB-seconds of compute time

For most learning exercises, you'll never exceed the Free Tier.

**Pricing beyond Free Tier:**
- $0.20 per 1 million requests
- $0.0000166667 per GB-second of compute

> ⚠️ **Easy to miss:** Lambda's Free Tier is **permanent** (unlike EC2 which is Free Tier for 12 months only). However, related services like API Gateway, S3, and CloudWatch Logs have their own pricing.

---

## References

### 📖 AWS Documentation
- [What is AWS Lambda?](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Getting started with Lambda](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html)
- [Using AWS Lambda with Amazon S3](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)
- [Using Lambda with API Gateway](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)
- [Lambda pricing](https://aws.amazon.com/lambda/pricing/)

### 🎬 YouTube Videos
- [AWS Lambda Tutorial For Beginners - TechWorld with Nana](https://www.youtube.com/watch?v=eOBq__h4OJ4)
- [AWS Lambda + API Gateway Tutorial](https://www.youtube.com/watch?v=uFsaiEhr1zs)
- [AWS Lambda Full Course - freeCodeCamp](https://www.youtube.com/watch?v=DLWd85JfBus)

---

**← Previous:** [06 — RDS](../06-rds/README.md)  
**← Back to:** [Main Index](../README.md)
