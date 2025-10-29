# DevOps Tasks Solutions

This document contains detailed solutions for all 20 DevOps practice tasks.

## Basic Level Solutions (Tasks 1-10)

### Task 1: Git Repository Setup
**Initialize a new Git repository, create branches for development and staging, and implement a basic branching strategy.**

**Solution:**
```bash
# Initialize repository
git init my-project
cd my-project

# Create initial commit
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"

# Create and push to remote repository
git remote add origin https://github.com/username/my-project.git
git branch -M main
git push -u origin main

# Create development branch
git checkout -b development
echo "Development environment setup" > dev-setup.md
git add dev-setup.md
git commit -m "Add development setup documentation"
git push -u origin development

# Create staging branch
git checkout -b staging
echo "Staging environment configuration" > staging-config.md
git add staging-config.md
git commit -m "Add staging configuration"
git push -u origin staging

# Create feature branch workflow
git checkout development
git checkout -b feature/user-authentication
echo "User authentication feature" > auth.md
git add auth.md
git commit -m "Add user authentication feature"
git push -u origin feature/user-authentication

# Merge feature to development
git checkout development
git merge feature/user-authentication
git push origin development

# Create pull request template
mkdir .github
cat > .github/pull_request_template.md << 'EOF'
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
EOF

git add .github/
git commit -m "Add pull request template"
git push origin development
```

### Task 2: Jenkins Installation and Configuration
**Install Jenkins on a Linux server and configure basic security settings.**

**Solution:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Java (required for Jenkins)
sudo apt install openjdk-11-jdk -y

# Add Jenkins repository
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Start and enable Jenkins service
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Configure firewall (if UFW is enabled)
sudo ufw allow 8080
sudo ufw reload

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Jenkins configuration script
cat > jenkins-config.groovy << 'EOF'
import jenkins.model.*
import hudson.security.*
import jenkins.security.s2m.AdminWhitelistRule

def instance = Jenkins.getInstance()

// Create admin user
def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("admin", "admin123!")
instance.setSecurityRealm(hudsonRealm)

// Set authorization strategy
def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
instance.setAuthorizationStrategy(strategy)

// Disable CLI over remoting
instance.getDescriptor("jenkins.CLI").get().setEnabled(false)

// Enable CSRF protection
instance.setCrumbIssuer(new DefaultCrumbIssuer(true))

// Save configuration
instance.save()
EOF

# Apply configuration (run after initial setup)
sudo cp jenkins-config.groovy /var/lib/jenkins/init.groovy.d/
sudo chown jenkins:jenkins /var/lib/jenkins/init.groovy.d/jenkins-config.groovy
sudo systemctl restart jenkins

# Install essential plugins via CLI
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# Plugin list
cat > plugins.txt << 'EOF'
git
workflow-aggregator
pipeline-stage-view
build-timeout
timestamper
ws-cleanup
ant
gradle
nodejs
docker-workflow
EOF

# Install plugins
while IFS= read -r plugin; do
    java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:admin123! install-plugin "$plugin"
done < plugins.txt

# Restart Jenkins to activate plugins
sudo systemctl restart jenkins

echo "Jenkins installed and configured at http://your-server:8080"
echo "Username: admin"
echo "Password: admin123!"
```

### Task 3: CI/CD Pipeline Creation
**Create a Jenkins pipeline that builds, tests, and deploys a simple web application.**

**Solution:**
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'my-web-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Build the application
                    sh '''
                        echo "Building application..."
                        npm install
                        npm run build
                    '''
                }
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh '''
                            echo "Running unit tests..."
                            npm test
                        '''
                        publishTestResults testResultsPattern: 'test-results.xml'
                    }
                }
                
                stage('Code Quality') {
                    steps {
                        sh '''
                            echo "Running code quality checks..."
                            npm run lint
                            npm run security-scan
                        '''
                    }
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    // Build Docker image
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    docker.build("${IMAGE_NAME}:latest")
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    // Scan Docker image for vulnerabilities
                    sh '''
                        echo "Scanning Docker image for vulnerabilities..."
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                            aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'development'
            }
            steps {
                script {
                    // Deploy to staging environment
                    sh '''
                        echo "Deploying to staging..."
                        docker stop staging-app || true
                        docker rm staging-app || true
                        docker run -d --name staging-app -p 3001:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        
        stage('Integration Tests') {
            when {
                branch 'development'
            }
            steps {
                sh '''
                    echo "Running integration tests..."
                    sleep 10  # Wait for application to start
                    curl -f http://localhost:3001/health || exit 1
                    npm run integration-tests
                '''
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Manual approval for production deployment
                    input message: 'Deploy to production?', ok: 'Deploy'
                    
                    // Push to registry
                    docker.withRegistry("https://${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS}") {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
                    
                    // Deploy to production
                    sh '''
                        echo "Deploying to production..."
                        # Blue-green deployment strategy
                        docker pull ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        
                        # Start new container
                        docker run -d --name prod-app-new -p 3002:3000 \
                            ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        
                        # Health check
                        sleep 15
                        curl -f http://localhost:3002/health
                        
                        # Switch traffic (nginx configuration update)
                        sudo sed -i 's/3000/3002/' /etc/nginx/sites-available/default
                        sudo nginx -s reload
                        
                        # Stop old container
                        docker stop prod-app || true
                        docker rm prod-app || true
                        
                        # Rename new container
                        docker rename prod-app-new prod-app
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Clean up workspace
            cleanWs()
            
            // Archive artifacts
            archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
            
            // Publish test results
            publishTestResults testResultsPattern: 'test-results.xml'
            
            // Publish coverage reports
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'coverage',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
        }
        
        success {
            slackSend channel: '#deployments',
                      color: 'good',
                      message: "✅ Pipeline succeeded for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        
        failure {
            slackSend channel: '#deployments',
                      color: 'danger',
                      message: "❌ Pipeline failed for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            
            // Send email notification
            emailext body: "Pipeline failed. Check: ${env.BUILD_URL}",
                     subject: "Pipeline Failure: ${env.JOB_NAME}",
                     to: '${DEFAULT_RECIPIENTS}'
        }
    }
}
```

