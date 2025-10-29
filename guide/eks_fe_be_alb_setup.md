# Deploying Frontend (FE) + Backend (BE) Application on Amazon EKS with ALB Ingress

This guide provides a **step-by-step walkthrough** for deploying a sample Frontend and Backend application on **Amazon EKS (Elastic Kubernetes Service)**, exposed via an **Application Load Balancer (ALB)** managed by the **AWS Load Balancer Controller**.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Create an EKS Cluster](#2-create-an-eks-cluster)
3. [Install AWS Load Balancer Controller](#3-install-aws-load-balancer-controller)
4. [Deploy Backend and Frontend Applications](#4-deploy-backend-and-frontend-applications)
5. [Expose Application using ALB Ingress](#5-expose-application-using-alb-ingress)
6. [Verify ALB and Access Application](#6-verify-alb-and-access-application)
7. [Common Issues & Troubleshooting](#7-common-issues--troubleshooting)
8. [Next Steps & Best Practices](#8-next-steps--best-practices)

---

## 1. Prerequisites

Before you begin, ensure the following tools are installed and configured on your machine:

- AWS CLI (`aws configure`)
- `eksctl`
- `kubectl`
- `helm` (v3+)
- AWS Account with admin access or appropriate IAM permissions

Ensure you’re authenticated to AWS and can list IAM roles and VPCs.

```bash
aws sts get-caller-identity
aws ec2 describe-vpcs
```

---

## 2. Create an EKS Cluster

You can create an EKS cluster quickly with `eksctl`.

```bash
export AWS_REGION=us-west-2

eksctl create cluster   --name demo-eks-alb   --region $AWS_REGION   --nodegroup-name demo-nodes   --node-type t3.medium   --nodes 2   --nodes-min 1   --nodes-max 3   --managed
```

After creation:

```bash
kubectl get nodes
```

Enable the OIDC provider for IAM integration (if not already enabled):

```bash
eksctl utils associate-iam-oidc-provider   --cluster demo-eks-alb   --approve
```

---

## 3. Install AWS Load Balancer Controller

### 3.1 Create IAM Policy for the Controller

Download the policy JSON from AWS docs and create the IAM policy:

```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy   --policy-name AWSLoadBalancerControllerIAMPolicy   --policy-document file://iam-policy.json
```

### 3.2 Create IAM Service Account

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
CLUSTER_NAME=demo-eks-alb

eksctl create iamserviceaccount   --cluster $CLUSTER_NAME   --namespace kube-system   --name aws-load-balancer-controller   --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy   --override-existing-serviceaccounts   --approve
```

### 3.3 Install the Controller via Helm

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller   -n kube-system   --set clusterName=$CLUSTER_NAME   --set serviceAccount.create=false   --set serviceAccount.name=aws-load-balancer-controller   --set region=$AWS_REGION
```

Verify installation:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

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

You’ll see an **ADDRESS** (DNS name) like:

```
k8s-sampleapp-frontend-1234567890.us-west-2.elb.amazonaws.com
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

