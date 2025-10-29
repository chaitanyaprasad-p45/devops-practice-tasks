# Deploying Frontend (FE) + Backend (BE) Application on Amazon EKS with ALB Ingress# Deploying Frontend (FE) + Backend (BE) Application on Amazon EKS with ALB Ingress



This guide provides a **step-by-step walkthrough** for deploying a sample Frontend and Backend application on **Amazon EKS (Elastic Kubernetes Service)**, exposed via an **Application Load Balancer (ALB)** managed by the **AWS Load Balancer Controller**.This guide provides a **step-by-step walkthrough** for deploying a sample Frontend and Backend application on **Amazon EKS (Elastic Kubernetes Service)**, exposed via an **Application Load Balancer (ALB)** managed by the **AWS Load Balancer Controller**.



------



## Table of Contents## Table of Contents



1. [Prerequisites](#1-prerequisites)1. [Prerequisites](#1-prerequisites)

2. [Create an EKS Cluster](#2-create-an-eks-cluster)2. [Create an EKS Cluster](#2-create-an-eks-cluster)

3. [Install AWS Load Balancer Controller](#3-install-aws-load-balancer-controller)3. [Install AWS Load Balancer Controller](#3-install-aws-load-balancer-controller)

4. [Deploy Backend and Frontend Applications](#4-deploy-backend-and-frontend-applications)4. [Deploy Backend and Frontend Applications](#4-deploy-backend-and-frontend-applications)

5. [Expose Application using ALB Ingress](#5-expose-application-using-alb-ingress)5. [Expose Application using ALB Ingress](#5-expose-application-using-alb-ingress)

6. [Verify ALB and Access Application](#6-verify-alb-and-access-application)6. [Verify ALB and Access Application](#6-verify-alb-and-access-application)

7. [Common Issues & Troubleshooting](#7-common-issues--troubleshooting)7. [Common Issues & Troubleshooting](#7-common-issues--troubleshooting)

8. [Next Steps & Best Practices](#8-next-steps--best-practices)8. [Next Steps & Best Practices](#8-next-steps--best-practices)



------



## 1. Prerequisites## 1. Prerequisites



Before you begin, ensure the following tools are installed and configured on your machine:Before you begin, ensure the following tools are installed and configured on your machine:



### Required Tools- AWS CLI (`aws configure`)

- `eksctl`

#### 1.1 AWS CLI- `kubectl`

The AWS Command Line Interface for interacting with AWS services.- `helm` (v3+)

- AWS Account with admin access or appropriate IAM permissions

**Download & Installation:**

- **Windows**: [Download AWS CLI MSI Installer](https://awscli.amazonaws.com/AWSCLIV2.msi)Ensure you’re authenticated to AWS and can list IAM roles and VPCs.

- **macOS**: 

  ```bash```bash

  curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"aws sts get-caller-identity

  sudo installer -pkg AWSCLIV2.pkg -target /aws ec2 describe-vpcs

  ``````

- **Linux**: 

  ```bash---

  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

  unzip awscliv2.zip## 2. Create an EKS Cluster

  sudo ./aws/install

  ```You can create an EKS cluster quickly with `eksctl`.

- **Documentation**: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```bash

**Verify Installation:**export AWS_REGION=us-west-2

```bash

aws --versioneksctl create cluster   --name demo-eks-alb   --region $AWS_REGION   --nodegroup-name demo-nodes   --node-type t3.medium   --nodes 2   --nodes-min 1   --nodes-max 3   --managed

``````



**Configure AWS CLI:**After creation:

```bash

aws configure```bash

# Enter your AWS Access Key ID, Secret Access Key, Region, and Output formatkubectl get nodes

``````



#### 1.2 eksctlEnable the OIDC provider for IAM integration (if not already enabled):

Command line tool for creating and managing EKS clusters.

```bash

**Download & Installation:**eksctl utils associate-iam-oidc-provider   --cluster demo-eks-alb   --approve

- **Windows (via Chocolatey)**:```

  ```powershell

  choco install eksctl---

  ```

- **Windows (Manual)**:## 3. Install AWS Load Balancer Controller

  ```powershell

  # Download from https://github.com/weaveworks/eksctl/releases/latest### 3.1 Create IAM Policy for the Controller

  # Extract and add to PATH

  ```Download the policy JSON from AWS docs and create the IAM policy:

- **macOS**:

  ```bash```bash

  brew tap weaveworks/tapcurl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

  brew install weaveworks/tap/eksctl

  ```aws iam create-policy   --policy-name AWSLoadBalancerControllerIAMPolicy   --policy-document file://iam-policy.json

- **Linux**:```

  ```bash

  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp### 3.2 Create IAM Service Account

  sudo mv /tmp/eksctl /usr/local/bin

  ``````bash

- **Documentation**: https://eksctl.io/installation/ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

CLUSTER_NAME=demo-eks-alb

**Verify Installation:**

```basheksctl create iamserviceaccount   --cluster $CLUSTER_NAME   --namespace kube-system   --name aws-load-balancer-controller   --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy   --override-existing-serviceaccounts   --approve

eksctl version```

```

### 3.3 Install the Controller via Helm

#### 1.3 kubectl

Kubernetes command-line tool for running commands against clusters.```bash

helm repo add eks https://aws.github.io/eks-charts

**Download & Installation:**helm repo update

- **Windows**:

  ```powershellhelm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller   -n kube-system   --set clusterName=$CLUSTER_NAME   --set serviceAccount.create=false   --set serviceAccount.name=aws-load-balancer-controller   --set region=$AWS_REGION

  curl.exe -LO "https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe"```

  # Add to PATH

  ```Verify installation:

- **macOS**:

  ```bash```bash

  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"kubectl get deployment -n kube-system aws-load-balancer-controller

  chmod +x kubectl```

  sudo mv kubectl /usr/local/bin/

  ```---

- **Linux**:

  ```bash## 4. Deploy Backend and Frontend Applications

  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

  chmod +x kubectlCreate a namespace and deploy the sample apps.

  sudo mv kubectl /usr/local/bin/

  ``````yaml

- **Documentation**: https://kubernetes.io/docs/tasks/tools/# app.yaml

apiVersion: v1

**Verify Installation:**kind: Namespace

```bashmetadata:

kubectl version --client  name: sample-app

```---

apiVersion: apps/v1

#### 1.4 Helmkind: Deployment

Package manager for Kubernetes (v3+).metadata:

  name: backend

**Download & Installation:**  namespace: sample-app

- **Windows (via Chocolatey)**:spec:

  ```powershell  replicas: 2

  choco install kubernetes-helm  selector:

  ```    matchLabels:

- **macOS**:      app: backend

  ```bash  template:

  brew install helm    metadata:

  ```      labels:

- **Linux**:        app: backend

  ```bash    spec:

  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash      containers:

  ```      - name: backend

- **Manual Download**: https://github.com/helm/helm/releases        image: hashicorp/http-echo:0.2.3

- **Documentation**: https://helm.sh/docs/intro/install/        args: ["-text", "Hello from backend"]

        ports:

**Verify Installation:**        - containerPort: 5678

```bash---

helm versionapiVersion: v1

```kind: Service

metadata:

### AWS Account Requirements  name: backend

  namespace: sample-app

- **AWS Account** with admin access or the following IAM permissions:spec:

  - EC2, EKS, VPC management  ports:

  - IAM role/policy creation  - port: 80

  - Elastic Load Balancing    targetPort: 5678

  - CloudFormation (for eksctl)  selector:

  - Auto Scaling    app: backend

  type: ClusterIP

### Verify AWS Authentication---

apiVersion: apps/v1

Ensure you're authenticated to AWS and can access required services:kind: Deployment

metadata:

```bash  name: frontend

# Verify AWS identity  namespace: sample-app

aws sts get-caller-identityspec:

  replicas: 2

# Check VPC access  selector:

aws ec2 describe-vpcs    matchLabels:

      app: frontend

# Verify region is set  template:

echo $AWS_REGION    metadata:

```      labels:

        app: frontend

### Quick Setup Script (Optional)    spec:

      containers:

For **Linux/macOS**, you can use this script to verify all prerequisites:      - name: frontend

        image: nginx:stable

```bash        ports:

#!/bin/bash        - containerPort: 80

echo "Checking prerequisites..."---

command -v aws >/dev/null 2>&1 && echo "✓ AWS CLI installed" || echo "✗ AWS CLI missing"apiVersion: v1

command -v eksctl >/dev/null 2>&1 && echo "✓ eksctl installed" || echo "✗ eksctl missing"kind: Service

command -v kubectl >/dev/null 2>&1 && echo "✓ kubectl installed" || echo "✗ kubectl missing"metadata:

command -v helm >/dev/null 2>&1 && echo "✓ Helm installed" || echo "✗ Helm missing"  name: frontend

aws sts get-caller-identity >/dev/null 2>&1 && echo "✓ AWS authenticated" || echo "✗ AWS authentication failed"  namespace: sample-app

```spec:

  selector:

---    app: frontend

  ports:

## 2. Create an EKS Cluster  - port: 80

    targetPort: 80

You can create an EKS cluster quickly with `eksctl`.  type: NodePort

```

```bash

export AWS_REGION=ap-south-1Apply it:



eksctl create cluster \```bash

  --name demo-eks-alb \kubectl apply -f app.yaml

  --region $AWS_REGION \kubectl -n sample-app get pods,svc

  --nodegroup-name demo-nodes \```

  --node-type t3.medium \

  --nodes 2 \---

  --nodes-min 1 \

  --nodes-max 3 \## 5. Expose Application using ALB Ingress

  --managed

```Create `ingress.yaml`:



After creation:```yaml

apiVersion: networking.k8s.io/v1

```bashkind: Ingress

kubectl get nodesmetadata:

```  name: frontend-ingress

  namespace: sample-app

Enable the OIDC provider for IAM integration (if not already enabled):  annotations:

    kubernetes.io/ingress.class: alb

```bash    alb.ingress.kubernetes.io/scheme: internet-facing

eksctl utils associate-iam-oidc-provider \    alb.ingress.kubernetes.io/target-type: ip

  --cluster demo-eks-alb \    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80}]'

  --approvespec:

```  rules:

    - http:

---        paths:

          - path: /

## 3. Install AWS Load Balancer Controller            pathType: Prefix

            backend:

### 3.1 Create IAM Policy for the Controller              service:

                name: frontend

Download the policy JSON from AWS docs and create the IAM policy:                port:

                  number: 80

```bash```

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

Apply:

aws iam create-policy \

  --policy-name AWSLoadBalancerControllerIAMPolicy \```bash

  --policy-document file://iam-policy.jsonkubectl apply -f ingress.yaml

``````



### 3.2 Create IAM Service Account---



```bash## 6. Verify ALB and Access Application

ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

CLUSTER_NAME=demo-eks-albCheck ingress:



eksctl create iamserviceaccount \```bash

  --cluster $CLUSTER_NAME \kubectl -n sample-app get ingress

  --namespace kube-system \```

  --name aws-load-balancer-controller \

  --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \You’ll see an **ADDRESS** (DNS name) like:

  --override-existing-serviceaccounts \

  --approve```

```k8s-sampleapp-frontend-1234567890.us-west-2.elb.amazonaws.com

```

### 3.3 Install the Controller via Helm

Visit that address in a browser.

```bash

helm repo add eks https://aws.github.io/eks-charts---

helm repo update

## 7. Common Issues & Troubleshooting

helm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller \

  -n kube-system \- **ALB not created:** Check controller logs:  

  --set clusterName=$CLUSTER_NAME \  `kubectl -n kube-system logs deployment/aws-load-balancer-controller`

  --set serviceAccount.create=false \- **Target group unhealthy:** Confirm health check path matches your service path.

  --set serviceAccount.name=aws-load-balancer-controller \- **IAM issues:** Verify the controller service account and OIDC provider.

  --set region=$AWS_REGION

```---



Verify installation:## 8. Next Steps & Best Practices



```bash- Use HTTPS (TLS) with ACM certificates (`alb.ingress.kubernetes.io/certificate-arn`).

kubectl get deployment -n kube-system aws-load-balancer-controller- Use IngressClassParams to standardize ALB settings.

```- Add autoscaling for your workloads.

- Use GitOps (ArgoCD / Flux) for deployment pipelines.

---

---

## 4. Deploy Backend and Frontend Applications


Create a namespace and deploy the sample apps.

```yaml
# app.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: sample-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo:0.2.3
        args: ["-text", "Hello from backend"]
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: sample-app
spec:
  ports:
  - port: 80
    targetPort: 5678
  selector:
    app: backend
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:stable
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: sample-app
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```

Apply it:

```bash
kubectl apply -f app.yaml
kubectl -n sample-app get pods,svc
```

---

## 5. Expose Application using ALB Ingress

Create `ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
  namespace: sample-app
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80}]'
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

---

## 6. Verify ALB and Access Application

Check ingress:

```bash
kubectl -n sample-app get ingress
```

You'll see an **ADDRESS** (DNS name) like:

```
k8s-sampleapp-frontend-1234567890.ap-south-1.elb.amazonaws.com
```

Visit that address in a browser.

---

## 7. Common Issues & Troubleshooting

- **ALB not created:** Check controller logs:  
  `kubectl -n kube-system logs deployment/aws-load-balancer-controller`
- **Target group unhealthy:** Confirm health check path matches your service path.
- **IAM issues:** Verify the controller service account and OIDC provider.

---

## 8. Next Steps & Best Practices

- Use HTTPS (TLS) with ACM certificates (`alb.ingress.kubernetes.io/certificate-arn`).
- Use IngressClassParams to standardize ALB settings.
- Add autoscaling for your workloads.
- Use GitOps (ArgoCD / Flux) for deployment pipelines.

---
