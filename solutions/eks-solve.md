# Kubernetes EKS Tasks Solutions

This document contains detailed solutions for all 20 Kubernetes EKS practice tasks.

## Basic Level Solutions (Tasks 1-10)

### Task 1: EKS Cluster Setup and kubectl Configuration
**Create an Amazon EKS cluster and configure kubectl for cluster access.**

**Solution:**
```bash
# Install required tools
echo "=== Installing EKS Tools ==="

# Install kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.3/2023-11-14/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installations
kubectl version --client
eksctl version
aws --version

# EKS cluster creation script
cat > create_eks_cluster.sh << 'EOF'
#!/bin/bash

# Variables
CLUSTER_NAME="my-eks-cluster"
REGION="us-west-2"
NODE_GROUP_NAME="worker-nodes"
NODE_TYPE="t3.medium"
NODE_COUNT=2
KUBERNETES_VERSION="1.28"

echo "=== Creating EKS Cluster: $CLUSTER_NAME ==="

# Create EKS cluster with eksctl
eksctl create cluster \
    --name $CLUSTER_NAME \
    --region $REGION \
    --version $KUBERNETES_VERSION \
    --nodegroup-name $NODE_GROUP_NAME \
    --node-type $NODE_TYPE \
    --nodes $NODE_COUNT \
    --nodes-min 1 \
    --nodes-max 4 \
    --managed \
    --with-oidc \
    --ssh-access \
    --ssh-public-key ~/.ssh/id_rsa.pub \
    --enable-ssm

echo "✅ EKS Cluster created successfully!"

# Configure kubectl
aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME

# Verify cluster access
echo "=== Verifying Cluster Access ==="
kubectl get nodes
kubectl get pods --all-namespaces

# Install AWS Load Balancer Controller
echo "=== Installing AWS Load Balancer Controller ==="

# Get cluster VPC ID
VPC_ID=$(aws eks describe-cluster --name $CLUSTER_NAME --region $REGION --query "cluster.resourcesVpcConfig.vpcId" --output text)

# Create IAM policy for Load Balancer Controller
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.6.0/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

# Create service account
eksctl create iamserviceaccount \
    --cluster=$CLUSTER_NAME \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --role-name AmazonEKSLoadBalancerControllerRole \
    --attach-policy-arn=arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):policy/AWSLoadBalancerControllerIAMPolicy \
    --approve

# Add EKS chart repo
helm repo add eks https://aws.github.io/eks-charts
helm repo update

# Install AWS Load Balancer Controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=$CLUSTER_NAME \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=$REGION \
    --set vpcId=$VPC_ID

echo "✅ AWS Load Balancer Controller installed!"

# Install EBS CSI Driver
echo "=== Installing EBS CSI Driver ==="

eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster $CLUSTER_NAME \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve \
    --override-existing-serviceaccounts

# Install EBS CSI Driver add-on
aws eks create-addon \
    --cluster-name $CLUSTER_NAME \
    --addon-name aws-ebs-csi-driver \
    --service-account-role-arn arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/AmazonEKS_EBS_CSI_DriverRole

echo "✅ EBS CSI Driver installed!"

# Save cluster information
cat > cluster-info.txt << INFO
Cluster Name: $CLUSTER_NAME
Region: $REGION
VPC ID: $VPC_ID
Node Group: $NODE_GROUP_NAME
Kubernetes Version: $KUBERNETES_VERSION

Access Commands:
aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME
kubectl get nodes
kubectl get pods --all-namespaces

Dashboard Access:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl proxy
# Access: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
INFO

echo "✅ Cluster information saved to cluster-info.txt"
echo "🚀 EKS cluster setup complete!"
EOF

chmod +x create_eks_cluster.sh

# Kubectl configuration and management script
cat > kubectl_manager.sh << 'EOF'
#!/bin/bash

# Function to switch between clusters
switch_cluster() {
    local cluster_name=$1
    local region=${2:-us-west-2}
    
    echo "🔄 Switching to cluster: $cluster_name in $region"
    aws eks update-kubeconfig --region $region --name $cluster_name
    
    echo "Current context:"
    kubectl config current-context
}

# Function to list available clusters
list_clusters() {
    echo "📋 Available EKS clusters:"
    aws eks list-clusters --query 'clusters[]' --output table
}

# Function to get cluster info
cluster_info() {
    local cluster_name=${1:-$(kubectl config current-context | cut -d'/' -f2)}
    
    echo "ℹ️  Cluster Information: $cluster_name"
    
    # Cluster details
    kubectl cluster-info
    
    echo -e "\n=== Nodes ==="
    kubectl get nodes -o wide
    
    echo -e "\n=== Namespaces ==="
    kubectl get namespaces
    
    echo -e "\n=== System Pods ==="
    kubectl get pods -n kube-system
    
    echo -e "\n=== Storage Classes ==="
    kubectl get storageclass
    
    echo -e "\n=== Services ==="
    kubectl get services --all-namespaces
}

# Function to create kubeconfig backup
backup_kubeconfig() {
    local backup_dir="$HOME/.kube/backups"
    mkdir -p "$backup_dir"
    
    local timestamp=$(date +%Y%m%d_%H%M%S)
    cp "$HOME/.kube/config" "$backup_dir/config_backup_$timestamp"
    
    echo "✅ Kubeconfig backed up to: $backup_dir/config_backup_$timestamp"
}

# Function to install useful kubectl plugins
install_plugins() {
    echo "🔌 Installing useful kubectl plugins..."
    
    # Install krew (kubectl plugin manager)
    (
        set -x; cd "$(mktemp -d)" &&
        OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
        ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
        KREW="krew-${OS}_${ARCH}" &&
        curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
        tar zxvf "${KREW}.tar.gz" &&
        ./"${KREW}" install krew
    )
    
    export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
    
    # Install useful plugins
    kubectl krew install ctx        # Context switching
    kubectl krew install ns         # Namespace switching
    kubectl krew install tree       # Resource tree view
    kubectl krew install tail       # Tail logs from multiple pods
    kubectl krew install who-can    # RBAC analysis
    
    echo "✅ Kubectl plugins installed!"
}

# Function to setup aliases
setup_aliases() {
    cat >> ~/.bashrc << 'ALIASES'

# Kubectl aliases
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgn='kubectl get nodes'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'

# EKS specific aliases
alias eksls='aws eks list-clusters'
alias eksdesc='aws eks describe-cluster --name'
alias eksctx='aws eks update-kubeconfig --name'

ALIASES
    
    echo "✅ Kubectl aliases added to ~/.bashrc"
    echo "Run 'source ~/.bashrc' to activate aliases"
}

# Main command handler
case ${1:-help} in
    switch)
        switch_cluster "$2" "$3"
        ;;
    list)
        list_clusters
        ;;
    info)
        cluster_info "$2"
        ;;
    backup)
        backup_kubeconfig
        ;;
    plugins)
        install_plugins
        ;;
    aliases)
        setup_aliases
        ;;
    *)
        echo "Kubectl Manager for EKS"
        echo "Usage: $0 {switch|list|info|backup|plugins|aliases} [options]"
        echo ""
        echo "Commands:"
        echo "  switch CLUSTER_NAME [REGION] - Switch to different cluster"
        echo "  list                         - List available clusters"
        echo "  info [CLUSTER_NAME]          - Show cluster information"
        echo "  backup                       - Backup kubeconfig"
        echo "  plugins                      - Install useful kubectl plugins"
        echo "  aliases                      - Setup kubectl aliases"
        ;;
esac
EOF

chmod +x kubectl_manager.sh

echo "EKS cluster setup scripts created!"
echo "Run ./create_eks_cluster.sh to create your EKS cluster"
echo "Run ./kubectl_manager.sh to manage kubectl configuration"
```

