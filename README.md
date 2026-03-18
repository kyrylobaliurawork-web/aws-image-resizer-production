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