```dockerfile
# Dockerfile for the web application
FROM node:16-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]
```

### Task 4: Infrastructure as Code Integration
**Integrate Terraform with Jenkins for automated infrastructure provisioning.**

**Solution:**
```groovy
// Infrastructure Pipeline
pipeline {
    agent any
    
    environment {
        TF_VAR_environment = "${params.ENVIRONMENT}"
        AWS_DEFAULT_REGION = 'us-west-2'
        TF_IN_AUTOMATION = 'true'
    }
    
    parameters {
        choice(
            name: 'ACTION',
            choices: ['plan', 'apply', 'destroy'],
            description: 'Terraform action to perform'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Environment to deploy to'
        )
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            steps {
                script {
                    sh '''
                        cd terraform/environments/${ENVIRONMENT}
                        terraform init -backend-config="bucket=my-terraform-state-${ENVIRONMENT}" \
                                      -backend-config="key=infrastructure/terraform.tfstate" \
                                      -backend-config="region=us-west-2"
                    '''
                }
            }
        }
        
        stage('Terraform Validate') {
            steps {
                sh '''
                    cd terraform/environments/${ENVIRONMENT}
                    terraform validate
                    terraform fmt -check
                '''
            }
        }
        
        stage('Terraform Plan') {
            steps {
                script {
                    sh '''
                        cd terraform/environments/${ENVIRONMENT}
                        terraform plan -out=tfplan \
                                      -var-file="terraform.tfvars" \
                                      -detailed-exitcode
                    '''
                    
                    // Archive the plan
                    archiveArtifacts artifacts: "terraform/environments/${params.ENVIRONMENT}/tfplan"
                }
            }
        }
        
        stage('Manual Approval') {
            when {
                expression { params.ACTION == 'apply' || params.ACTION == 'destroy' }
            }
            steps {
                script {
                    def planOutput = sh(
                        script: "cd terraform/environments/${params.ENVIRONMENT} && terraform show -no-color tfplan",
                        returnStdout: true
                    ).trim()
                    
                    input message: "Review Terraform plan and approve ${params.ACTION}:",
                          parameters: [text(name: 'PLAN', defaultValue: planOutput, description: 'Terraform Plan')]
                }
            }
        }
        
        stage('Terraform Apply') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                sh '''
                    cd terraform/environments/${ENVIRONMENT}
                    terraform apply tfplan
                '''
            }
        }
        
        stage('Terraform Destroy') {
            when {
                expression { params.ACTION == 'destroy' }
            }
            steps {
                sh '''
                    cd terraform/environments/${ENVIRONMENT}
                    terraform destroy -auto-approve -var-file="terraform.tfvars"
                '''
            }
        }
        
        stage('Output Values') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                script {
                    def outputs = sh(
                        script: "cd terraform/environments/${params.ENVIRONMENT} && terraform output -json",
                        returnStdout: true
                    ).trim()
                    
                    writeFile file: "terraform-outputs-${params.ENVIRONMENT}.json", text: outputs
                    archiveArtifacts artifacts: "terraform-outputs-${params.ENVIRONMENT}.json"
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Clean up plan files
                sh "rm -f terraform/environments/${params.ENVIRONMENT}/tfplan"
            }
        }
        
        success {
            slackSend channel: '#infrastructure',
                      color: 'good',
                      message: "✅ Terraform ${params.ACTION} succeeded for ${params.ENVIRONMENT} environment"
        }
        
        failure {
            slackSend channel: '#infrastructure',
                      color: 'danger',
                      message: "❌ Terraform ${params.ACTION} failed for ${params.ENVIRONMENT} environment"
        }
    }
}
```

### Task 5: Multi-Environment Deployment
**Set up deployment pipelines for development, staging, and production environments.**