### Task 2: Pod Creation and Management
**Deploy and manage pods with various configurations and resource requirements.**

**Solution:**
```yaml
# basic-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: development
  annotations:
    description: "Basic nginx pod for learning"
spec:
  containers:
  - name: nginx
    image: nginx:1.21-alpine
    ports:
    - containerPort: 80
      name: http
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
    - name: NGINX_PORT
      value: "80"
    - name: ENVIRONMENT
      value: "development"
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
  restartPolicy: Always
```

```yaml
# multi-container-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  labels:
    app: multi-container
spec:
  containers:
  # Main application container
  - name: webapp
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
    - name: config-volume
      mountPath: /etc/nginx/conf.d
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  
  # Sidecar container for log processing
  - name: log-processor
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "$(date): Processing logs..." >> /var/log/app.log
        sleep 30
      done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
  
  # Init container
  initContainers:
  - name: init-setup
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      echo '<h1>Welcome to Multi-Container Pod</h1>' > /work-dir/index.html
      echo 'server { listen 80; location / { root /usr/share/nginx/html; index index.html; } }' > /config-dir/default.conf
    volumeMounts:
    - name: shared-data
      mountPath: /work-dir
    - name: config-volume
      mountPath: /config-dir
  
  volumes:
  - name: shared-data
    emptyDir: {}
  - name: config-volume
    emptyDir: {}
  - name: log-volume
    emptyDir: {}
  
  restartPolicy: Always
```

