# Kubernetes & Amazon EKS Beginner's Guide

## Table of Contents
1. [What is Kubernetes?](#what-is-kubernetes)
2. [Kubernetes Architecture](#kubernetes-architecture)
3. [Core Kubernetes Concepts](#core-kubernetes-concepts)
4. [Kubernetes Installation](#kubernetes-installation)
5. [Working with Pods](#working-with-pods)
6. [Services and Networking](#services-and-networking)
7. [Deployments and ReplicaSets](#deployments-and-replicasets)
8. [ConfigMaps and Secrets](#configmaps-and-secrets)
9. [Persistent Volumes](#persistent-volumes)
10. [Introduction to Amazon EKS](#introduction-to-amazon-eks)
11. [Setting Up EKS](#setting-up-eks)
12. [Deploying Applications to EKS](#deploying-applications-to-eks)
13. [EKS Best Practices](#eks-best-practices)

## What is Kubernetes?

Kubernetes (often abbreviated as K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. Think of it as an operating system for your containerized applications.

### Why Kubernetes?

**Before Kubernetes:**
- Manual container management
- Difficult scaling
- No built-in load balancing
- Complex service discovery
- Manual health monitoring

**With Kubernetes:**
- Automated container management
- Automatic scaling
- Built-in load balancing
- Service discovery
- Self-healing capabilities

### Key Benefits

- **Scalability**: Automatically scale applications up or down
- **High Availability**: Ensure applications are always running
- **Portability**: Run on any cloud or on-premises
- **Self-healing**: Automatically restart failed containers
- **Rolling Updates**: Deploy new versions without downtime
- **Resource Optimization**: Efficient resource utilization

### Kubernetes vs Docker

**Docker**: Containerization platform
**Kubernetes**: Container orchestration platform

Docker creates containers, Kubernetes manages them at scale.

## Kubernetes Architecture

### Master Node Components

**API Server**: Entry point for all REST commands
**etcd**: Distributed key-value store for cluster data
**Scheduler**: Assigns pods to nodes
**Controller Manager**: Manages controllers and cluster state

### Worker Node Components

**kubelet**: Node agent that communicates with master
**kube-proxy**: Network proxy for service communication
**Container Runtime**: Docker, containerd, or CRI-O

### Add-on Components

**DNS**: Service discovery within cluster
**Dashboard**: Web-based UI
**Ingress Controller**: Manages external access
**Monitoring**: Metrics collection and alerting

```
┌─────────────────────────────────────────────────────────┐
│                    Master Node                         │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────┐   │
│  │ API Server  │ │    etcd     │ │   Scheduler     │   │
│  └─────────────┘ └─────────────┘ └─────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │            Controller Manager                   │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                            │
                            │
    ┌───────────────────────┼───────────────────────┐
    │                       │                       │
┌───▼────┐              ┌───▼────┐              ┌───▼────┐
│Worker 1│              │Worker 2│              │Worker 3│
│        │              │        │              │        │
│kubelet │              │kubelet │              │kubelet │
│kube-   │              │kube-   │              │kube-   │
│proxy   │              │proxy   │              │proxy   │
│        │              │        │              │        │
│  Pods  │              │  Pods  │              │  Pods  │
└────────┘              └────────┘              └────────┘
```

## Core Kubernetes Concepts

### Namespaces
Logical separation of resources within a cluster

```bash
# List namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace development
kubectl create namespace production

# Set default namespace
kubectl config set-context --current --namespace=development
```

### Labels and Selectors
Key-value pairs used to organize and select resources

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: web
    environment: production
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.20
```

```bash
# Select pods by label
kubectl get pods -l app=web
kubectl get pods -l environment=production,tier=frontend
```

### Annotations
Non-identifying metadata attached to objects

```yaml
metadata:
  annotations:
    description: "Frontend web server"
    maintainer: "team@company.com"
    version: "1.2.3"
```

## Kubernetes Installation

### Local Development Options

**1. Minikube** (Recommended for beginners)
```bash
# Install minikube (Windows with Chocolatey)
choco install minikube

# Start minikube
minikube start

# Check status
minikube status

# Access dashboard
minikube dashboard
```

**2. Docker Desktop**
- Enable Kubernetes in Docker Desktop settings
- Built-in single-node cluster

**3. Kind (Kubernetes in Docker)**
```bash
# Install kind
go install sigs.k8s.io/kind@latest

# Create cluster
kind create cluster --name my-cluster

# Delete cluster
kind delete cluster --name my-cluster
```

### kubectl Installation

```bash
# Windows (Chocolatey)
choco install kubernetes-cli

# Verify installation
kubectl version --client

# Configure kubectl for minikube
kubectl config use-context minikube
```

### Basic kubectl Commands

```bash
# Cluster information
kubectl cluster-info
kubectl get nodes

# API resources
kubectl api-resources
kubectl api-versions

# Help
kubectl help
kubectl explain pods
```

## Working with Pods

### What is a Pod?
A Pod is the smallest deployable unit in Kubernetes, containing one or more containers that share storage and network.

### Creating Pods

**Simple Pod YAML:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.20
    ports:
    - containerPort: 80
```

**Multi-container Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: web
    image: nginx:1.20
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'tail -f /var/log/nginx/access.log']
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  volumes:
  - name: shared-logs
    emptyDir: {}
```

### Pod Operations

```bash
# Create pod from YAML
kubectl apply -f pod.yaml

# Create pod imperatively
kubectl run nginx --image=nginx:1.20 --port=80

# List pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels

# Describe pod
kubectl describe pod nginx-pod

# View pod logs
kubectl logs nginx-pod
kubectl logs nginx-pod -c container-name  # Multi-container pod

# Execute commands in pod
kubectl exec -it nginx-pod -- bash
kubectl exec nginx-pod -- ls -la

# Port forwarding
kubectl port-forward nginx-pod 8080:80

# Delete pod
kubectl delete pod nginx-pod
kubectl delete -f pod.yaml
```

### Pod Lifecycle

**Pending**: Pod accepted but not yet running
**Running**: Pod bound to node and containers created
**Succeeded**: All containers terminated successfully
**Failed**: At least one container failed
**Unknown**: Pod state cannot be determined

## Services and Networking

### What are Services?
Services provide stable network endpoints for accessing pods, as pods are ephemeral and their IPs change.

### Service Types

**ClusterIP** (Default): Internal cluster access only
**NodePort**: Exposes service on each node's IP
**LoadBalancer**: External load balancer (cloud provider)
**ExternalName**: Maps service to external DNS name

### ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### Service Operations

```bash
# Create service
kubectl apply -f service.yaml

# Create service imperatively
kubectl expose pod nginx-pod --port=80 --type=NodePort

# List services
kubectl get services
kubectl get svc

# Describe service
kubectl describe service nginx-service

# Service endpoints
kubectl get endpoints nginx-service

# Delete service
kubectl delete service nginx-service
```

### Ingress

Manages external access to services, typically HTTP/HTTPS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

## Deployments and ReplicaSets

### What is a Deployment?
Deployments manage ReplicaSets and provide declarative updates to pods with rollback capabilities.

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Deployment Operations

```bash
# Create deployment
kubectl apply -f deployment.yaml

# Create deployment imperatively
kubectl create deployment nginx --image=nginx:1.20 --replicas=3

# List deployments
kubectl get deployments
kubectl get deploy

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Update deployment image
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Rollout status
kubectl rollout status deployment/nginx-deployment

# Rollback deployment
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Delete deployment
kubectl delete deployment nginx-deployment
```

### ReplicaSets

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
```

```bash
# List ReplicaSets
kubectl get replicasets
kubectl get rs

# Describe ReplicaSet
kubectl describe rs nginx-replicaset
```

### Rolling Updates and Rollbacks

```bash
# Update strategy in deployment
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1

# Pause rollout
kubectl rollout pause deployment/nginx-deployment

# Resume rollout
kubectl rollout resume deployment/nginx-deployment
```

## ConfigMaps and Secrets

### ConfigMaps
Store configuration data as key-value pairs

**Creating ConfigMaps:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://localhost:5432/myapp"
  debug_mode: "true"
  max_connections: "100"
```

```bash
# Create from literal
kubectl create configmap app-config \
  --from-literal=database_url=postgresql://localhost:5432/myapp \
  --from-literal=debug_mode=true

# Create from file
kubectl create configmap app-config --from-file=config.properties

# Create from directory
kubectl create configmap app-config --from-file=configs/
```

**Using ConfigMaps in Pods:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### Secrets
Store sensitive data like passwords and tokens

**Creating Secrets:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded 'admin'
  password: MWYyZDFlMmU2N2Rm  # base64 encoded password
```

```bash
# Create secret imperatively
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Create TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key

# Create docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=my-registry.com \
  --docker-username=user \
  --docker-password=password \
  --docker-email=user@example.com
```

**Using Secrets in Pods:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
  imagePullSecrets:
  - name: regcred
```

## Persistent Volumes

### Storage in Kubernetes

**Volumes**: Tied to pod lifecycle
**Persistent Volumes (PV)**: Cluster-level storage resource
**Persistent Volume Claims (PVC)**: Request for storage by pods

### Persistent Volume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/data/pv-storage"
```

### Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

### Using PVC in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-claim
```

### Storage Classes

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  replication-type: none
allowVolumeExpansion: true
```

```bash
# List storage classes
kubectl get storageclass
kubectl get sc

# List persistent volumes
kubectl get pv

# List persistent volume claims
kubectl get pvc
```

## Introduction to Amazon EKS

### What is Amazon EKS?

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service that eliminates the need to install, operate, and maintain your own Kubernetes control plane.

### Benefits of EKS

- **Fully Managed**: AWS manages the control plane
- **Highly Available**: Multi-AZ control plane
- **Secure**: Integrated with AWS security services
- **Compliant**: SOC, PCI, ISO, FedRAMP compliant
- **Integrated**: Works with AWS services
- **Open Source**: Standard Kubernetes APIs

### EKS vs Self-Managed Kubernetes

**EKS Benefits:**
- No control plane management
- Automatic updates and patches
- Built-in security
- AWS integration
- High availability

**Self-Managed Challenges:**
- Complex setup and maintenance
- Security configuration
- Upgrade management
- High availability setup

## Setting Up EKS

### Prerequisites

1. **AWS CLI**: Installed and configured
2. **kubectl**: Kubernetes command-line tool
3. **eksctl**: EKS management tool
4. **IAM Permissions**: EKS and related service permissions

### Installing Required Tools

```bash
# Install AWS CLI (Windows)
# Download from: https://aws.amazon.com/cli/

# Install kubectl
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/windows/amd64/kubectl.exe

# Install eksctl (Windows)
choco install eksctl

# Verify installations
aws --version
kubectl version --client
eksctl version
```

### Creating EKS Cluster with eksctl

**Simple Cluster:**
```bash
# Create cluster
eksctl create cluster \
  --name my-cluster \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed

# Update kubeconfig
aws eks update-kubeconfig --region us-west-2 --name my-cluster

# Verify cluster
kubectl get nodes
```

**Advanced Cluster Configuration:**
```yaml
# cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: advanced-cluster
  region: us-west-2
  version: "1.21"

vpc:
  cidr: "10.0.0.0/16"
  nat:
    gateway: Single

iam:
  withOIDC: true

managedNodeGroups:
  - name: ng-1
    instanceType: t3.medium
    minSize: 1
    maxSize: 5
    desiredCapacity: 3
    ssh:
      allow: true
    labels:
      role: worker
    tags:
      Environment: development
    iam:
      withAddonPolicies:
        autoScaler: true
        ebs: true
        efs: true
        albIngress: true
        cloudWatch: true

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-ebs-csi-driver
```

```bash
# Create cluster from config
eksctl create cluster -f cluster.yaml
```

### Manual EKS Setup (AWS CLI)

**1. Create Service Role:**
```bash
# Create IAM role for EKS
aws iam create-role \
  --role-name eksServiceRole \
  --assume-role-policy-document file://eks-service-role-trust-policy.json

# Attach policies
aws iam attach-role-policy \
  --role-name eksServiceRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```

**2. Create VPC:**
```bash
# Create VPC stack
aws cloudformation create-stack \
  --stack-name eks-vpc \
  --template-url https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
```

**3. Create EKS Cluster:**
```bash
# Create cluster
aws eks create-cluster \
  --name my-cluster \
  --version 1.21 \
  --role-arn arn:aws:iam::123456789012:role/eksServiceRole \
  --resources-vpc-config subnetIds=subnet-12345,subnet-67890,securityGroupIds=sg-abcdef
```

**4. Create Node Group:**
```bash
# Create node group
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name standard-workers \
  --node-role arn:aws:iam::123456789012:role/NodeInstanceRole \
  --subnets subnet-12345 subnet-67890 \
  --instance-types t3.medium \
  --scaling-config minSize=1,maxSize=4,desiredSize=3
```

### Cluster Access and RBAC

```bash
# Update kubeconfig
aws eks update-kubeconfig --region us-west-2 --name my-cluster

# Test access
kubectl get svc

# View AWS authentication
kubectl get configmap aws-auth -n kube-system -o yaml
```

**Adding IAM Users/Roles:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789012:role/NodeInstanceRole
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
  mapUsers: |
    - userarn: arn:aws:iam::123456789012:user/developer
      username: developer
      groups:
        - system:masters
```

## Deploying Applications to EKS

### Complete Application Deployment

**1. Prepare Application**
```dockerfile
# Dockerfile
FROM node:16-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
EXPOSE 3000

USER node
CMD ["node", "server.js"]
```

**2. Build and Push to ECR**
```bash
# Create ECR repository
aws ecr create-repository --repository-name my-node-app

# Get login token
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com

# Build and push
docker build -t my-node-app .
docker tag my-node-app:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-node-app:latest
docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-node-app:latest
```

**3. Kubernetes Manifests**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-node-app
  labels:
    app: my-node-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-node-app
  template:
    metadata:
      labels:
        app: my-node-app
    spec:
      containers:
      - name: my-node-app
        image: 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-node-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: my-node-app-service
spec:
  selector:
    app: my-node-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

**4. Deploy Application**
```bash
# Apply manifests
kubectl apply -f deployment.yaml

# Check deployment status
kubectl get deployments
kubectl get pods
kubectl get services

# Get external IP
kubectl get service my-node-app-service
```

### Using Helm for Package Management

**Install Helm:**
```bash
# Install Helm (Windows)
choco install kubernetes-helm

# Add Helm repositories
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

**Deploy with Helm:**
```bash
# Install NGINX Ingress Controller
helm install nginx-ingress ingress-nginx/ingress-nginx

# Install monitoring stack
helm install prometheus prometheus-community/kube-prometheus-stack

# Install application
helm install my-app ./my-app-chart
```

**Create Helm Chart:**
```bash
# Create chart
helm create my-app

# Chart structure
my-app/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
    ingress.yaml
```

### CI/CD Pipeline for EKS

**GitHub Actions Example:**
```yaml
name: Deploy to EKS

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
      
    - name: Build, tag, and push image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: my-node-app
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        
    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --name my-cluster --region us-west-2
        
    - name: Deploy to EKS
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: my-node-app
        IMAGE_TAG: ${{ github.sha }}
      run: |
        sed -i.bak "s|IMAGE_TAG|$IMAGE_TAG|g" k8s/deployment.yaml
        kubectl apply -f k8s/
        kubectl rollout status deployment/my-node-app
```

## EKS Best Practices

### Security Best Practices

**1. RBAC Configuration**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "create", "update", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: development
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

**2. Network Policies**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**3. Pod Security Standards**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: my-app:1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

### Monitoring and Logging

**1. CloudWatch Container Insights**
```bash
# Install CloudWatch agent
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml

kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-daemonset.yaml
```

**2. Prometheus and Grafana**
```bash
# Install using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack

# Access Grafana
kubectl port-forward svc/prometheus-grafana 3000:80
```

**3. Fluentd for Logging**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-cloudwatch
spec:
  selector:
    matchLabels:
      name: fluentd-cloudwatch
  template:
    metadata:
      labels:
        name: fluentd-cloudwatch
    spec:
      containers:
      - name: fluentd-cloudwatch
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-cloudwatch
        env:
        - name: AWS_REGION
          value: us-west-2
        - name: LOG_GROUP_NAME
          value: /aws/eks/cluster-name/containers
```

### Auto Scaling

**1. Horizontal Pod Autoscaler (HPA)**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**2. Vertical Pod Autoscaler (VPA)**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: my-app
      maxAllowed:
        cpu: 1
        memory: 500Mi
      minAllowed:
        cpu: 100m
        memory: 50Mi
```

**3. Cluster Autoscaler**
```bash
# Install cluster autoscaler
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

# Annotate deployment
kubectl annotate deployment.apps/cluster-autoscaler \
  cluster-autoscaler.kubernetes.io/safe-to-evict="false" \
  -n kube-system
```

### Cost Optimization

**1. Resource Requests and Limits**
```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**2. Node Group Optimization**
- Use Spot instances for non-critical workloads
- Mix instance types and sizes
- Use managed node groups
- Enable cluster autoscaler

**3. Storage Optimization**
- Use appropriate storage classes
- Enable volume encryption
- Monitor storage usage
- Clean up unused volumes

### Backup and Disaster Recovery

**1. Velero for Backup**
```bash
# Install Velero
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.2.0 \
  --bucket my-velero-backups \
  --backup-location-config region=us-west-2 \
  --snapshot-location-config region=us-west-2

# Create backup
velero backup create my-backup --include-namespaces production

# Restore from backup
velero restore create --from-backup my-backup
```

**2. Multi-Region Setup**
- Deploy clusters in multiple regions
- Use cross-region replication for data
- Implement traffic routing between regions
- Regular disaster recovery testing

## Advanced Topics

### Service Mesh with Istio

```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
istioctl install --set values.defaultRevision=default

# Enable sidecar injection
kubectl label namespace default istio-injection=enabled

# Deploy application with Istio
kubectl apply -f <(istioctl kube-inject -f deployment.yaml)
```

### GitOps with ArgoCD

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Operators and Custom Resources

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
              size:
                type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

## Troubleshooting

### Common Issues

**Pods not starting:**
```bash
# Check pod status
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

**Network connectivity:**
```bash
# Test DNS resolution
kubectl run test-pod --image=busybox -it --rm -- nslookup kubernetes.default

# Test service connectivity
kubectl run test-pod --image=busybox -it --rm -- wget -qO- http://service-name.namespace.svc.cluster.local
```

**Resource issues:**
```bash
# Check node resources
kubectl top nodes
kubectl describe node <node-name>

# Check pod resources
kubectl top pods
kubectl describe pod <pod-name>
```

### Debugging Tools

```bash
# Interactive debugging pod
kubectl run debug --image=busybox -it --rm -- sh

# Debug with network tools
kubectl run netshoot --image=nicolaka/netshoot -it --rm -- bash

# Copy files from pod
kubectl cp <pod-name>:/path/to/file ./local-file

# Port forward for debugging
kubectl port-forward pod/<pod-name> 8080:80
```

## Next Steps

1. **Master the Basics**
   - Practice with local Kubernetes cluster
   - Deploy various types of applications
   - Understand networking and storage

2. **Advanced Kubernetes**
   - Custom Resource Definitions (CRDs)
   - Operators development
   - Service mesh (Istio/Linkerd)

3. **Production Readiness**
   - Security hardening
   - Monitoring and alerting
   - Backup and disaster recovery

4. **Ecosystem Tools**
   - Helm package management
   - GitOps with ArgoCD
   - Policy management with OPA/Gatekeeper

5. **Certifications**
   - Certified Kubernetes Administrator (CKA)
   - Certified Kubernetes Application Developer (CKAD)
   - Certified Kubernetes Security Specialist (CKS)

Remember: Kubernetes has a steep learning curve, but start with basics and gradually build complexity. Practice regularly and focus on understanding core concepts before moving to advanced topics!