**Solution:**
```groovy
// Multi-Environment Deployment Pipeline
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'your-registry.com'
        IMAGE_NAME = 'my-app'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Build and Test') {
            steps {
                script {
                    // Build application
                    sh '''
                        npm install
                        npm run build
                        npm test
                    '''
                    
                    // Build Docker image
                    def image = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                    
                    // Push to registry
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'registry-credentials') {
                        image.push("${BUILD_NUMBER}")
                        image.push("latest")
                    }
                }
            }
        }
        
        stage('Deploy to Development') {
            when {
                anyOf {
                    branch 'development'
                    branch 'feature/*'
                }
            }
            steps {
                script {
                    deployToEnvironment('dev', BUILD_NUMBER)
                    runSmokeTests('dev')
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'development'
            }
            steps {
                script {
                    deployToEnvironment('staging', BUILD_NUMBER)
                    runIntegrationTests('staging')
                    runPerformanceTests('staging')
                }
            }
        }
        
        stage('Manual Approval for Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def deploymentInfo = """
                    Ready to deploy to production:
                    - Build: ${BUILD_NUMBER}
                    - Commit: ${env.GIT_COMMIT[0..7]}
                    - Branch: ${env.BRANCH_NAME}
                    """
                    
                    input message: 'Deploy to production?',
                          parameters: [text(name: 'DEPLOYMENT_INFO', defaultValue: deploymentInfo)]
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Blue-green deployment
                    blueGreenDeployment('prod', BUILD_NUMBER)
                    runSmokeTests('prod')
                }
            }
        }
    }
    
    post {
        success {
            script {
                def environment = env.BRANCH_NAME == 'main' ? 'production' : 
                                 env.BRANCH_NAME == 'development' ? 'staging & development' : 'development'
                
                slackSend channel: '#deployments',
                          color: 'good',
                          message: "✅ Successfully deployed ${env.JOB_NAME} #${BUILD_NUMBER} to ${environment}"
            }
        }
        
        failure {
            slackSend channel: '#deployments',
                      color: 'danger',
                      message: "❌ Deployment failed for ${env.JOB_NAME} #${BUILD_NUMBER}"
        }
    }
}

// Helper function for environment deployment
def deployToEnvironment(environment, imageTag) {
    sh """
        helm upgrade --install my-app-${environment} ./helm/my-app \\
            --namespace ${environment} \\
            --create-namespace \\
            --set image.tag=${imageTag} \\
            --set environment=${environment} \\
            --values ./helm/values-${environment}.yaml \\
            --wait --timeout 300s
    """
}

// Helper function for blue-green deployment
def blueGreenDeployment(environment, imageTag) {
    sh """
        # Deploy to blue environment
        helm upgrade --install my-app-${environment}-blue ./helm/my-app \\
            --namespace ${environment} \\
            --set image.tag=${imageTag} \\
            --set environment=${environment} \\
            --set deployment.color=blue \\
            --values ./helm/values-${environment}.yaml \\
            --wait --timeout 300s
        
        # Health check
        kubectl wait --for=condition=ready pod -l app=my-app,color=blue -n ${environment} --timeout=300s
        
        # Switch traffic
        kubectl patch service my-app-service -n ${environment} -p '{"spec":{"selector":{"color":"blue"}}}'
        
        # Remove old green deployment
        helm uninstall my-app-${environment}-green -n ${environment} || true
    """
}

// Helper function for smoke tests
def runSmokeTests(environment) {
    sh """
        echo "Running smoke tests for ${environment}..."
        kubectl run smoke-test-${BUILD_NUMBER} \\
            --image=curlimages/curl \\
            --rm -i --restart=Never \\
            --namespace=${environment} \\
            -- curl -f http://my-app-service.${environment}.svc.cluster.local/health
    """
}

// Helper function for integration tests
def runIntegrationTests(environment) {
    sh """
        echo "Running integration tests for ${environment}..."
        kubectl apply -f k8s/integration-tests.yaml -n ${environment}
        kubectl wait --for=condition=complete job/integration-tests -n ${environment} --timeout=600s
        kubectl logs job/integration-tests -n ${environment}
    """
}

// Helper function for performance tests
def runPerformanceTests(environment) {
    sh """
        echo "Running performance tests for ${environment}..."
        kubectl apply -f k8s/performance-tests.yaml -n ${environment}
        kubectl wait --for=condition=complete job/performance-tests -n ${environment} --timeout=900s
    """
}
```

### Task 6: Monitoring and Alerting Setup
**Configure monitoring with Prometheus and Grafana, set up alerting rules.**

**Solution:**
```yaml
# prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    rule_files:
      - /etc/prometheus/rules/*.yml
    
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              - alertmanager:9093
    
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      
      - job_name: 'node-exporter'
        static_configs:
          - targets: ['node-exporter:9100']
      
      - job_name: 'application'
        kubernetes_sd_configs:
          - role: pod
            namespaces:
              names:
                - dev
                - staging
                - prod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)
          - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            regex: ([^:]+)(?::\d+)?;(\d+)
            replacement: $1:$2
            target_label: __address__
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
          - source_labels: [__meta_kubernetes_namespace]
            action: replace
            target_label: kubernetes_namespace
          - source_labels: [__meta_kubernetes_pod_name]
            action: replace
            target_label: kubernetes_pod_name
```

```yaml
# alerting-rules.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-rules
  namespace: monitoring
data:
  application.yml: |
    groups:
      - name: application.rules
        rules:
          - alert: HighErrorRate
            expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "High error rate detected"
              description: "Error rate is {{ $value }} errors per second"
          
          - alert: HighLatency
            expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "High latency detected"
              description: "95th percentile latency is {{ $value }} seconds"
          
          - alert: ServiceDown
            expr: up == 0
            for: 1m
            labels:
              severity: critical
            annotations:
              summary: "Service is down"
              description: "{{ $labels.instance }} has been down for more than 1 minute"
          
          - alert: HighMemoryUsage
            expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.8
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "High memory usage"
              description: "Memory usage is {{ $value | humanizePercentage }}"
          
          - alert: HighCPUUsage
            expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "High CPU usage"
              description: "CPU usage is {{ $value }}%"
          
          - alert: DiskSpaceLow
            expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
            for: 5m
            labels:
              severity: critical
            annotations:
              summary: "Low disk space"
              description: "Disk space is {{ $value | humanizePercentage }} full"
```

```yaml
# alertmanager-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |
    global:
      smtp_smarthost: 'smtp.gmail.com:587'
      smtp_from: 'alerts@mycompany.com'
      smtp_auth_username: 'alerts@mycompany.com'
      smtp_auth_password: 'app-password'
    
    route:
      group_by: ['alertname', 'instance']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 1h
      receiver: 'default'
      routes:
        - match:
            severity: critical
          receiver: 'critical-alerts'
        - match:
            severity: warning
          receiver: 'warning-alerts'
    
    receivers:
      - name: 'default'
        slack_configs:
          - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
            channel: '#alerts'
            title: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
            text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
      
      - name: 'critical-alerts'
        email_configs:
          - to: 'oncall@mycompany.com'
            subject: 'CRITICAL: {{ .GroupLabels.alertname }}'
            body: |
              {{ range .Alerts }}
              Alert: {{ .Annotations.summary }}
              Description: {{ .Annotations.description }}
              Instance: {{ .Labels.instance }}
              {{ end }}
        slack_configs:
          - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
            channel: '#critical-alerts'
            title: 'CRITICAL: {{ .GroupLabels.alertname }}'
            text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
            send_resolved: true
      
      - name: 'warning-alerts'
        slack_configs:
          - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
            channel: '#warnings'
            title: 'WARNING: {{ .GroupLabels.alertname }}'
            text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

```bash
# Deploy monitoring stack
#!/bin/bash