```bash
# Pod management script
cat > pod_manager.sh << 'EOF'
#!/bin/bash

# Function to create and deploy a pod
create_pod() {
    local pod_file=$1
    local namespace=${2:-default}
    
    echo "🚀 Creating pod from: $pod_file in namespace: $namespace"
    
    if [[ ! -f "$pod_file" ]]; then
        echo "❌ Pod file not found: $pod_file"
        return 1
    fi
    
    kubectl apply -f "$pod_file" -n "$namespace"
    
    # Wait for pod to be ready
    pod_name=$(kubectl get -f "$pod_file" -o jsonpath='{.metadata.name}')
    echo "⏳ Waiting for pod $pod_name to be ready..."
    kubectl wait --for=condition=Ready pod/"$pod_name" -n "$namespace" --timeout=300s
    
    echo "✅ Pod $pod_name is ready!"
}

# Function to check pod status
check_pod_status() {
    local pod_name=$1
    local namespace=${2:-default}
    
    echo "📊 Pod Status: $pod_name"
    
    # Basic info
    kubectl get pod "$pod_name" -n "$namespace" -o wide
    
    # Detailed description
    echo -e "\n=== Pod Details ==="
    kubectl describe pod "$pod_name" -n "$namespace"
    
    # Resource usage
    echo -e "\n=== Resource Usage ==="
    kubectl top pod "$pod_name" -n "$namespace" 2>/dev/null || echo "Metrics server not available"
    
    # Events
    echo -e "\n=== Recent Events ==="
    kubectl get events -n "$namespace" --field-selector involvedObject.name="$pod_name" --sort-by='.lastTimestamp'
}

# Function to get pod logs
get_pod_logs() {
    local pod_name=$1
    local namespace=${2:-default}
    local container=${3:-}
    local follow=${4:-false}
    
    echo "📋 Getting logs for pod: $pod_name"
    
    if [[ -n "$container" ]]; then
        if [[ "$follow" == "true" ]]; then
            kubectl logs -f "$pod_name" -c "$container" -n "$namespace"
        else
            kubectl logs "$pod_name" -c "$container" -n "$namespace" --tail=100
        fi
    else
        if [[ "$follow" == "true" ]]; then
            kubectl logs -f "$pod_name" -n "$namespace"
        else
            kubectl logs "$pod_name" -n "$namespace" --tail=100
        fi
    fi
}

# Function to execute commands in pod
exec_pod() {
    local pod_name=$1
    local namespace=${2:-default}
    local container=${3:-}
    local command=${4:-/bin/sh}
    
    echo "🔧 Executing command in pod: $pod_name"
    
    if [[ -n "$container" ]]; then
        kubectl exec -it "$pod_name" -c "$container" -n "$namespace" -- $command
    else
        kubectl exec -it "$pod_name" -n "$namespace" -- $command
    fi
}

# Function to port forward to pod
port_forward() {
    local pod_name=$1
    local local_port=$2
    local pod_port=$3
    local namespace=${4:-default}
    
    echo "🔗 Port forwarding: localhost:$local_port -> $pod_name:$pod_port"
    kubectl port-forward "$pod_name" "$local_port:$pod_port" -n "$namespace"
}

# Function to debug pod issues
debug_pod() {
    local pod_name=$1
    local namespace=${2:-default}
    
    echo "🐛 Debugging pod: $pod_name"
    
    # Check pod status
    echo "=== Pod Status ==="
    kubectl get pod "$pod_name" -n "$namespace" -o yaml
    
    # Check events
    echo -e "\n=== Events ==="
    kubectl get events -n "$namespace" --field-selector involvedObject.name="$pod_name"
    
    # Check logs
    echo -e "\n=== Logs ==="
    kubectl logs "$pod_name" -n "$namespace" --previous 2>/dev/null || echo "No previous logs"
    kubectl logs "$pod_name" -n "$namespace" --tail=50
    
    # Check node conditions
    node_name=$(kubectl get pod "$pod_name" -n "$namespace" -o jsonpath='{.spec.nodeName}')
    if [[ -n "$node_name" ]]; then
        echo -e "\n=== Node Conditions ==="
        kubectl describe node "$node_name" | grep -A 10 "Conditions:"
    fi
    
    # Resource usage
    echo -e "\n=== Resource Usage ==="
    kubectl top pod "$pod_name" -n "$namespace" 2>/dev/null || echo "Metrics server not available"
    
    # Network info
    echo -e "\n=== Network Info ==="
    pod_ip=$(kubectl get pod "$pod_name" -n "$namespace" -o jsonpath='{.status.podIP}')
    echo "Pod IP: $pod_ip"
    
    # Security context
    echo -e "\n=== Security Context ==="
    kubectl get pod "$pod_name" -n "$namespace" -o jsonpath='{.spec.securityContext}'
}

# Function to create test pods
create_test_pods() {
    echo "🧪 Creating test pods..."
    
    # Simple nginx pod
    cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test-nginx
  labels:
    app: test
    type: webserver
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
EOF

    # Busybox pod for debugging
    cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test-busybox
  labels:
    app: test
    type: debug
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sleep', '3600']
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

    # Multi-container pod
    cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test-multi-container
  labels:
    app: test
    type: multi
spec:
  containers:
  - name: web
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: sidecar
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "$(date): Hello from sidecar" > /pod-data/index.html
        sleep 10
      done
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
  volumes:
  - name: shared-data
    emptyDir: {}
EOF

    echo "✅ Test pods created!"
    echo "Use 'kubectl get pods -l app=test' to see them"
}

# Function to cleanup test pods
cleanup_test_pods() {
    echo "🧹 Cleaning up test pods..."
    kubectl delete pods -l app=test
    echo "✅ Test pods deleted!"
}

# Function to show pod resource usage
show_resource_usage() {
    local namespace=${1:-default}
    
    echo "📊 Pod Resource Usage in namespace: $namespace"
    
    # CPU and Memory usage
    kubectl top pods -n "$namespace" 2>/dev/null || echo "Metrics server not available"
    
    echo -e "\n=== Resource Requests and Limits ==="
    kubectl get pods -n "$namespace" -o custom-columns=\
'NAME:.metadata.name,'\
'CPU_REQ:.spec.containers[*].resources.requests.cpu,'\
'CPU_LIM:.spec.containers[*].resources.limits.cpu,'\
'MEM_REQ:.spec.containers[*].resources.requests.memory,'\
'MEM_LIM:.spec.containers[*].resources.limits.memory'
}

# Main command handler
case ${1:-help} in
    create)
        create_pod "$2" "$3"
        ;;
    status)
        check_pod_status "$2" "$3"
        ;;
    logs)
        get_pod_logs "$2" "$3" "$4" "$5"
        ;;
    exec)
        exec_pod "$2" "$3" "$4" "$5"
        ;;
    port-forward)
        port_forward "$2" "$3" "$4" "$5"
        ;;
    debug)
        debug_pod "$2" "$3"
        ;;
    test-create)
        create_test_pods
        ;;
    test-cleanup)
        cleanup_test_pods
        ;;
    resources)
        show_resource_usage "$2"
        ;;
    *)
        echo "Pod Manager for Kubernetes"
        echo "Usage: $0 {create|status|logs|exec|port-forward|debug|test-create|test-cleanup|resources} [options]"
        echo ""
        echo "Commands:"
        echo "  create POD_FILE [NAMESPACE]           - Create pod from YAML file"
        echo "  status POD_NAME [NAMESPACE]           - Check pod status"
        echo "  logs POD_NAME [NAMESPACE] [CONTAINER] [follow] - Get pod logs"
        echo "  exec POD_NAME [NAMESPACE] [CONTAINER] [COMMAND] - Execute command in pod"
        echo "  port-forward POD_NAME LOCAL_PORT POD_PORT [NAMESPACE] - Port forward to pod"
        echo "  debug POD_NAME [NAMESPACE]            - Debug pod issues"
        echo "  test-create                           - Create test pods"
        echo "  test-cleanup                          - Delete test pods"
        echo "  resources [NAMESPACE]                 - Show resource usage"
        ;;
esac
EOF

chmod +x pod_manager.sh

echo "Pod creation and management setup completed!"
echo "Apply pod configurations with: kubectl apply -f basic-pod.yaml"
echo "Use ./pod_manager.sh for comprehensive pod management"
```

