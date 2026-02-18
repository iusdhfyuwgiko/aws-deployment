# ☁️ Serverless File Sharing Platform

> A cloud-native file sharing solution built on **AWS Lambda**, **API Gateway**, and **Amazon S3** — no servers to manage, infinitely scalable, and accessible from any HTTP client.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Use Cases](#use-cases)
- [Prerequisites](#prerequisites)
- [Step-by-Step Deployment Guide](#step-by-step-deployment-guide)
  - [Step 1: Create S3 Bucket](#step-1-create-s3-bucket)
  - [Step 2: Create Upload Lambda Function](#step-2-create-upload-lambda-function)
  - [Step 3: Create Download Lambda Function](#step-3-create-download-lambda-function)
  - [Step 4: Set Up API Gateway](#step-4-set-up-api-gateway)
  - [Step 5: Configure GET Method](#step-5-configure-get-method)
  - [Step 6: Configure POST Method](#step-6-configure-post-method)
  - [Step 7: Deploy API Gateway](#step-7-deploy-api-gateway)
  - [Step 8: Testing](#step-8-testing)
- [Lambda Function Code](#lambda-function-code)
- [IAM Permissions](#iam-permissions)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## Overview

The **Serverless File Sharing Platform** allows users to securely **upload** and **download** files via a simple HTTP API. Because it's fully serverless, you only pay for what you use — no idle compute costs, no server maintenance.

| Component | Service | Purpose |
|-----------|---------|---------|
| Compute | AWS Lambda | Handles upload/download logic |
| API Layer | Amazon API Gateway | Exposes RESTful HTTP endpoints |
| Storage | Amazon S3 | Durable, scalable object storage |

---

## Architecture

```
Client (Postman / curl / Browser)
        │
        ▼
┌───────────────────┐
│   API Gateway     │  ← RESTful API (POST /files, GET /files)
│  my-file-sharing  │
└────────┬──────────┘
         │
    ┌────┴─────┐
    │          │
    ▼          ▼
┌────────┐  ┌──────────────┐
│ Upload │  │   Download   │  ← AWS Lambda Functions
│Function│  │   Function   │
└────┬───┘  └──────┬───────┘
     │              │
     └──────┬───────┘
            ▼
   ┌─────────────────┐
   │   Amazon S3     │  ← File Storage
   │  (your-bucket)  │
   └─────────────────┘
```

**Request Flow:**
- `POST /files?fileName=<name>` → API Gateway → `UploadFunction` → S3 `PutObject`
- `GET /files?fileName=<name>` → API Gateway → `DownloadFunction` → S3 `GetObject`

---

## Use Cases

**📄 File Sharing**
Upload documents, images, or any digital asset via a POST request and share the download link with others.

**📦 File Distribution**
Content creators and developers can distribute software updates, media files, or data packages to end users.

**🤝 Collaborative Work**
Teams can securely share project resources and documents across different locations without needing a shared drive.

---

## Prerequisites

Before you begin, make sure you have the following:

- ✅ An **AWS Account** with permissions to create:
  - S3 Buckets
  - Lambda Functions
  - IAM Roles & Policies
  - API Gateway APIs
- ✅ Access to the **AWS Management Console**
- ✅ **Postman** or **curl** installed for testing
- ✅ Passion to Learn! 🔥

---

## Step-by-Step Deployment Guide

### Step 1: Create S3 Bucket

1. Navigate to **S3** in the AWS Console.
2. Click **"Create bucket"**.
3. Enter a unique bucket name:
   ```
   my-file-sharing-bucket-amc
   ```
4. Choose your preferred **AWS Region**.
5. Leave **Block all public access** enabled (access is controlled via Lambda + IAM).
6. Click **"Create bucket"**.

> 💡 **Tip:** S3 bucket names must be globally unique. If the name is taken, add a suffix like your initials or a random number.

---

### Step 2: Create Upload Lambda Function

1. Navigate to **Lambda** → **"Create function"**.
2. Configure the function:
   - **Name:** `UploadFunction`
   - **Runtime:** `Python 3.x` (use the latest available version)
   - **Execution role:** Create a new role with **S3 write permissions** (see [IAM Permissions](#iam-permissions))
3. Paste the **UploadFunction** Python code (see [Lambda Function Code](#lambda-function-code)).
4. Click **"Deploy"**.

---

### Step 3: Create Download Lambda Function

1. Navigate to **Lambda** → **"Create function"**.
2. Configure the function:
   - **Name:** `DownloadFunction`
   - **Runtime:** `Python 3.x`
   - **Execution role:** Create a new role with **S3 read permissions** (see [IAM Permissions](#iam-permissions))
3. Paste the **DownloadFunction** Python code (see [Lambda Function Code](#lambda-function-code)).
4. Click **"Deploy"**.

---

### Step 4: Set Up API Gateway

1. Navigate to **API Gateway** → **"Create API"** → Choose **REST API**.
2. **Name:** `my-file-sharing-api-amc`
3. Create a resource:
   - Click **"Actions"** → **"Create Resource"**
   - **Resource Name:** `files`
   - **Resource Path:** `/files`
4. Create two methods under `/files`:
   - **POST** → Lambda Integration → Select `UploadFunction`
   - **GET** → Lambda Integration → Select `DownloadFunction`

---

### Step 5: Configure GET Method

This step ensures the `fileName` query parameter is passed correctly to the Lambda function.

1. Select the **GET** method under `/files`.
2. Click **"Method Request"** → **"Edit"**:
   - Set **Request Validator** to: `Validate Query String Parameters and Headers`
   - Under **URL Query String Parameters**, add: `fileName` (mark as **Required**)
3. Click **"Integration Request"** → **"Edit"** → **"Mapping Templates"**:
   - **Content-Type:** `application/json`
   - Paste the following template:
     ```json
     {
       "queryStringParameters": {
         "fileName": "$input.params('fileName')"
       }
     }
     ```
4. Click **"Save"**.

---

### Step 6: Configure POST Method

This step passes both the file content (body) and the `fileName` query parameter to the Lambda function.

1. Select the **POST** method under `/files`.
2. Click **"Integration Request"** → **"Edit"** → **"Mapping Templates"**:
   - **Content-Type:** `text/plain`
   - Paste the following template:
     ```json
     {
       "body": "$input.body",
       "queryStringParameters": {
         "fileName": "$input.params('fileName')"
       }
     }
     ```
3. Click **"Save"**.

---

### Step 7: Deploy API Gateway

1. Click **"Actions"** → **"Deploy API"**.
2. For **Deployment Stage**, choose or create a stage named `dev`.
3. Click **"Deploy"**.
4. Note the **Invoke URL** — it will look like:
   ```
   https://<api-id>.execute-api.<region>.amazonaws.com/dev
   ```

> 💡 Every time you make changes to your API, you need to **re-deploy** for the changes to take effect.

---

### Step 8: Testing

Replace `<api-id>` and `<region>` with the values from your deployed API Gateway.

#### 📤 Upload a File

**Using curl:**
```bash
curl --location 'https://<api-id>.execute-api.<region>.amazonaws.com/dev/files?fileName=test.txt' \
--header 'Content-Type: text/plain' \
--data 'Hello World from A Monk in Cloud!'
```

**Using Postman:**
1. Set method to **POST**
2. URL: `https://<api-id>.execute-api.<region>.amazonaws.com/dev/files?fileName=test.txt`
3. Body → **raw** → **Text**
4. Enter content: `Hello World from A Monk in Cloud!`
5. Click **Send**

**Expected Response:**
```json
{
  "message": "File uploaded successfully",
  "fileName": "test.txt"
}
```

---

#### 📥 Download a File

**Using curl:**
```bash
curl --location 'https://<api-id>.execute-api.<region>.amazonaws.com/dev/files?fileName=test.txt'
```

**Using Postman:**
1. Set method to **GET**
2. URL: `https://<api-id>.execute-api.<region>.amazonaws.com/dev/files?fileName=test.txt`
3. Click **Send**

**Expected Response:**
```
Hello World from A Monk in Cloud!
```

---

## Lambda Function Code

### UploadFunction (Python)

```python
import json
import boto3
import base64

s3 = boto3.client('s3')
BUCKET_NAME = 'my-file-sharing-bucket-amc'

def lambda_handler(event, context):
    try:
        file_name = event['queryStringParameters']['fileName']
        file_content = event['body']

        s3.put_object(
            Bucket=BUCKET_NAME,
            Key=file_name,
            Body=file_content
        )

        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'File uploaded successfully',
                'fileName': file_name
            })
        }

    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

### DownloadFunction (Python)

```python
import json
import boto3

s3 = boto3.client('s3')
BUCKET_NAME = 'my-file-sharing-bucket-amc'

def lambda_handler(event, context):
    try:
        file_name = event['queryStringParameters']['fileName']

        response = s3.get_object(
            Bucket=BUCKET_NAME,
            Key=file_name
        )

        file_content = response['Body'].read().decode('utf-8')

        return {
            'statusCode': 200,
            'body': file_content
        }

    except s3.exceptions.NoSuchKey:
        return {
            'statusCode': 404,
            'body': json.dumps({'error': f'File "{file_name}" not found'})
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

---

## IAM Permissions

Each Lambda function needs an execution role with the appropriate S3 permissions.

### UploadFunction Role Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-file-sharing-bucket-amc/*"
    }
  ]
}
```

### DownloadFunction Role Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-file-sharing-bucket-amc/*"
    }
  ]
}
```

> 🔐 **Security Best Practice:** Follow the **Principle of Least Privilege** — only grant the exact permissions each function needs. The UploadFunction only needs `PutObject`, not `GetObject` or `DeleteObject`.

---

## Troubleshooting

| Problem | Likely Cause | Solution |
|--------|-------------|---------|
| `403 Forbidden` on upload | Lambda role missing S3 write permission | Attach `s3:PutObject` to the Lambda execution role |
| `403 Forbidden` on download | Lambda role missing S3 read permission | Attach `s3:GetObject` to the Lambda execution role |
| `404 Not Found` on download | File doesn't exist in S3 | Verify the `fileName` matches exactly what was uploaded |
| `500 Internal Server Error` | Mapping template misconfigured | Re-check Steps 5 & 6 mapping templates |
| Changes not reflecting | API not re-deployed | Re-deploy the API Gateway after any changes |
| `Missing Authentication Token` | Wrong URL or stage | Confirm the full URL includes `/dev/files` |

---

## Contributing

Contributions are welcome! If you'd like to improve this project:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

---

> Built with ❤️ using AWS Serverless Services | Inspired by **A Monk in Cloud**
