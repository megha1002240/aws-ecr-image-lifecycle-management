# aws-ecr-image-lifecycle-management
# Secure Private Container Registry with Image Lifecycle and Retention Policies

## Project Overview

This project demonstrates how to implement a **secure private container registry** using AWS services.
The solution ensures secure storage and management of Docker images with controlled access and automated lifecycle cleanup.

The project uses **Amazon Elastic Container Registry (ECR)** as a private container registry and implements **IAM-based access control** along with lifecycle policies for automatic image cleanup.

---

## Objectives

* Create a **private container registry**
* Push multiple Docker image versions
* Implement **access control using IAM**
* Configure **lifecycle policy for automatic cleanup**
* Demonstrate **security validation using unauthorized access**

---

## Technologies Used

* Docker
* AWS EC2
* Amazon ECR
* AWS IAM
* AWS CLI

---

## Architecture Workflow

Developer → Build Docker Image → Push to ECR → Apply Lifecycle Policy → Secure Access via IAM

---

## Step 1: Launch EC2 Instance

An EC2 instance is launched to build and push Docker images.

Steps:

1. Launch Ubuntu EC2 instance
2. Connect using SSH
3. Install Docker and AWS CLI

Commands:

```
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

Install AWS CLI:

```
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

---

## Step 2: Configure AWS CLI

```
aws configure
```

Provide:

* Access Key
* Secret Key
* Region
* Output format

---

## Step 3: Create Private ECR Repository

1. Go to AWS Console
2. Open **Amazon ECR**
3. Create repository

Repository Name:

```
secure-app
```

Repository URI example:

```
718383533665.dkr.ecr.eu-north-1.amazonaws.com/secure-app
```

---

## Step 4: Docker Application

### app.py

```
print("Hello DevOps Secure Registry Project")
```

### Dockerfile

```
FROM python:3.9
WORKDIR /app
COPY app.py .
CMD ["python","app.py"]
```

---

## Step 5: Build Docker Image

```
docker build -t secure-app:v1 .
```

---

## Step 6: Login to Amazon ECR

```
aws ecr get-login-password --region eu-north-1 \
| docker login --username AWS \
--password-stdin 718383533665.dkr.ecr.eu-north-1.amazonaws.com
```

---

## Step 7: Tag Docker Image

```
docker tag secure-app:v1 \
718383533665.dkr.ecr.eu-north-1.amazonaws.com/secure-app:v1
```

---

## Step 8: Push Image to ECR

```
docker push \
718383533665.dkr.ecr.eu-north-1.amazonaws.com/secure-app:v1
```

Multiple versions were pushed:

```
v1
v2
v3
v4
v5
```

---

## Step 9: Lifecycle Policy

A lifecycle policy was configured to retain only the **latest 3 images**.

Example lifecycle policy:

```
{
 "rules": [
   {
     "rulePriority": 1,
     "description": "Keep last 3 images",
     "selection": {
       "tagStatus": "any",
       "countType": "imageCountMoreThan",
       "countNumber": 3
     },
     "action": {
       "type": "expire"
     }
   }
 ]
}
```

Result:

Before Cleanup

```
v1
v2
v3
v4
v5
```

After Cleanup

```
v3
v4
v5
```

Older images are automatically deleted.

---

## Step 10: Repository Security Policy

Access to the repository is restricted using IAM role.

Example repository policy:

```
{
 "Version": "2008-10-17",
 "Statement": [
   {
     "Sid": "AllowPushPullFromDevOpsRole",
     "Effect": "Allow",
     "Principal": {
       "AWS": "arn:aws:iam::718383533665:role/DevOpsRole"
     },
     "Action": [
       "ecr:GetDownloadUrlForLayer",
       "ecr:BatchGetImage",
       "ecr:BatchCheckLayerAvailability",
       "ecr:PutImage"
     ]
   }
 ]
}
```

---

## Step 11: Security Validation

An unauthorized IAM user attempted to push an image to the repository.

Result:

```
AccessDeniedException
```

This confirms that **only authorized roles can push or pull images**.

---

## Project Deliverables

* Repository Policy JSON
* Lifecycle Policy JSON
* Screenshots of image cleanup
* Unauthorized access demonstration
* Project documentation (README)

---

## Conclusion

This project successfully demonstrates the implementation of a **secure private container registry** using AWS services.
The system ensures controlled access using IAM policies and automatically manages image retention using lifecycle rules.
This improves security, reduces storage costs, and ensures efficient container image management.

---
