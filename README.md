# 🚀 AWS Image Resizer Pro
Improved version of previous project(aws-image-resizer):
[link to the basic version](https://github.com/kyrylobaliurawork-web/aws-image-resizer)

---

# 📌 Project Overview

The main goal of this project is to demonstrate:

- Serverless architecture with AWS
- Image processing with Python
- API-driven image resizing
- Low-cost infrastructure using serverless services
- DNS and CDN with AWS
- Basic DB level 
- Unlike the basic version, improve our understanding of the code
- Configure custom domain routing using Amazon Route 53

---

# 🏗 Architecture

![Architecture Diagram](images/001-project-diagram.png)

# 🔄 Request Flow
  - User Request
      - A user sends a request via a custom domain.
  - DNS Routing
      - Amazon Route 53 resolves the domain and routes traffic to the CDN.
      - Configure custom domain.
  - Content Delivery
      - Amazon CloudFront acts as a CDN layer:
        - Serves cached images if available
        - Forwards uncached requests to the backend
  - API Gateway
        - Amazon API Gateway receives the request and triggers backend logic.
  - Lambda Image Resizer
      - AWS Lambda (Image Resizer function):
        - Fetches the original image from Amazon S3
        - Resizes the image using Python
        - Stores the processed image back to S3
        - Sends a message to the queue for analytics
  - Storage Layer
      - Amazon S3:
        - Stores both original and resized images
        - Acts as the origin for CloudFront
# 📊 Analytics Pipeline
  - Message Queue
      - Amazon SQS receives events about image processing
      - Click Count Processor
  - Another AWS Lambda function:
      - Processes messages from SQS
      - Updates usage statistics
  - Database
    - Amazon DynamoDB stores:
      - Image request counts
      - Analytics data
  - Monitoring
    - Amazon CloudWatch collects:
      - Logs
      - Metrics
      - System health data

# 🔐 Security Layer
  - IAM Roles control permissions between services:
      Lambda can access S3, SQS, and DynamoDB
      Logging permissions via CloudWatch

## AWS Lambda Image Resizer (Python)

```python
import boto3
from PIL import Image
import io
import base64
import json

# Create an S3 client
s3 = boto3.client("s3")

# Name of the S3 bucket
BUCKET = "images-resizer-bucket-own"

def lambda_handler(event, context):

    # Detect where parameters come from
    if "queryStringParameters" in event and event["queryStringParameters"]:
        params = event["queryStringParameters"]
    else:
        params = event

    # Validate parameters
    if "image" not in params or "width" not in params:
        return {
            "statusCode": 400,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({
                "error": "Missing parameters. Required: image, width"
            })
        }

    key = params["image"]
    width = int(params["width"])

    # Get image from S3
    obj = s3.get_object(Bucket=BUCKET, Key=key)
    image_bytes = obj["Body"].read()

    # Open image
    image = Image.open(io.BytesIO(image_bytes))

    # Keep aspect ratio
    ratio = width / image.width
    height = int(image.height * ratio)

    # Resize image
    resized = image.resize((width, height))

    # Save to memory
    buffer = io.BytesIO()
    resized.save(buffer, "JPEG")

    # Return image
    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "image/jpeg"
        },
        "body": base64.b64encode(buffer.getvalue()).decode("utf-8"),
        "isBase64Encoded": True
    }
```
## 📌 What this code does

This AWS Lambda function resizes an image stored in S3 based on a requested width and returns it as a JPEG.

---

## ⚙️ Key parts explained

### 🔹 `boto3.client("s3")`
Used to access AWS S3.  
Without it, the function cannot retrieve images.

---

### 🔹 `queryStringParameters` handling
Allows the function to work both via API Gateway and direct invocation.

---

### 🔹 Aspect ratio logic

```python
ratio = width / image.width
height = int(image.height * ratio)
```
Keeps the image proportions correct.  
Without this, the image would be distorted.

---

### 🔹 `Image.open(io.BytesIO(...))`

Opens the image directly from memory instead of saving to disk.  
✔ Important for AWS Lambda (no filesystem dependency)

---

### 🔹 `base64.b64encode(...)`

Required to return binary data (image) via API Gateway.  
Without this, the image would break in the response.

---

### 🔹 `isBase64Encoded = True`

Tells API Gateway the response is encoded.  
Without it, the client won’t decode the image properly.
