# pipeline
testing and learning
# CI Pipeline: GitHub → Docker Hub + AWS SNS Notification

This project demonstrates how to automate Docker image creation using GitHub Actions and get a notification when the image is pushed to Docker Hub using AWS Lambda + SNS.

This is **Continuous Integration (CI)** only.  
(We are *not* automatically deploying to Kubernetes in this workflow.)

---

## 1️⃣ Objective

Whenever we push code to the `main` branch:

1. GitHub Actions should build a Docker image.
2. The image should be pushed to **Docker Hub**.
3. An **AWS Lambda** function should run automatically (through API Gateway).
4. The Lambda function sends an **Email Notification** using **AWS SNS**.

---

## 2️⃣ Prerequisites

| Requirement | Description |
|------------|-------------|
| GitHub Account | To host code and run workflow |
| Docker Hub Account | To store images |
| AWS Account | To configure Lambda + SNS + API Gateway |
| EC2/Kubernetes (optional) | Only if deploying the container later |

---

## 3️⃣ Configuring Docker Hub

1. Create a repository in Docker Hub  
   Example:  

2. Create a **Docker Hub Access Token**
- Login → Account Settings → **Security**
- Click **New Access Token**
- Copy the token (will be used as GitHub Secret)

3. Store these in GitHub → Settings → Secrets → Actions:

| Secret Name | Value |
|------------|--------|
| `DOCKER_USERNAME` | Your Docker Hub username |
| `DOCKER_TOKEN` | The access token you generated |

---

## 4️⃣ GitHub Actions Workflow (CI Pipeline)

File: `.github/workflows/docker-pipeline.yml`

```yaml
name: Build and Push Docker Image

on:
push:
 branches: 
   - main

jobs:
build:
 runs-on: ubuntu-latest

 steps:
 - name: Checkout Code
   uses: actions/checkout@v3

 - name: Login to Docker Hub
   run: echo "${{ secrets.DOCKER_TOKEN }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

 - name: Build Docker Image
   run: docker build -t ${{ secrets.DOCKER_USERNAME }}/pipeline:latest .

 - name: Push to Docker Hub
   run: docker push ${{ secrets.DOCKER_USERNAME }}/pipeline:latest

 - name: Trigger Lambda Notification
   run: curl -X GET "https://zkaordy2p6.execute-api.us-west-1.amazonaws.com/Notify"
