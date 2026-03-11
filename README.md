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
<img width="940" height="152" alt="image" src="https://github.com/user-attachments/assets/dc6b290b-1788-4248-8d5f-d11b660ee3de" />
<img width="840" height="86" alt="image" src="https://github.com/user-attachments/assets/1edc25b1-98f3-4df3-b903-45ced4dd4616" />
<img width="715" height="70" alt="image" src="https://github.com/user-attachments/assets/80aed0c0-ce38-4411-8fe5-7d89b6b5e1d7" />



Install AWS CLI:

```
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
<img width="1920" height="1008" alt="Screenshot 2026-03-09 141139" src="https://github.com/user-attachments/assets/07b42a8c-b055-4934-9ced-765bc4754d85" />

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
<img width="1920" height="1008" alt="Screenshot 2026-03-09 141139" src="https://github.com/user-attachments/assets/aa3ecab4-1e89-4d94-8514-cf159321e88a" />

---

## Step 3: Create Private ECR Repository

1. Go to AWS Console
2. Open **Amazon ECR**
3. Create repository

Repository Name:

```
secure-app
```
<img width="1920" height="1008" alt="Screenshot 2026-03-09 140945" src="https://github.com/user-attachments/assets/fdd937c8-2150-46c3-88ec-64d7ac19a130" />

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
<img width="1470" height="681" alt="image" src="https://github.com/user-attachments/assets/22f592a1-d2cf-4061-9d0d-bd8841d762fb" />

---

## Step 6: Login to Amazon ECR

```
aws ecr get-login-password --region eu-north-1 \
| docker login --username AWS \
--password-stdin 718383533665.dkr.ecr.eu-north-1.amazonaws.com
```
<img width="1912" height="153" alt="image" src="https://github.com/user-attachments/assets/8797bdc9-c2a1-44bd-a594-7b034d92dfcb" />

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
<img width="1629" height="100" alt="image" src="https://github.com/user-attachments/assets/8b79559d-794f-444f-a626-414579c99b4f" />

Multiple versions were pushed:

```
v1
v2
v3
v4
v5
```
<img width="1920" height="1008" alt="image" src="https://github.com/user-attachments/assets/73237178-448b-471b-af63-a7de82900086" />

---

## Step 9: Lifecycle Policy

A lifecycle policy was configured to retain only the **latest 3 images**.
<img width="1920" height="1008" alt="Screenshot 2026-03-09 144927" src="https://github.com/user-attachments/assets/782fdc88-ec89-4713-8157-1be192e39297" />
<img width="1920" height="1008" alt="Screenshot 2026-03-09 144949" src="https://github.com/user-attachments/assets/bd03742e-c81f-439e-93cc-41875c74b6b9" />


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
<img width="1920" height="1008" alt="Screenshot 2026-03-09 145103" src="https://github.com/user-attachments/assets/220cd80f-efca-483c-83a1-9ba4dce04e8f" />

Result:

Before Cleanup

```
v1
v2
v3
v4
v5
```
<img width="1920" height="1008" alt="image" src="https://github.com/user-attachments/assets/ab760bc0-9a04-484e-a8e2-88667d0dbb6d" />

After Cleanup

```
v3
v4
v5
```
<img width="1920" height="1008" alt="image" src="https://github.com/user-attachments/assets/d766fce1-fa7c-455a-9edb-80dba5b526b2" />

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
<img width="1905" height="309" alt="Screenshot 2026-03-11 152220" src="https://github.com/user-attachments/assets/ce751ae3-be2a-4d73-982d-69ee60e4f9b4" />

## Conclusion

This project successfully demonstrates the implementation of a **secure private container registry** using AWS services.
The system ensures controlled access using IAM policies and automatically manages image retention using lifecycle rules.
This improves security, reduces storage costs, and ensures efficient container image management.

---