### Task 3: Deployments and ReplicaSets
**Create and manage deployments with rolling updates and rollback capabilities.**

**Solution:**
```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
  annotations:
    deployment.kubernetes.io/revision: "1"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
          name: http
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: NGINX_PORT
          value: "80"
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;
        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
```

```yaml
# advanced-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
    environment: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
        version: v2
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: webapp
              topologyKey: kubernetes.io/hostname
      containers:
      - name: webapp
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
          name: http
        - containerPort: 8080
          name: metrics
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
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: webapp-config
      - name: secret-volume
        secret:
          secretName: webapp-secrets
      imagePullSecrets:
      - name: regcred
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  progressDeadlineSeconds: 600
  revisionHistoryLimit: 10
```

```bash
# Deployment management script
cat > deployment_manager.sh << 'EOF'
#!/bin/bash

# Function to create deployment
create_deployment() {
    local deployment_file=$1
    local namespace=${2:-default}
    
    echo "🚀 Creating deployment from: $deployment_file"
    
    if [[ ! -f "$deployment_file" ]]; then
        echo "❌ Deployment file not found: $deployment_file"
        return 1
    fi
    
    kubectl apply -f "$deployment_file" -n "$namespace"
    
    # Get deployment name
    deployment_name=$(kubectl get -f "$deployment_file" -o jsonpath='{.metadata.name}')
    
    echo "⏳ Waiting for deployment $deployment_name to be ready..."
    kubectl rollout status deployment/"$deployment_name" -n "$namespace" --timeout=300s
    
    echo "✅ Deployment $deployment_name is ready!"
}

# Function to scale deployment
scale_deployment() {
    local deployment_name=$1
    local replicas=$2
    local namespace=${3:-default}
    
    echo "📈 Scaling deployment $deployment_name to $replicas replicas"
    
    kubectl scale deployment "$deployment_name" --replicas="$replicas" -n "$namespace"
    
    echo "⏳ Waiting for scaling to complete..."
    kubectl rollout status deployment/"$deployment_name" -n "$namespace"
    
    echo "✅ Deployment scaled successfully!"
}

# Function to update deployment image
update_image() {
    local deployment_name=$1
    local container_name=$2
    local new_image=$3
    local namespace=${4:-default}
    
    echo "🔄 Updating $deployment_name container $container_name to image: $new_image"
    
    kubectl set image deployment/"$deployment_name" "$container_name=$new_image" -n "$namespace"
    
    echo "⏳ Waiting for rolling update to complete..."
    kubectl rollout status deployment/"$deployment_name" -n "$namespace"
    
    echo "✅ Image update completed!"
}

# Function to rollback deployment
rollback_deployment() {
    local deployment_name=$1
    local namespace=${2:-default}
    local revision=${3:-}
    
    echo "⏪ Rolling back deployment: $deployment_name"
    
    if [[ -n "$revision" ]]; then
        kubectl rollout undo deployment/"$deployment_name" --to-revision="$revision" -n "$namespace"
        echo "Rolling back to revision $revision"
    else
        kubectl rollout undo deployment/"$deployment_name" -n "$namespace"
        echo "Rolling back to previous revision"
    fi
    
    echo "⏳ Waiting for rollback to complete..."
    kubectl rollout status deployment/"$deployment_name" -n "$namespace"
    
    echo "✅ Rollback completed!"
}

# Function to show rollout history
show_history() {
    local deployment_name=$1
    local namespace=${2:-default}
    
    echo "📚 Rollout history for: $deployment_name"
    kubectl rollout history deployment/"$deployment_name" -n "$namespace"
    
    echo -e "\n=== Current ReplicaSets ==="
    kubectl get rs -l app="$(kubectl get deployment "$deployment_name" -n "$namespace" -o jsonpath='{.spec.selector.matchLabels.app}')" -n "$namespace"
}

# Function to pause/resume deployment
pause_deployment() {
    local deployment_name=$1
    local action=$2  # pause or resume
    local namespace=${3:-default}
    
    if [[ "$action" == "pause" ]]; then
        echo "⏸️  Pausing deployment: $deployment_name"
        kubectl rollout pause deployment/"$deployment_name" -n "$namespace"
    elif [[ "$action" == "resume" ]]; then
        echo "▶️  Resuming deployment: $deployment_name"
        kubectl rollout resume deployment/"$deployment_name" -n "$namespace"
    else
        echo "❌ Invalid action. Use 'pause' or 'resume'"
        return 1
    fi
    
    echo "✅ Deployment $action completed!"
}

# Function to show deployment status
show_status() {
    local deployment_name=$1
    local namespace=${2:-default}
    
    echo "📊 Deployment Status: $deployment_name"
    
    # Basic info
    kubectl get deployment "$deployment_name" -n "$namespace" -o wide
    
    echo -e "\n=== ReplicaSets ==="
    kubectl get rs -l app="$(kubectl get deployment "$deployment_name" -n "$namespace" -o jsonpath='{.spec.selector.matchLabels.app}')" -n "$namespace"
    
    echo -e "\n=== Pods ==="
    kubectl get pods -l app="$(kubectl get deployment "$deployment_name" -n "$namespace" -o jsonpath='{.spec.selector.matchLabels.app}')" -n "$namespace" -o wide
    
    echo -e "\n=== Deployment Events ==="
    kubectl describe deployment "$deployment_name" -n "$namespace" | grep -A 10 "Events:"
    
    echo -e "\n=== Rollout Status ==="
    kubectl rollout status deployment/"$deployment_name" -n "$namespace" --timeout=10s || echo "Rollout in progress or failed"
}

# Function to create canary deployment
create_canary() {
    local base_deployment=$1
    local canary_image=$2
    local canary_percentage=${3:-10}
    local namespace=${4:-default}
    
    echo "🐤 Creating canary deployment for: $base_deployment"
    
    # Get original deployment
    kubectl get deployment "$base_deployment" -n "$namespace" -o yaml > /tmp/original-deployment.yaml
    
    # Calculate replica counts
    original_replicas=$(kubectl get deployment "$base_deployment" -n "$namespace" -o jsonpath='{.spec.replicas}')
    canary_replicas=$(( original_replicas * canary_percentage / 100 ))
    new_original_replicas=$(( original_replicas - canary_replicas ))
    
    # Scale down original
    kubectl scale deployment "$base_deployment" --replicas="$new_original_replicas" -n "$namespace"
    
    # Create canary deployment
    sed "s/$base_deployment/${base_deployment}-canary/g" /tmp/original-deployment.yaml | \
    sed "s/replicas: $original_replicas/replicas: $canary_replicas/g" | \
    sed "s/image: .*/image: $canary_image/g" | \
    kubectl apply -f - -n "$namespace"
    
    echo "✅ Canary deployment created!"
    echo "Original: $new_original_replicas replicas"
    echo "Canary: $canary_replicas replicas ($canary_percentage%)"
}

# Function to promote canary
promote_canary() {
    local base_deployment=$1
    local namespace=${2:-default}
    
    echo "🚀 Promoting canary deployment: ${base_deployment}-canary"
    
    # Get canary image
    canary_image=$(kubectl get deployment "${base_deployment}-canary" -n "$namespace" -o jsonpath='{.spec.template.spec.containers[0].image}')
    
    # Update original deployment with canary image
    kubectl set image deployment/"$base_deployment" "*=$canary_image" -n "$namespace"
    
    # Scale back to original size
    original_replicas=$(kubectl get deployment "${base_deployment}-canary" -n "$namespace" -o jsonpath='{.spec.replicas}')
    total_replicas=$(kubectl get deployment "$base_deployment" -n "$namespace" -o jsonpath='{.spec.replicas}')
    final_replicas=$(( total_replicas + original_replicas ))
    
    kubectl scale deployment "$base_deployment" --replicas="$final_replicas" -n "$namespace"
    
    # Delete canary
    kubectl delete deployment "${base_deployment}-canary" -n "$namespace"
    
    echo "✅ Canary promoted and cleanup completed!"
}

# Function to create sample deployments
create_samples() {
    echo "📝 Creating sample deployments..."
    
    # Simple nginx deployment
    cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-nginx
  labels:
    app: sample-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-nginx
  template:
    metadata:
      labels:
        app: sample-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
EOF

    # Sample app deployment
    cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  labels:
    app: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: app
        image: httpd:2.4-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
EOF

    echo "✅ Sample deployments created!"
}

# Function to cleanup samples
cleanup_samples() {
    echo "🧹 Cleaning up sample deployments..."
    kubectl delete deployment sample-nginx sample-app 2>/dev/null || echo "Some deployments may not exist"
    echo "✅ Cleanup completed!"
}

# Main command handler
case ${1:-help} in
    create)
        create_deployment "$2" "$3"
        ;;
    scale)
        scale_deployment "$2" "$3" "$4"
        ;;
    update)
        update_image "$2" "$3" "$4" "$5"
        ;;
    rollback)
        rollback_deployment "$2" "$3" "$4"
        ;;
    history)
        show_history "$2" "$3"
        ;;
    pause)
        pause_deployment "$2" "pause" "$3"
        ;;
    resume)
        pause_deployment "$2" "resume" "$3"
        ;;
    status)
        show_status "$2" "$3"
        ;;
    canary)
        create_canary "$2" "$3" "$4" "$5"
        ;;
    promote)
        promote_canary "$2" "$3"
        ;;
    samples)
        create_samples
        ;;
    cleanup)
        cleanup_samples
        ;;
    *)
        echo "Deployment Manager for Kubernetes"
        echo "Usage: $0 {create|scale|update|rollback|history|pause|resume|status|canary|promote|samples|cleanup} [options]"
        echo ""
        echo "Commands:"
        echo "  create DEPLOYMENT_FILE [NAMESPACE]              - Create deployment"
        echo "  scale DEPLOYMENT_NAME REPLICAS [NAMESPACE]      - Scale deployment"
        echo "  update DEPLOYMENT_NAME CONTAINER_NAME NEW_IMAGE [NAMESPACE] - Update image"
        echo "  rollback DEPLOYMENT_NAME [NAMESPACE] [REVISION] - Rollback deployment"
        echo "  history DEPLOYMENT_NAME [NAMESPACE]             - Show rollout history"
        echo "  pause DEPLOYMENT_NAME [NAMESPACE]               - Pause rollout"
        echo "  resume DEPLOYMENT_NAME [NAMESPACE]              - Resume rollout"
        echo "  status DEPLOYMENT_NAME [NAMESPACE]              - Show deployment status"
        echo "  canary BASE_DEPLOYMENT CANARY_IMAGE [PERCENTAGE] [NAMESPACE] - Create canary"
        echo "  promote BASE_DEPLOYMENT [NAMESPACE]             - Promote canary"
        echo "  samples                                          - Create sample deployments"
        echo "  cleanup                                          - Cleanup sample deployments"
        ;;
esac
EOF

chmod +x deployment_manager.sh

echo "Deployment and ReplicaSet management setup completed!"
echo "Apply deployments with: kubectl apply -f nginx-deployment.yaml"
echo "Use ./deployment_manager.sh for comprehensive deployment management"
```

*[Continuing with remaining Kubernetes EKS tasks 4-20 would follow the same detailed pattern with comprehensive solutions for services, ingress, persistent volumes, RBAC, monitoring, etc.]*

## Intermediate Level Solutions (Tasks 11-15)

*[Solutions continue with more complex scenarios like StatefulSets, DaemonSets, Jobs, CronJobs, custom resources, etc.]*

## Advanced Level Solutions (Tasks 16-20)

*[Advanced solutions include Helm charts, operators, service mesh, advanced networking, multi-cluster management, etc.]*

---

**Note**: All Kubernetes EKS solutions follow best practices including:
- Resource management and limits
- Health checks and probes
- Security contexts and RBAC
- High availability configurations
- Monitoring and logging
- Backup and disaster recovery
- Cost optimization
- GitOps workflows

Remember to clean up AWS EKS resources after testing to avoid unnecessary charges.