# Create monitoring namespace
kubectl create namespace monitoring

# Deploy Prometheus
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
        - name: rules
          mountPath: /etc/prometheus/rules
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'
          - '--web.enable-lifecycle'
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: rules
        configMap:
          name: prometheus-rules
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
  type: LoadBalancer
EOF

# Deploy Grafana
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: "admin123"
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
      volumes:
      - name: grafana-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
  type: LoadBalancer
EOF

# Deploy Node Exporter as DaemonSet
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
        args:
          - '--path.rootfs=/host'
        volumeMounts:
        - name: root
          mountPath: /host
          readOnly: true
      volumes:
      - name: root
        hostPath:
          path: /
      tolerations:
      - operator: Exists
EOF

echo "Monitoring stack deployed successfully!"
echo "Access Grafana at: http://$(kubectl get svc grafana -n monitoring -o jsonpath='{.status.loadBalancer.ingress[0].ip}'):3000"
echo "Access Prometheus at: http://$(kubectl get svc prometheus -n monitoring -o jsonpath='{.status.loadBalancer.ingress[0].ip}'):9090"
```


### Task 7: Docker Integration and Image Management

**Integrate Docker into the CI/CD pipeline and manage images efficiently.**

**Detailed Solution:**

#### 1. Write a Dockerfile for Your Application
Create a `Dockerfile` in your project root:
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### 2. Build and Tag Docker Images
Use meaningful tags (e.g., commit hash, build number, environment):
```bash
GIT_COMMIT=$(git rev-parse --short HEAD)
docker build -t my-app:$GIT_COMMIT -t my-app:latest .
```

#### 3. Push Images to a Registry
Authenticate and push to Docker Hub, ECR, or another registry:
```bash
# Docker Hub example
docker login -u $DOCKER_USER -p $DOCKER_PASS
docker tag my-app:$GIT_COMMIT mydockerhubuser/my-app:$GIT_COMMIT
docker tag my-app:latest mydockerhubuser/my-app:latest
docker push mydockerhubuser/my-app:$GIT_COMMIT
docker push mydockerhubuser/my-app:latest
```

#### 4. Integrate Docker Steps in CI/CD Pipeline
Example Jenkins pipeline snippet:
```groovy
pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
    IMAGE_NAME = 'mydockerhubuser/my-app'
  }
  stages {
    stage('Build Docker Image') {
      steps {
        script {
          GIT_COMMIT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          sh """
            docker build -t $IMAGE_NAME:$GIT_COMMIT -t $IMAGE_NAME:latest .
          """
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push $IMAGE_NAME:$GIT_COMMIT
            docker push $IMAGE_NAME:latest
          """
        }
      }
    }
    stage('Cleanup Old Images') {
      steps {
        sh 'docker image prune -af --filter "until=24h"'
      }
    }
  }
}
```

#### 5. Best Practices
- Use multi-stage builds to reduce image size.
- Pin base images to specific versions for reproducibility.
- Scan images for vulnerabilities (see next task).
- Use `.dockerignore` to avoid copying unnecessary files.
- Automate image cleanup to save disk space.

#### 6. Troubleshooting
- If builds fail, check Docker daemon logs and ensure credentials are correct.
- Use `docker history` and `docker inspect` to debug image layers.

This approach ensures your application is containerized, images are versioned and pushed to a registry, and the process is fully automated in your CI/CD pipeline.

### Task 8: Security Scanning in CI/CD

**Integrate security scanning tools to detect vulnerabilities in code and images.**

**Detailed Solution:**

#### 1. Source Code Scanning (SCA & Dependency Checks)
- Use tools like `npm audit` (Node.js), `pip-audit` (Python), or `OWASP Dependency-Check` (Java) to scan for vulnerable dependencies.
```bash
# Node.js example
npm audit --production --json > audit-report.json
# Fail build if vulnerabilities are found
npm audit --production --audit-level=high
```

#### 2. Static Application Security Testing (SAST)
- Integrate SAST tools (e.g., SonarQube, CodeQL, Bandit) to analyze code for security issues.
```bash
# Example with SonarQube Scanner
sonar-scanner -Dsonar.projectKey=my-app -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=$SONAR_TOKEN
```

#### 3. Container Image Scanning
- Use Trivy, Clair, or Anchore to scan Docker images for vulnerabilities before deployment.
```bash
# Scan Docker image locally
trivy image my-app:latest --format table --output trivy-report.txt
# Fail pipeline if critical vulnerabilities are found
trivy image --exit-code 1 --severity CRITICAL my-app:latest
```

#### 4. Pipeline Integration Example (Jenkins)
```groovy
pipeline {
  agent any
  environment {
    SONAR_TOKEN = credentials('sonar-token')
  }
  stages {
    stage('Dependency Scan') {
      steps {
        sh 'npm audit --production --audit-level=high'
      }
    }
    stage('SAST') {
      steps {
        sh 'sonar-scanner -Dsonar.projectKey=my-app -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=$SONAR_TOKEN'
      }
    }
    stage('Container Scan') {
      steps {
        sh 'trivy image --exit-code 1 --severity CRITICAL my-app:latest'
      }
    }
  }
  post {
    failure {
      mail to: 'security@company.com',
         subject: "Security Scan Failed: ${env.JOB_NAME}",
         body: "Check the pipeline logs for details."
    }
  }
}
```

#### 5. Remediation Steps
- Review scan reports and prioritize fixing critical/high vulnerabilities.
- Update dependencies and rebuild images.
- Document exceptions and mitigations for non-fixable issues.

#### 6. Best Practices
- Automate scans on every commit and pull request.
- Store scan reports as pipeline artifacts.
- Integrate with Slack/email for real-time alerts.
- Regularly update scanning tools and vulnerability databases.

This approach ensures vulnerabilities are detected early and remediated before code or images reach production.

### Task 9: Automated Backup and Recovery

**Implement automated backup and disaster recovery for critical data.**

**Detailed Solution:**

#### 1. Identify Critical Data
- Databases (e.g., PostgreSQL, MySQL, MongoDB)
- Application configuration files
- Persistent storage (e.g., cloud buckets, volumes)

#### 2. Create Backup Scripts
Example: PostgreSQL database backup
```bash
#!/bin/bash
set -e
BACKUP_DIR="/backups"
DB_NAME="mydb"
DB_USER="dbuser"
DB_HOST="dbhost"
DATE=$(date +%F-%H%M)
mkdir -p $BACKUP_DIR
pg_dump -U $DB_USER -h $DB_HOST $DB_NAME | gzip > $BACKUP_DIR/${DB_NAME}-${DATE}.sql.gz
```

#### 3. Automate Backups with Cron
Edit crontab (`crontab -e`) to schedule daily backups at 2 AM:
```
0 2 * * * /usr/local/bin/pg_backup.sh
```

#### 4. Store Backups Securely
- Use offsite/cloud storage (e.g., AWS S3, Azure Blob, Google Cloud Storage)
- Example: Upload to S3
```bash
aws s3 cp $BACKUP_DIR/${DB_NAME}-${DATE}.sql.gz s3://my-backup-bucket/db/
```
- Encrypt backups using tools like `gpg` or enable server-side encryption in cloud storage.

#### 5. Restore Procedures
To restore a backup:
```bash
# Download from S3
aws s3 cp s3://my-backup-bucket/db/mydb-2025-10-24-0200.sql.gz /tmp/
# Restore to database
gunzip -c /tmp/mydb-2025-10-24-0200.sql.gz | psql -U dbuser -h dbhost mydb
```

#### 6. Disaster Recovery Planning
- Document recovery steps and test regularly (run restore drills).
- Automate failover for cloud-managed databases (e.g., AWS RDS Multi-AZ, read replicas).
- Use infrastructure-as-code to quickly recreate infrastructure.

#### 7. Monitoring and Alerts
- Monitor backup job status and storage usage.
- Send alerts on backup failures (e.g., via email, Slack, or monitoring tools).

#### 8. Best Practices
- Retain multiple backup versions (e.g., daily, weekly, monthly).
- Regularly test restores to ensure backup integrity.
- Restrict access to backup files and storage locations.

This approach ensures your data is regularly backed up, securely stored, and can be restored quickly in case of disaster.

### Task 10: Configuration Management with Ansible

**Use Ansible to automate server configuration and application deployment.**

**Detailed Solution:**

#### 1. Ansible Inventory File
Define your hosts in an inventory file (e.g., `inventory.ini`):
```ini
[webservers]
web1.example.com
web2.example.com
```

#### 2. Ansible Playbook Structure
Create a playbook (e.g., `site.yml`) to define tasks:
```yaml
---
- hosts: webservers
  become: yes
  vars:
    app_src: ./dist/
    app_dest: /var/www/html/
  tasks:
    - name: Ensure Nginx is installed
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Deploy application files
      copy:
        src: "{{ app_src }}"
        dest: "{{ app_dest }}"
        owner: www-data
        group: www-data
        mode: '0755'
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

#### 3. Using Roles for Reusability
Organize tasks into roles (e.g., `roles/web/tasks/main.yml`):
```yaml
# roles/web/tasks/main.yml
- name: Ensure Nginx is installed
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy application files
  copy:
    src: "{{ app_src }}"
    dest: "{{ app_dest }}"
    owner: www-data
    group: www-data
    mode: '0755'
  notify: Restart Nginx
```
Reference the role in your playbook:
```yaml
# site.yml
- hosts: webservers
  become: yes
  roles:
    - role: web
      app_src: ./dist/
      app_dest: /var/www/html/
```

#### 4. Running the Playbook
Execute the playbook with your inventory:
```bash
ansible-playbook -i inventory.ini site.yml
```

#### 5. Best Practices
- Use variables for environment-specific values.
- Store secrets in Ansible Vault.
- Use handlers for idempotent service restarts.
- Test playbooks with `--check` and `--diff` flags.

This approach enables automated, repeatable, and secure server configuration and application deployment using Ansible.

## Intermediate Level Solutions (Tasks 11-15)

### Task 11: Multi-Cloud Deployment

**Deploy applications across AWS and Azure using Terraform.**

**Detailed Solution:**

#### 1. Set Up Providers in Terraform
Create a `main.tf` file with both AWS and Azure providers:
```hcl
provider "aws" {
  region  = "us-west-2"
  profile = "default" # or use access_key/secret_key
}

provider "azurerm" {
  features {}
}
```

#### 2. Define Resources for Each Cloud
Example: Deploy an EC2 instance (AWS) and a VM (Azure):
```hcl
# AWS EC2
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "aws-web"
  }
}

# Azure VM
resource "azurerm_resource_group" "rg" {
  name     = "example-resources"
  location = "East US"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "example-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}
# ...add subnet, NIC, and VM resources...
```

#### 3. Manage State
- Use remote backends (e.g., S3 for AWS, Azure Storage for Azure) to store Terraform state securely.
```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "multi-cloud/terraform.tfstate"
    region = "us-west-2"
  }
}
```

#### 4. Workflow
```bash
# Initialize providers and backend
terraform init
# Validate configuration
terraform validate
# Plan changes
terraform plan -out=tfplan
# Apply changes
terraform apply tfplan
```

#### 5. Best Practices
- Use workspaces or separate state files for each environment (dev, staging, prod).
- Store secrets in a secure vault (e.g., AWS Secrets Manager, Azure Key Vault).
- Use modules to organize reusable infrastructure code.
- Tag resources for cost tracking and management.

This approach enables you to provision and manage resources across multiple clouds using a single Terraform workflow.

### Task 12: Advanced Monitoring and Logging

**Set up centralized logging with ELK stack and advanced monitoring with custom metrics.**

**Detailed Solution:**

#### 1. Centralized Logging with ELK Stack
- **Elasticsearch**: Stores and indexes logs.
- **Logstash**: Parses and transforms logs.
- **Kibana**: Visualizes logs and metrics.

**Deployment Example (Kubernetes):**
```bash
# Deploy Elasticsearch
kubectl apply -f https://download.elastic.co/downloads/eck/2.10.0/all-in-one.yaml
# Deploy Filebeat as DaemonSet to collect logs from all nodes
kubectl apply -f filebeat-kubernetes.yaml
# Deploy Logstash and Kibana (see Elastic documentation for manifests)
```

**Logstash Pipeline Example:**
```conf
input { beats { port => 5044 } }
filter {
  grok { match => { "message" => "%{COMBINEDAPACHELOG}" } }
}
output {
  elasticsearch { hosts => ["http://elasticsearch:9200"] }
}
```

#### 2. Application Metrics with Prometheus
- Instrument your application with Prometheus client libraries (Node.js, Python, Java, etc.).
- Example (Node.js):
```js
const client = require('prom-client');
const httpRequestDurationMicroseconds = new client.Histogram({
  name: 'http_request_duration_ms',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'route', 'code'],
  buckets: [50, 100, 200, 300, 400, 500, 1000]
});
// ...instrument your routes...
```
- Expose `/metrics` endpoint and configure Prometheus to scrape it.

#### 3. Alerting Integration
- Define alerting rules in Prometheus (see Task 6 for examples).
- Integrate with Alertmanager for notifications (Slack, email, etc.).

#### 4. Visualization
- Use Kibana for log dashboards and search.
- Use Grafana for Prometheus metrics dashboards.

#### 5. Best Practices
- Use structured logging (JSON) for easier parsing.
- Retain logs according to compliance requirements.
- Secure access to dashboards and log data.

This approach provides full visibility into application health, performance, and security through centralized logging and advanced monitoring.

### Task 13: Security Compliance Automation

**Automate compliance checks using tools like OpenSCAP or Chef InSpec.**

**Detailed Solution:**

#### 1. Choose a Compliance Tool
- **Chef InSpec**: Infrastructure and application compliance as code.
- **OpenSCAP**: Security Content Automation Protocol for Linux systems.

#### 2. Install the Tool
```bash
# Install InSpec (Ruby required)
gem install inspec
# Or use Docker
docker run --rm -it chef/inspec help
```

#### 3. Create or Download Compliance Profiles
- Use prebuilt profiles (e.g., CIS benchmarks) or author custom controls.
```bash
# Download a CIS profile
inspec supermarket exec dev-sec/linux-baseline
# Or generate a new profile
inspec init profile my-profile
```

#### 4. Run Compliance Scans
```bash
# Scan a remote host via SSH
inspec exec my-profile -t ssh://user@host
# Scan a Docker container
inspec exec my-profile -t docker://container_id
```

#### 5. Generate and Store Reports
```bash
# Generate HTML or JSON reports
inspec exec my-profile -t ssh://user@host --reporter html:report.html json:report.json
# Store reports as pipeline artifacts
```

#### 6. Integrate with CI/CD
Example Jenkins pipeline step:
```groovy
stage('Compliance Scan') {
  steps {
    sh 'inspec exec my-profile -t docker://my-app --reporter html:inspec-report.html'
    archiveArtifacts artifacts: 'inspec-report.html'
  }
}
```

#### 7. Remediation and Best Practices
- Review failed controls and remediate issues.
- Automate regular scans (e.g., nightly or on deployment).
- Use version control for compliance profiles.

This approach ensures your infrastructure and applications are continuously checked for compliance with security standards.

### Task 14: Blue-Green and Canary Deployments

**Implement blue-green and canary deployment strategies in Kubernetes.**

**Detailed Solution:**

#### 1. Blue-Green Deployment in Kubernetes
- Deploy two versions of your app (e.g., `blue` and `green`).
- Use labels to distinguish versions and a single Service to route traffic.

**Deployment Example:**
```yaml
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: app
        image: my-app:blue
        ports:
        - containerPort: 80
```
```yaml
# green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: app
        image: my-app:green
        ports:
        - containerPort: 80
```
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue # or green
  ports:
  - port: 80
    targetPort: 80
```

**Switch traffic:**
```bash
# Switch service selector to green
kubectl patch service my-app-service -p '{"spec":{"selector":{"app":"my-app","version":"green"}}}'
```

#### 2. Canary Deployment in Kubernetes
- Gradually shift traffic to a new version by running both old and new pods.

**Canary Deployment Example:**
```yaml
# canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: canary
  template:
    metadata:
      labels:
        app: my-app
        version: canary
    spec:
      containers:
      - name: app
        image: my-app:canary
        ports:
        - containerPort: 80
```

**Weighted Routing (with Istio or Linkerd):**
Use a service mesh to split traffic between stable and canary versions.
```yaml
# Istio VirtualService example
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app.example.com
  http:
  - route:
    - destination:
        host: my-app
        subset: stable
      weight: 90
    - destination:
        host: my-app
        subset: canary
      weight: 10
```

#### 3. Rollback
- If issues are detected, revert the service selector or scale down the canary deployment.
```bash
# Rollback to blue
kubectl patch service my-app-service -p '{"spec":{"selector":{"app":"my-app","version":"blue"}}}'
# Scale down canary
kubectl scale deployment my-app-canary --replicas=0
```

#### 4. Best Practices
- Automate traffic switching and monitoring in your CI/CD pipeline.
- Use health checks and monitoring to validate new versions before full rollout.
- Document rollback procedures and test them regularly.

This approach enables safe, low-risk deployments and fast rollback in Kubernetes environments.

### Task 15: Infrastructure Cost Optimization

**Automate cost monitoring and optimize resource usage.**

**Detailed Solution:**

#### 1. Monitor Cloud and Kubernetes Costs
- Use tools like Kubecost (Kubernetes), AWS Cost Explorer, or Azure Cost Management.
```bash
# Deploy Kubecost for Kubernetes cost monitoring
kubectl apply -f https://raw.githubusercontent.com/kubecost/cost-analyzer-helm-chart/master/kubernetes/kubecost.yaml
# Access Kubecost dashboard for real-time cost analysis
```

#### 2. Tag and Organize Resources
- Tag resources by environment, team, and project for accurate cost allocation.
- Example (Terraform):
```hcl
resource "aws_instance" "web" {
  # ...
  tags = {
    Environment = "dev"
    Team        = "backend"
    Project     = "my-app"
  }
}
```

#### 3. Automate Scaling and Scheduling
- Use auto-scaling groups and Kubernetes HPA to match resource usage to demand.
- Schedule non-production environments to scale down during off-hours.
```bash
# Scale down dev environment at night (cronjob example)
0 20 * * * kubectl scale deployment my-app -n dev --replicas=0
# Scale up in the morning
0 8 * * * kubectl scale deployment my-app -n dev --replicas=2
```

#### 4. Rightsize Resources
- Regularly review and adjust CPU/memory requests and limits in Kubernetes.
- Use cloud provider recommendations for VM sizing.

#### 5. Use Spot/Preemptible Instances
- Deploy non-critical workloads on spot/preemptible VMs for significant savings.

#### 6. Set Budgets and Alerts
- Configure budgets and alerts in your cloud provider to avoid overspending.
- Example: AWS Budgets, Azure Cost Alerts.

#### 7. Best Practices
- Review cost reports monthly.
- Remove unused resources (e.g., orphaned volumes, old snapshots).
- Use reserved instances or savings plans for predictable workloads.

This approach ensures you monitor, control, and optimize infrastructure costs proactively.

## Advanced Level Solutions (Tasks 16-20)

### Task 16: Disaster Recovery Automation

**Automate disaster recovery drills and failover.**

**Detailed Solution:**

#### 1. Define Recovery Objectives
- **RTO (Recovery Time Objective):** Maximum acceptable downtime.
- **RPO (Recovery Point Objective):** Maximum acceptable data loss.

#### 2. Automate Backups and Replication
- Enable automated backups and cross-region replication (e.g., AWS RDS Multi-AZ, S3 cross-region replication).

#### 3. Automate Failover
- Use managed services with built-in failover (e.g., AWS RDS, Azure SQL, GCP Cloud SQL).
- Example: Force failover for testing
```bash
aws rds failover-db-cluster --db-cluster-identifier mydb-cluster
```

#### 4. Restore from Backup
- Automate restoration from the latest backup in a new region or environment.
```bash
aws rds restore-db-instance-from-s3 \
  --db-instance-identifier mydb-restore \
  --s3-bucket-name my-backups \
  --s3-ingestion-role-arn arn:aws:iam::123456789012:role/MyS3Role
```

#### 5. Infrastructure as Code for Recovery
- Use Terraform/CloudFormation to recreate infrastructure in DR region.
```bash
terraform apply -var="region=us-east-2" -var="environment=dr"
```

#### 6. Validate Backups and Recovery
- Schedule regular restore drills to test backup integrity and recovery procedures.
- Automate validation (e.g., run DB checks, application smoke tests after restore).

#### 7. Document and Automate DR Runbooks
- Maintain up-to-date runbooks for disaster scenarios.
- Automate as much as possible (scripts, CI/CD jobs, cloud automation).

#### 8. Monitoring and Alerts
- Monitor replication lag, backup status, and failover events.
- Alert on failures or RTO/RPO breaches.

This approach ensures your organization can recover quickly and reliably from disasters, with minimal manual intervention.

### Task 17: Compliance as Code

**Enforce compliance policies using policy-as-code tools (e.g., OPA, Sentinel).**

**Detailed Solution:**

#### 1. Choose a Policy-as-Code Tool
- **OPA (Open Policy Agent):** Kubernetes, Terraform, microservices.
- **HashiCorp Sentinel:** Terraform Enterprise, Consul, Vault.

#### 2. Author Compliance Policies
Example: OPA policy to enforce non-root containers in Kubernetes
```rego
package kubernetes.admission
deny[msg] {
  input.request.kind.kind == "Pod"
  not input.request.object.spec.securityContext.runAsNonRoot
  msg := "Pods must not run as root."
}
```

#### 3. Enforce Policies in CI/CD
- Integrate OPA with admission controllers (e.g., Gatekeeper for Kubernetes).
- Example: Validate Kubernetes manifests before deployment
```bash
opa eval --input manifest.yaml --data policy.rego "data.kubernetes.admission.deny"
```
- For Terraform, use OPA or Sentinel to check plans
```bash
terraform plan -out=tfplan
opa eval --input tfplan.json --data terraform-policy.rego "data.terraform.deny"
```

#### 4. Integrate with CI/CD Pipeline
Example Jenkins step:
```groovy
stage('Policy Check') {
    steps {
        sh 'opa eval --input manifest.yaml --data policy.rego "data.kubernetes.admission.deny"'
    }
}
```

#### 5. Best Practices
- Store policies in version control.
- Review and update policies regularly.
- Provide clear error messages for policy violations.

This approach ensures compliance requirements are codified, versioned, and automatically enforced across your infrastructure and deployments.

### Task 18: Advanced Security Scanning

**Integrate SAST, DAST, and container scanning in CI/CD.**

**Detailed Solution:**

#### 1. Static Application Security Testing (SAST)
- Analyze source code for vulnerabilities before build/deploy.
- Example: SonarQube, CodeQL, Bandit (Python).
```bash
# SonarQube scan
sonar-scanner -Dsonar.projectKey=my-app -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=$SONAR_TOKEN
# Bandit for Python
bandit -r . -f html -o bandit-report.html
```

#### 2. Dynamic Application Security Testing (DAST)
- Test running applications for vulnerabilities (e.g., OWASP ZAP, Burp Suite).
```bash
# Run OWASP ZAP baseline scan
docker run -t owasp/zap2docker-stable zap-baseline.py -t http://my-app-url -r zap-report.html
```

#### 3. Container Image Scanning
- Scan Docker images for vulnerabilities (e.g., Trivy, Clair, Anchore).
```bash
trivy image --format html --output trivy-report.html my-app:latest
```

#### 4. Integrate Scans in CI/CD Pipeline
Example Jenkins pipeline:
```groovy
pipeline {
  agent any
  environment {
    SONAR_TOKEN = credentials('sonar-token')
  }
  stages {
    stage('SAST') {
      steps {
        sh 'sonar-scanner -Dsonar.projectKey=my-app -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=$SONAR_TOKEN'
      }
    }
    stage('DAST') {
      steps {
        sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t http://my-app-url -r zap-report.html'
        archiveArtifacts artifacts: 'zap-report.html'
      }
    }
    stage('Container Scan') {
      steps {
        sh 'trivy image --format html --output trivy-report.html my-app:latest'
        archiveArtifacts artifacts: 'trivy-report.html'
      }
    }
  }
}
```

#### 5. Reporting and Remediation
- Store scan reports as pipeline artifacts.
- Fail builds on critical vulnerabilities.
- Review and fix issues before production deployment.

#### 6. Best Practices
- Automate scans on every commit and pull request.
- Regularly update scanning tools and vulnerability databases.
- Integrate with Slack/email for real-time alerts.

This approach ensures comprehensive security coverage across code, running applications, and container images.

### Task 19: Performance Optimization and Load Testing

**Automate load testing and performance tuning.**

**Detailed Solution:**

#### 1. Define Performance Goals
- Set SLAs for response time, throughput, and error rates.

#### 2. Load Testing
- Use tools like k6, JMeter, or Locust to simulate user traffic.
```bash
# Example: k6 load test script (loadtest.js)
import http from 'k6/http';
import { sleep } from 'k6';

export let options = {
  vus: 50,
  duration: '5m',
};

export default function () {
  http.get('http://my-app-url/health');
  sleep(1);
}
# Run the test
k6 run loadtest.js
```

#### 3. Monitor Resource Usage
- Use `kubectl top` or cloud monitoring tools to observe CPU, memory, and network usage during tests.
```bash
kubectl top pods -n prod
kubectl top nodes
```

#### 4. Analyze Results and Tune
- Identify bottlenecks (e.g., slow endpoints, high CPU/memory usage).
- Tune application code, database queries, and infrastructure.
- Adjust Kubernetes resource requests/limits:
```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"
```

#### 5. Automate Performance Testing in CI/CD
- Run load tests as part of the pipeline for every release.
- Fail builds if performance thresholds are not met.

#### 6. Best Practices
- Test with production-like data and environments.
- Monitor application and infrastructure metrics during tests.
- Document and track performance improvements over time.

This approach ensures your application meets performance goals and scales reliably under load.

### Task 20: GitOps Implementation

**Implement GitOps workflows using ArgoCD or Flux.**

**Detailed Solution:**

#### 1. Install ArgoCD (GitOps Controller)
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
# Expose ArgoCD API server (NodePort or Ingress)
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

#### 2. Login and Configure Access
```bash
# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
# Login via CLI
argocd login localhost:8080
```

#### 3. Create a GitOps Application
Define an ArgoCD Application manifest (e.g., `argocd-app.yaml`):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/myorg/my-app-manifests.git'
    targetRevision: HEAD
    path: k8s
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
Apply the manifest:
```bash
kubectl apply -f argocd-app.yaml -n argocd
```

#### 4. Sync and Manage Applications
```bash
# Sync application state to match Git
argocd app sync my-app
# View status
argocd app get my-app
```

#### 5. Flux Alternative
- Install Flux with the Flux CLI and bootstrap your repo.
```bash
flux bootstrap github --owner=myorg --repository=my-app-manifests --branch=main --path=./k8s
```

#### 6. Best Practices
- Store all Kubernetes manifests/Helm charts in Git.
- Use separate branches or folders for different environments.
- Enable automated sync and self-healing.
- Protect production branches with PR reviews.

This approach ensures all cluster changes are versioned, auditable, and automatically applied from Git, enabling true GitOps workflows.


---

**Note**: All solutions follow DevOps best practices including version control, automated testing, security scanning, and proper monitoring. Customize the configurations according to your specific infrastructure and requirements.