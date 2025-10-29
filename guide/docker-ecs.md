# Docker & Amazon ECS Beginner's Guide

## Table of Contents
1. [What is Docker?](#what-is-docker)
2. [Docker Fundamentals](#docker-fundamentals)
3. [Docker Installation](#docker-installation)
4. [Working with Docker Images](#working-with-docker-images)
5. [Working with Docker Containers](#working-with-docker-containers)
6. [Docker Networking](#docker-networking)
7. [Docker Volumes](#docker-volumes)
8. [Dockerfile Best Practices](#dockerfile-best-practices)
9. [Introduction to Amazon ECS](#introduction-to-amazon-ecs)
10. [ECS Core Concepts](#ecs-core-concepts)
11. [Setting Up ECS](#setting-up-ecs)
12. [Deploying Applications to ECS](#deploying-applications-to-ecs)
13. [ECS Best Practices](#ecs-best-practices)

## What is Docker?

Docker is a containerization platform that allows you to package applications and their dependencies into lightweight, portable containers. Think of containers as lightweight virtual machines that share the host operating system kernel.

### Why Use Docker?

**Before Docker:**
- "It works on my machine" problems
- Complex environment setup
- Inconsistent deployments
- Resource-heavy virtual machines

**With Docker:**
- Consistent environments across development, testing, and production
- Lightweight and fast
- Easy application deployment
- Simplified dependency management

### Containers vs Virtual Machines

**Virtual Machines:**
- Include full operating system
- Heavy resource usage
- Slower startup times
- Hardware virtualization

**Containers:**
- Share host operating system
- Lightweight resource usage
- Fast startup times
- OS-level virtualization

### Key Benefits
- **Portability**: Run anywhere Docker is installed
- **Consistency**: Same environment everywhere
- **Efficiency**: Use fewer resources than VMs
- **Scalability**: Easy to scale up or down
- **Isolation**: Applications don't interfere with each other

## Docker Fundamentals

### Core Concepts

**Image**: A read-only template used to create containers
- Contains application code, runtime, libraries, and dependencies
- Built from instructions in a Dockerfile
- Stored in registries like Docker Hub

**Container**: A running instance of an image
- Isolated process on the host machine
- Can be started, stopped, moved, and deleted
- Contains everything needed to run an application

**Dockerfile**: A text file with instructions to build an image
- Defines the environment and dependencies
- Used to automate image creation
- Version controlled with your code

**Registry**: A service for storing and distributing images
- Docker Hub is the default public registry
- Private registries for enterprise use
- Amazon ECR for AWS environments

### Docker Architecture

**Docker Client**: Command-line interface (CLI)
**Docker Daemon**: Background service managing containers
**Docker Registry**: Stores Docker images
**Docker Objects**: Images, containers, networks, volumes

## Docker Installation

### Windows Installation

1. **Install Docker Desktop**:
   - Download from [docker.com](https://www.docker.com/products/docker-desktop)
   - Run installer and follow instructions
   - Restart computer if prompted

2. **Verify Installation**:
```powershell
docker --version
docker run hello-world
```

### Linux Installation (Ubuntu)

```bash
# Update package index
sudo apt-get update

# Install required packages
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Verify installation
sudo docker run hello-world
```

### macOS Installation

1. Download Docker Desktop for Mac from docker.com
2. Install by dragging to Applications folder
3. Start Docker Desktop
4. Verify: `docker --version`

## Working with Docker Images

### Essential Image Commands

```bash
# Search for images
docker search nginx

# Pull an image from registry
docker pull nginx:latest
docker pull ubuntu:20.04

# List local images
docker images
docker image ls

# Remove an image
docker rmi nginx:latest
docker image rm ubuntu:20.04

# Remove unused images
docker image prune

# View image history
docker history nginx:latest

# Inspect image details
docker inspect nginx:latest
```

### Building Images

**Basic Dockerfile Example:**
```dockerfile
# Use official Python runtime as base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements file
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Define startup command
CMD ["python", "app.py"]
```

**Build Image:**
```bash
# Build image with tag
docker build -t my-python-app:1.0 .

# Build with different context
docker build -t my-app:latest -f Dockerfile.prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t my-app .
```

### Tagging and Pushing Images

```bash
# Tag an image
docker tag my-python-app:1.0 myusername/my-python-app:1.0

# Push to Docker Hub
docker login
docker push myusername/my-python-app:1.0

# Tag for different environments
docker tag my-app:latest my-app:production
docker tag my-app:latest my-app:staging
```

## Working with Docker Containers

### Essential Container Commands

```bash
# Run a container
docker run nginx
docker run -d nginx  # Run in background (detached)
docker run -p 8080:80 nginx  # Port mapping
docker run --name my-nginx nginx  # Custom name

# List running containers
docker ps
docker container ls

# List all containers (including stopped)
docker ps -a
docker container ls -a

# Stop a container
docker stop my-nginx
docker stop <container-id>

# Start a stopped container
docker start my-nginx

# Restart a container
docker restart my-nginx

# Remove a container
docker rm my-nginx
docker container rm <container-id>

# Remove all stopped containers
docker container prune
```

### Interactive Containers

```bash
# Run container with interactive terminal
docker run -it ubuntu:20.04 bash

# Execute command in running container
docker exec -it my-nginx bash

# Execute single command
docker exec my-nginx ls -la /usr/share/nginx/html

# Copy files to/from container
docker cp file.txt my-nginx:/tmp/
docker cp my-nginx:/tmp/file.txt ./local-file.txt
```

### Container Lifecycle Management

```bash
# View container logs
docker logs my-nginx
docker logs -f my-nginx  # Follow logs

# View container resource usage
docker stats
docker stats my-nginx

# Inspect container details
docker inspect my-nginx

# View running processes in container
docker top my-nginx

# Pause/unpause container
docker pause my-nginx
docker unpause my-nginx
```

## Docker Networking

### Network Types

**Bridge**: Default network for containers
**Host**: Container shares host network
**None**: No network access
**Custom**: User-defined networks

### Network Commands

```bash
# List networks
docker network ls

# Create custom network
docker network create my-network
docker network create --driver bridge my-bridge-network

# Connect container to network
docker run --network my-network nginx

# Connect running container to network
docker network connect my-network my-nginx

# Disconnect container from network
docker network disconnect my-network my-nginx

# Inspect network
docker network inspect my-network

# Remove network
docker network rm my-network
```

### Example: Multi-Container Application

```bash
# Create network
docker network create app-network

# Run database container
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  postgres:13

# Run application container
docker run -d \
  --name my-app \
  --network app-network \
  -p 8000:8000 \
  -e DATABASE_URL=postgresql://user:password@postgres-db:5432/myapp \
  my-python-app:1.0
```

## Docker Volumes

### Types of Storage

**Volumes**: Managed by Docker (recommended)
**Bind Mounts**: Map host directory to container
**tmpfs**: Store in host memory (temporary)

### Volume Commands

```bash
# Create volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Remove unused volumes
docker volume prune
```

### Using Volumes

```bash
# Named volume
docker run -d \
  --name nginx-with-volume \
  -v my-volume:/usr/share/nginx/html \
  nginx

# Bind mount (absolute path required)
docker run -d \
  --name nginx-bind-mount \
  -v /host/path:/usr/share/nginx/html \
  nginx

# Read-only volume
docker run -d \
  --name nginx-readonly \
  -v my-volume:/usr/share/nginx/html:ro \
  nginx
```

### Example: Database with Persistent Storage

```bash
# Create volume for database data
docker volume create postgres-data

# Run PostgreSQL with persistent storage
docker run -d \
  --name postgres-persistent \
  -v postgres-data:/var/lib/postgresql/data \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  postgres:13
```

## Dockerfile Best Practices

### Multi-Stage Builds

```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:16-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Optimization Techniques

```dockerfile
# Use specific version tags
FROM python:3.9-slim

# Set working directory early
WORKDIR /app

# Install system dependencies first (better caching)
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copy and install dependencies before copying code
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code last
COPY . .

# Use non-root user
RUN useradd --create-home --shell /bin/bash app
USER app

# Use COPY instead of ADD
COPY src/ ./src/

# Combine RUN commands
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Use .dockerignore
# Create .dockerignore file:
# node_modules
# .git
# .env
# *.md
```

### Security Best Practices

```dockerfile
# Use official base images
FROM python:3.9-slim

# Update packages
RUN apt-get update && apt-get upgrade -y

# Don't run as root
RUN groupadd -r appgroup && useradd -r -g appgroup appuser
USER appuser

# Use COPY instead of ADD
COPY requirements.txt .

# Don't store secrets in images
# Use environment variables or secrets management

# Minimize image layers
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean
```

## Introduction to Amazon ECS

### What is Amazon ECS?

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that makes it easy to deploy, manage, and scale containerized applications using Docker containers.

### Key Benefits of ECS

- **Fully Managed**: No infrastructure to manage
- **Scalable**: Automatic scaling based on demand
- **Secure**: Integrated with AWS security services
- **Cost-Effective**: Pay only for what you use
- **Integrated**: Works seamlessly with other AWS services

### ECS vs Other Container Services

**ECS vs Kubernetes**:
- ECS: Simpler, AWS-native, less learning curve
- Kubernetes: More features, portable, steeper learning curve

**ECS vs Fargate**:
- ECS: You manage EC2 instances
- Fargate: Serverless, AWS manages infrastructure

## ECS Core Concepts

### Clusters
A logical grouping of EC2 instances or Fargate capacity that you can place tasks on.

```bash
# Create cluster
aws ecs create-cluster --cluster-name my-cluster
```

### Task Definitions
A blueprint that describes how containers should run, including:
- Container images
- CPU and memory requirements
- Networking configuration
- Environment variables

### Tasks
A running instance of a task definition

### Services
Maintain desired number of tasks running and replace unhealthy tasks

### Container Instances
EC2 instances running the ECS agent (for EC2 launch type)

## Setting Up ECS

### Prerequisites

1. **AWS Account**: Active AWS account
2. **AWS CLI**: Installed and configured
3. **Docker Images**: Application containerized
4. **ECR Repository**: Store Docker images

### Step 1: Create ECR Repository

```bash
# Create ECR repository
aws ecr create-repository --repository-name my-app

# Get login token
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com

# Tag and push image
docker tag my-app:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
```

### Step 2: Create Task Definition

```json
{
  "family": "my-app-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "my-app",
      "image": "123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest",
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### Step 3: Create ECS Cluster

```bash
# Create Fargate cluster
aws ecs create-cluster --cluster-name my-fargate-cluster

# Or create EC2 cluster
aws ecs create-cluster --cluster-name my-ec2-cluster
```

### Step 4: Create Service

```json
{
  "serviceName": "my-app-service",
  "cluster": "my-fargate-cluster",
  "taskDefinition": "my-app-task:1",
  "desiredCount": 2,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": [
        "subnet-12345678",
        "subnet-87654321"
      ],
      "securityGroups": [
        "sg-abcdef01"
      ],
      "assignPublicIp": "ENABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-targets/1234567890123456",
      "containerName": "my-app",
      "containerPort": 8000
    }
  ]
}
```

## Deploying Applications to ECS

### Complete Deployment Example

**1. Prepare Application**
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 8000

CMD ["python", "app.py"]
```

**2. Build and Push to ECR**
```bash
# Build image
docker build -t my-python-app .

# Tag for ECR
docker tag my-python-app:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-python-app:latest

# Push to ECR
docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-python-app:latest
```

**3. Create Task Definition**
```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

**4. Create Service**
```bash
aws ecs create-service --cli-input-json file://service-definition.json
```

**5. Verify Deployment**
```bash
# Check service status
aws ecs describe-services --cluster my-fargate-cluster --services my-app-service

# Check running tasks
aws ecs list-tasks --cluster my-fargate-cluster --service-name my-app-service

# Check task details
aws ecs describe-tasks --cluster my-fargate-cluster --tasks <task-arn>
```

### CI/CD Pipeline for ECS

**GitHub Actions Example:**
```yaml
name: Deploy to ECS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
      
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2
        
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1
      
    - name: Build, tag, and push image to Amazon ECR
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: my-app
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        
    - name: Deploy to Amazon ECS
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: task-definition.json
        service: my-app-service
        cluster: my-fargate-cluster
        wait-for-service-stability: true
```

### Blue/Green Deployment with ECS

```bash
# Create new task definition revision
aws ecs register-task-definition --cli-input-json file://task-definition-v2.json

# Update service to use new task definition
aws ecs update-service \
  --cluster my-fargate-cluster \
  --service my-app-service \
  --task-definition my-app-task:2 \
  --desired-count 2

# Monitor deployment
aws ecs wait services-stable \
  --cluster my-fargate-cluster \
  --services my-app-service
```

## ECS Best Practices

### Security Best Practices

1. **Use IAM Roles**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

2. **Network Security**
- Use VPC for network isolation
- Configure security groups properly
- Use private subnets for backend services

3. **Secrets Management**
```json
{
  "name": "my-app",
  "secrets": [
    {
      "name": "DB_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:us-west-2:123456789012:secret:db-password-AbCdEf"
    }
  ]
}
```

### Performance Optimization

1. **Right-size Resources**
```json
{
  "cpu": "256",
  "memory": "512",
  "memoryReservation": "256"
}
```

2. **Use Health Checks**
```json
{
  "healthCheck": {
    "command": [
      "CMD-SHELL",
      "curl -f http://localhost:8000/health || exit 1"
    ],
    "interval": 30,
    "timeout": 5,
    "retries": 3,
    "startPeriod": 60
  }
}
```

3. **Configure Auto Scaling**
```bash
# Create auto scaling target
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/my-fargate-cluster/my-app-service \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 1 \
  --max-capacity 10

# Create scaling policy
aws application-autoscaling put-scaling-policy \
  --policy-name my-app-scaling-policy \
  --service-namespace ecs \
  --resource-id service/my-fargate-cluster/my-app-service \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://scaling-policy.json
```

### Monitoring and Logging

1. **CloudWatch Logs**
```json
{
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/my-app",
      "awslogs-region": "us-west-2",
      "awslogs-stream-prefix": "ecs"
    }
  }
}
```

2. **CloudWatch Metrics**
- CPU utilization
- Memory utilization
- Network I/O
- Task count

3. **Application Monitoring**
- Use APM tools (New Relic, DataDog)
- Custom metrics with CloudWatch
- Health check endpoints

### Cost Optimization

1. **Use Fargate Spot**
```json
{
  "capacityProviders": ["FARGATE_SPOT"],
  "defaultCapacityProviderStrategy": [
    {
      "capacityProvider": "FARGATE_SPOT",
      "weight": 1
    }
  ]
}
```

2. **Right-size Containers**
- Monitor resource usage
- Adjust CPU and memory allocations
- Use container insights

3. **Optimize Images**
- Use multi-stage builds
- Minimize image size
- Use alpine-based images

### Troubleshooting Common Issues

**Task Won't Start**
```bash
# Check task definition
aws ecs describe-task-definition --task-definition my-app-task

# Check service events
aws ecs describe-services --cluster my-fargate-cluster --services my-app-service

# Check task logs
aws logs get-log-events --log-group-name /ecs/my-app --log-stream-name ecs/my-app/task-id
```

**Service Unhealthy**
```bash
# Check load balancer health
aws elbv2 describe-target-health --target-group-arn <target-group-arn>

# Check container health
docker ps
docker logs <container-id>
```

## Next Steps

1. **Practice with Simple Applications**
   - Containerize a web application
   - Deploy to ECS using Fargate

2. **Learn Advanced Features**
   - Service mesh with App Mesh
   - Blue/green deployments
   - Capacity providers

3. **Integrate with CI/CD**
   - Set up automated deployments
   - Implement proper testing

4. **Monitor and Optimize**
   - Set up comprehensive monitoring
   - Optimize costs and performance

5. **Study for Certifications**
   - AWS Certified Developer
   - AWS Certified DevOps Engineer

Remember: Start with simple examples and gradually build complexity as you become more comfortable with Docker and ECS concepts!