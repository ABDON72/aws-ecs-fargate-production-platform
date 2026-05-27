# AWS ECS Fargate CI/CD Pipeline

Automated CI/CD pipeline deploying containerized Node.js microservices to AWS ECS Fargate using Docker, Terraform, and Jenkins with auto-scaling.

## Architecture Overview

Developer → GitHub → Jenkins → ECR → ECS Fargate → ALB → User

## Tech Stack

| Tool | Purpose |
|---|---|
| Docker | Containerize frontend and backend apps |
| Terraform | Provision all AWS infrastructure as code |
| Jenkins | Automate CI/CD pipeline |
| AWS ECS Fargate | Run containers without managing servers |
| AWS ECR | Store Docker images |
| AWS ALB | Load balance traffic to containers |
| AWS VPC | Private network for all resources |
| Auto Scaling | Scale containers based on CPU usage |

## Project Structure

frontend/ - React frontend app with Dockerfile
backend/ - Node.js backend API with Dockerfile
terraform/ - AWS infrastructure as code
Jenkinsfile - CI/CD pipeline definition

## Phase 1 — Local Setup

Prerequisites: Node.js v18+, Git

git clone https://github.com/ABDON72/aws-ecs-fargate-cicd-pipeline.git
cd aws-ecs-fargate-cicd-pipeline
cd backend && npm install && npm start
cd frontend && npm install && npm start
Open http://localhost:3000

## Phase 2 — Docker Setup

Prerequisites: Docker

docker network create app-network
docker build -t backend:latest ./backend
docker run -d --name backend --network app-network -p 8080:8080 backend:latest
docker build -t frontend:latest ./frontend
docker run -d --name frontend --network app-network -p 3000:3000 frontend:latest
Open http://localhost:3000

## Phase 3 — Terraform Infrastructure

Prerequisites: Terraform v1.0+, AWS CLI configured

cd terraform
terraform init
terraform plan
terraform apply -auto-approve

Resources created:
- VPC with public and private subnets
- Internet Gateway and NAT Gateway
- ECS Fargate cluster
- ECR repositories for frontend and backend
- Application Load Balancer
- IAM roles and security groups
- Auto Scaling policies trigger at 50% CPU

## Phase 4 — Jenkins Setup

Jenkins Infrastructure:
- EC2 instance type: t3.medium
- OS: Ubuntu 22.04
- Jenkins runs as Docker container
- Security group: port 22 SSH and 8080 Jenkins UI

Jenkins URL: http://44.198.161.151:8080

Required plugins:
- Docker Pipeline
- Amazon ECR
- AWS Credentials
- Pipeline AWS Steps

## Phase 5 — CI/CD Pipeline

Jenkinsfile stages:
- Checkout: Pull latest code from GitHub
- Build Docker Images: Build frontend and backend images
- Push to ECR: Push images to AWS ECR
- Deploy to ECS: Force new ECS deployment
- Validate Deployment: Confirm services are stable

## Phase 6 — Deployment Validation

Live application URL: http://ecs-fargate-cicd-alb-1845797115.us-east-1.elb.amazonaws.com

Validation:
- Frontend loads successfully
- Backend returns SUCCESS + GUID
- End-to-end communication confirmed

## Phase 6.5 — Load Testing and Auto Scaling

Load test command:
siege -c 200 -t 10M http://ecs-fargate-cicd-alb-1845797115.us-east-1.elb.amazonaws.com

Results:
- CPU spiked to 99.9% under load
- Auto Scaling triggered at 50% CPU threshold
- ECS scaled frontend from 1 to 2 tasks automatically
- Application remained available during scaling
- Availability: 99.82%

Auto Scaling configuration:
- Minimum tasks: 1
- Maximum tasks: 4
- Scale out trigger: 50% CPU
- Scale out cooldown: 60 seconds
- Scale in cooldown: 300 seconds

## Key DevOps Principles

- SCM-first: Jenkinsfile and Dockerfiles version controlled in GitHub
- Infrastructure as Code: Terraform for reproducible infrastructure
- Automation: Jenkins CI/CD pipeline
- Containerization: Docker for consistency across environments
- Serverless compute: ECS with Fargate no EC2 nodes to manage
- Auto Scaling: Load tested and validated
