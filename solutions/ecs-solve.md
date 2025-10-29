# Docker ECS Tasks Solutions

This document contains detailed solutions for all 20 Docker ECS practice tasks.

## Basic Level Solutions (Tasks 1-10)

### Task 1: Docker Installation and Basic Commands
**Install Docker and master fundamental Docker commands.**

**Solution:**
```bash
# Install Docker on Ubuntu/Debian
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Install Docker on CentOS/RHEL
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (avoid using sudo)
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker info
docker run hello-world

# Basic Docker commands
echo "=== Docker Basic Commands Reference ==="

# Image operations
docker images                           # List local images
docker pull ubuntu:20.04               # Download image
docker rmi ubuntu:20.04                 # Remove image
docker image prune                      # Remove unused images
docker search nginx                     # Search Docker Hub

# Container operations
docker run ubuntu:20.04 echo "Hello"   # Run container with command
docker run -it ubuntu:20.04 bash       # Interactive container
docker run -d nginx                     # Run in background (detached)
docker run --name my-nginx -d nginx    # Run with custom name
docker run -p 8080:80 nginx            # Port mapping (host:container)
docker run -v /host/path:/container/path nginx  # Volume mounting

# Container management
docker ps                              # List running containers
docker ps -a                           # List all containers
docker start container_name            # Start stopped container
docker stop container_name             # Stop running container
docker restart container_name          # Restart container
docker rm container_name               # Remove container
docker rm -f container_name            # Force remove running container

# Container inspection
docker logs container_name             # View container logs
docker logs -f container_name          # Follow logs
docker exec -it container_name bash   # Execute command in running container
docker inspect container_name          # Detailed container info
docker stats                           # Real-time resource usage

# Container networking
docker network ls                      # List networks
docker network create my-network       # Create custom network
docker run --network my-network nginx  # Connect container to network

# Volume management
docker volume ls                       # List volumes
docker volume create my-volume         # Create named volume
docker volume inspect my-volume        # Inspect volume
docker run -v my-volume:/data nginx   # Use named volume

# System commands
docker system df                       # Show disk usage
docker system prune                    # Clean up unused resources
docker system prune -a                 # Aggressive cleanup

# Practice script for Docker basics
cat > docker_basics_practice.sh << 'EOF'
#!/bin/bash

echo "=== Docker Basics Practice ==="

# 1. Run a simple container
echo "1. Running Ubuntu container..."
docker run --rm ubuntu:20.04 cat /etc/os-release

# 2. Run interactive container
echo -e "\n2. Creating interactive environment..."
docker run -it --name practice-ubuntu ubuntu:20.04 bash -c "
    apt update
    apt install -y curl
    curl -s https://httpbin.org/ip
    echo 'Practice completed!'
"

# 3. Clean up
docker rm practice-ubuntu 2>/dev/null

# 4. Run web server
echo -e "\n3. Starting web server..."
docker run -d --name practice-nginx -p 8080:80 nginx
echo "Nginx started on http://localhost:8080"

# 5. Check logs
echo -e "\n4. Checking nginx logs..."
sleep 2
docker logs practice-nginx

# 6. Test web server
echo -e "\n5. Testing web server..."
curl -s http://localhost:8080 | head -5

# 7. Stop and cleanup
echo -e "\n6. Cleaning up..."
docker stop practice-nginx
docker rm practice-nginx

echo -e "\nDocker basics practice completed!"
EOF

chmod +x docker_basics_practice.sh
./docker_basics_practice.sh
```

### Task 2: Dockerfile Creation and Best Practices
**Create optimized Dockerfiles following best practices.**

**Solution:**
```dockerfile
# Multi-stage Dockerfile example for Node.js application
# Dockerfile.nodejs
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine AS production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Set working directory
WORKDIR /app

# Copy built application from builder stage
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package*.json ./

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Start application
CMD ["node", "dist/index.js"]
```

```dockerfile
# Python application Dockerfile
# Dockerfile.python
FROM python:3.11-slim AS base

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Create app user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Development stage
FROM base AS development

# Install development dependencies
RUN pip install poetry

WORKDIR /app

# Copy poetry files
COPY pyproject.toml poetry.lock ./

# Install dependencies
RUN poetry config virtualenvs.create false && \
    poetry install --no-dev

# Copy source code
COPY . .

# Change ownership
RUN chown -R appuser:appuser /app

USER appuser

CMD ["python", "app.py"]

# Production stage
FROM base AS production

WORKDIR /app

# Copy requirements file
COPY requirements.txt .

# Install production dependencies
RUN pip install --no-deps -r requirements.txt

# Copy application
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python healthcheck.py

# Run application
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

```dockerfile
# Go application Dockerfile
# Dockerfile.go
FROM golang:1.21-alpine AS builder

# Install ca-certificates and git
RUN apk add --no-cache ca-certificates git

# Set working directory
WORKDIR /build

# Copy go mod files
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-extldflags "-static"' -o app .

# Final stage
FROM scratch

# Copy ca-certificates from builder
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy binary
COPY --from=builder /build/app /app

# Expose port
EXPOSE 8080

# Health check (for containers with shell)
# HEALTHCHECK --interval=30s --timeout=3s CMD ["/app", "health"]

# Run binary
ENTRYPOINT ["/app"]
```

```bash
# Docker best practices script
cat > dockerfile_best_practices.sh << 'EOF'
#!/bin/bash

echo "=== Dockerfile Best Practices Checker ==="

check_dockerfile() {
    local dockerfile=$1
    
    echo "Checking: $dockerfile"
    echo "========================"
    
    # Check if Dockerfile exists
    if [[ ! -f "$dockerfile" ]]; then
        echo "❌ Dockerfile not found: $dockerfile"
        return 1
    fi
    
    # Best practice checks
    echo "🔍 Running best practice checks..."
    
    # Check for FROM instruction
    if grep -q "^FROM" "$dockerfile"; then
        echo "✅ Has FROM instruction"
    else
        echo "❌ Missing FROM instruction"
    fi
    
    # Check for specific base image version
    if grep -q "FROM.*:latest" "$dockerfile"; then
        echo "⚠️  Using 'latest' tag (consider specific version)"
    else
        echo "✅ Using specific image version"
    fi
    
    # Check for non-root user
    if grep -q "USER" "$dockerfile"; then
        echo "✅ Sets non-root user"
    else
        echo "⚠️  Consider setting non-root user"
    fi
    
    # Check for HEALTHCHECK
    if grep -q "HEALTHCHECK" "$dockerfile"; then
        echo "✅ Has health check"
    else
        echo "⚠️  Consider adding health check"
    fi
    
    # Check for .dockerignore
    if [[ -f ".dockerignore" ]]; then
        echo "✅ Has .dockerignore file"
    else
        echo "⚠️  Consider adding .dockerignore file"
    fi
    
    # Check for multi-stage build
    if grep -c "^FROM" "$dockerfile" | grep -q "[2-9]"; then
        echo "✅ Uses multi-stage build"
    else
        echo "⚠️  Consider multi-stage build for optimization"
    fi
    
    # Check for COPY instead of ADD
    if grep -q "^ADD" "$dockerfile" && ! grep -q "\.tar\|\.zip" "$dockerfile"; then
        echo "⚠️  Consider using COPY instead of ADD"
    else
        echo "✅ Uses appropriate COPY/ADD"
    fi
    
    # Check for layer optimization
    local run_commands=$(grep -c "^RUN" "$dockerfile")
    if [[ $run_commands -gt 5 ]]; then
        echo "⚠️  Consider combining RUN commands to reduce layers"
    else
        echo "✅ Reasonable number of RUN commands"
    fi
    
    echo
}

# Create sample .dockerignore
create_dockerignore() {
    cat > .dockerignore << 'IGNORE'
# Version control
.git
.gitignore

# Dependencies
node_modules
vendor
__pycache__
*.pyc

# Build artifacts
dist
build
target
*.exe

# IDE files
.vscode
.idea
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs

# Documentation
README.md
docs
*.md

# Test files
tests
test
spec
*.test

# Configuration files (sensitive)
.env
.env.local
config.json
secrets.txt
IGNORE

    echo "✅ Created .dockerignore file"
}

# Security scanning function
security_scan() {
    local image_name=$1
    
    echo "🔒 Security scanning: $image_name"
    
    # Using hadolint for Dockerfile linting
    if command -v hadolint > /dev/null; then
        echo "Running hadolint..."
        hadolint Dockerfile
    else
        echo "Install hadolint for Dockerfile linting: docker pull hadolint/hadolint"
    fi
    
    # Using docker scout (if available)
    if docker scout version > /dev/null 2>&1; then
        echo "Running Docker Scout..."
        docker scout cves "$image_name"
    fi
    
    # Using trivy (if available)
    if command -v trivy > /dev/null; then
        echo "Running Trivy scan..."
        trivy image "$image_name"
    fi
}

# Build optimization function
optimize_build() {
    local dockerfile=$1
    local image_name=$2
    
    echo "🚀 Building optimized image: $image_name"
    
    # Build with BuildKit
    DOCKER_BUILDKIT=1 docker build \
        --file "$dockerfile" \
        --target production \
        --tag "$image_name" \
        --progress=plain \
        .
    
    # Show image size
    echo "📊 Image size:"
    docker images "$image_name" --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
}

# Main execution
case ${1:-check} in
    check)
        check_dockerfile "${2:-Dockerfile}"
        ;;
    ignore)
        create_dockerignore
        ;;
    scan)
        security_scan "${2:-myapp:latest}"
        ;;
    build)
        optimize_build "${2:-Dockerfile}" "${3:-myapp:latest}"
        ;;
    *)
        echo "Dockerfile Best Practices Tool"
        echo "Usage: $0 {check|ignore|scan|build} [dockerfile|image]"
        echo "  check  - Check Dockerfile best practices"
        echo "  ignore - Create .dockerignore file"
        echo "  scan   - Security scan image"
        echo "  build  - Build optimized image"
        ;;
esac
EOF

chmod +x dockerfile_best_practices.sh

# Create sample applications
mkdir -p sample-apps/{nodejs,python,go}

# Node.js sample app
cat > sample-apps/nodejs/package.json << 'EOF'
{
  "name": "sample-nodejs-app",
  "version": "1.0.0",
  "description": "Sample Node.js application for Docker",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "build": "echo 'Build completed'"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

cat > sample-apps/nodejs/index.js << 'EOF'
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Node.js Docker container!' });
});

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy' });
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});
EOF

cp Dockerfile.nodejs sample-apps/nodejs/Dockerfile

echo "Docker basics and Dockerfile best practices setup completed!"
echo "Run ./dockerfile_best_practices.sh to check Dockerfile quality"
```

### Task 3: Docker Compose for Multi-Container Applications
**Create Docker Compose configurations for complex applications.**

**Solution:**
```yaml
# docker-compose.yml - Complete web application stack
version: '3.8'

services:
  # Reverse proxy
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - nginx-logs:/var/log/nginx
    depends_on:
      - webapp
      - api
    networks:
      - frontend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Web application
  webapp:
    build:
      context: ./webapp
      dockerfile: Dockerfile
      target: production
    container_name: webapp
    environment:
      - NODE_ENV=production
      - API_URL=http://api:3000
    volumes:
      - webapp-uploads:/app/uploads
    networks:
      - frontend
      - backend
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # API service
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    container_name: api-service
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@database:5432/myapp
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET_FILE=/run/secrets/jwt_secret
    volumes:
      - api-logs:/app/logs
    networks:
      - backend
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_healthy
    secrets:
      - jwt_secret
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Database
  database:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d:ro
    networks:
      - backend
    secrets:
      - db_password
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d myapp"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: redis-cache
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD:-defaultpass}
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Background worker
  worker:
    build:
      context: ./worker
      dockerfile: Dockerfile
    container_name: bg-worker
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@database:5432/myapp
      - REDIS_URL=redis://redis:6379
    volumes:
      - worker-logs:/app/logs
    networks:
      - backend
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      replicas: 2

  # Monitoring
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    networks:
      - monitoring
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
    ports:
      - "3001:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
      - ./monitoring/grafana/datasources:/etc/grafana/provisioning/datasources:ro
    networks:
      - monitoring
    restart: unless-stopped

# Networks
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
  monitoring:
    driver: bridge

# Volumes
volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
  nginx-logs:
    driver: local
  webapp-uploads:
    driver: local
  api-logs:
    driver: local
  worker-logs:
    driver: local
  prometheus-data:
    driver: local
  grafana-data:
    driver: local

# Secrets
secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

```yaml
# docker-compose.override.yml - Development overrides
version: '3.8'

services:
  webapp:
    build:
      target: development
    environment:
      - NODE_ENV=development
    volumes:
      - ./webapp:/app
      - /app/node_modules
    command: npm run dev
    ports:
      - "3000:3000"

  api:
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    volumes:
      - ./api:/app
      - /app/node_modules
    command: npm run dev
    ports:
      - "3001:3000"

  database:
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_PASSWORD=devpassword

  redis:
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
```

```bash
# Docker Compose management script
cat > docker_compose_manager.sh << 'EOF'
#!/bin/bash

# Docker Compose management script

COMPOSE_FILE="docker-compose.yml"
PROJECT_NAME="myapp"

# Function to setup project structure
setup_project() {
    echo "🚀 Setting up Docker Compose project structure..."
    
    # Create directories
    mkdir -p {webapp,api,worker,database/init,nginx,monitoring/{grafana/{dashboards,datasources}},secrets}
    
    # Create nginx configuration
    cat > nginx/nginx.conf << 'NGINX'
events {
    worker_connections 1024;
}

http {
    upstream webapp {
        server webapp:3000;
    }
    
    upstream api {
        server api:3000;
    }
    
    server {
        listen 80;
        
        location / {
            proxy_pass http://webapp;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        
        location /api/ {
            proxy_pass http://api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
NGINX

    # Create Prometheus configuration
    cat > monitoring/prometheus.yml << 'PROM'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'webapp'
    static_configs:
      - targets: ['webapp:3000']
  
  - job_name: 'api'
    static_configs:
      - targets: ['api:3000']
      
  - job_name: 'postgres'
    static_configs:
      - targets: ['database:5432']
PROM

    # Create secrets
    echo "supersecretjwtkey" > secrets/jwt_secret.txt
    echo "supersecretdbpassword" > secrets/db_password.txt
    
    # Create environment file
    cat > .env << 'ENV'
COMPOSE_PROJECT_NAME=myapp
POSTGRES_PASSWORD=supersecretdbpassword
REDIS_PASSWORD=supersecretredispassword
GRAFANA_PASSWORD=admin
ENV

    echo "✅ Project structure created"
}

# Function to validate compose file
validate_compose() {
    echo "🔍 Validating Docker Compose configuration..."
    
    if docker-compose -f "$COMPOSE_FILE" config > /dev/null 2>&1; then
        echo "✅ Docker Compose configuration is valid"
    else
        echo "❌ Docker Compose configuration has errors:"
        docker-compose -f "$COMPOSE_FILE" config
        return 1
    fi
}

# Function to start services
start_services() {
    local profile=${1:-production}
    
    echo "🚀 Starting services with profile: $profile"
    
    if [[ "$profile" == "development" ]]; then
        docker-compose -f "$COMPOSE_FILE" -f docker-compose.override.yml up -d
    else
        docker-compose -f "$COMPOSE_FILE" up -d
    fi
    
    echo "⏳ Waiting for services to be healthy..."
    sleep 10
    
    # Check service health
    check_health
}

# Function to check service health
check_health() {
    echo "🏥 Checking service health..."
    
    services=("webapp" "api" "database" "redis")
    
    for service in "${services[@]}"; do
        if docker-compose ps "$service" | grep -q "healthy\|Up"; then
            echo "✅ $service is healthy"
        else
            echo "❌ $service is not healthy"
            docker-compose logs "$service" | tail -5
        fi
    done
}

# Function to scale services
scale_services() {
    local service=$1
    local replicas=$2
    
    echo "📈 Scaling $service to $replicas replicas..."
    docker-compose up -d --scale "$service=$replicas"
}

# Function to show logs
show_logs() {
    local service=${1:-}
    
    if [[ -n "$service" ]]; then
        docker-compose logs -f "$service"
    else
        docker-compose logs -f
    fi
}

# Function to backup data
backup_data() {
    local backup_dir="backups/$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$backup_dir"
    
    echo "💾 Creating backup in $backup_dir..."
    
    # Backup database
    docker-compose exec -T database pg_dump -U postgres myapp > "$backup_dir/database.sql"
    
    # Backup volumes
    docker run --rm -v myapp_postgres-data:/data -v "$PWD/$backup_dir":/backup alpine tar czf /backup/postgres-data.tar.gz -C /data .
    docker run --rm -v myapp_redis-data:/data -v "$PWD/$backup_dir":/backup alpine tar czf /backup/redis-data.tar.gz -C /data .
    
    echo "✅ Backup completed: $backup_dir"
}

# Function to restore data
restore_data() {
    local backup_dir=$1
    
    if [[ ! -d "$backup_dir" ]]; then
        echo "❌ Backup directory not found: $backup_dir"
        return 1
    fi
    
    echo "📥 Restoring from backup: $backup_dir"
    
    # Stop services
    docker-compose stop database redis
    
    # Restore database
    if [[ -f "$backup_dir/database.sql" ]]; then
        docker-compose start database
        sleep 10
        docker-compose exec -T database psql -U postgres -d myapp < "$backup_dir/database.sql"
    fi
    
    # Restore volumes
    if [[ -f "$backup_dir/postgres-data.tar.gz" ]]; then
        docker run --rm -v myapp_postgres-data:/data -v "$PWD/$backup_dir":/backup alpine tar xzf /backup/postgres-data.tar.gz -C /data
    fi
    
    if [[ -f "$backup_dir/redis-data.tar.gz" ]]; then
        docker run --rm -v myapp_redis-data:/data -v "$PWD/$backup_dir":/backup alpine tar xzf /backup/redis-data.tar.gz -C /data
    fi
    
    # Restart services
    docker-compose up -d
    
    echo "✅ Restore completed"
}

# Function to cleanup
cleanup() {
    echo "🧹 Cleaning up..."
    
    # Stop and remove containers
    docker-compose down
    
    # Remove unused images and volumes
    docker system prune -f
    
    # Remove project volumes (optional)
    read -p "Remove project volumes? (y/N): " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        docker-compose down -v
    fi
}

# Main menu
case ${1:-help} in
    setup)
        setup_project
        ;;
    validate)
        validate_compose
        ;;
    start)
        start_services "${2:-production}"
        ;;
    stop)
        docker-compose stop
        ;;
    restart)
        docker-compose restart "${2:-}"
        ;;
    logs)
        show_logs "$2"
        ;;
    health)
        check_health
        ;;
    scale)
        scale_services "$2" "$3"
        ;;
    backup)
        backup_data
        ;;
    restore)
        restore_data "$2"
        ;;
    cleanup)
        cleanup
        ;;
    *)
        echo "Docker Compose Manager"
        echo "Usage: $0 {setup|validate|start|stop|restart|logs|health|scale|backup|restore|cleanup}"
        echo ""
        echo "Commands:"
        echo "  setup      - Create project structure"
        echo "  validate   - Validate compose configuration"
        echo "  start      - Start services [development|production]"
        echo "  stop       - Stop all services"
        echo "  restart    - Restart services [service_name]"
        echo "  logs       - Show logs [service_name]"
        echo "  health     - Check service health"
        echo "  scale      - Scale service [service_name] [replicas]"
        echo "  backup     - Backup data and volumes"
        echo "  restore    - Restore from backup [backup_dir]"
        echo "  cleanup    - Clean up containers and images"
        ;;
esac
EOF

chmod +x docker_compose_manager.sh

echo "Docker Compose multi-container setup completed!"
echo "Run ./docker_compose_manager.sh setup to create project structure"
echo "Run ./docker_compose_manager.sh start to launch the application stack"
```

### Task 4: AWS ECS Cluster Setup
**Create and configure an Amazon ECS cluster with proper networking.**

**Solution:**
```bash
# ECS cluster setup script
cat > ecs_cluster_setup.sh << 'EOF'
#!/bin/bash

# Variables
CLUSTER_NAME="my-ecs-cluster"
REGION="us-west-2"
VPC_CIDR="10.0.0.0/16"
SUBNET_1_CIDR="10.0.1.0/24"
SUBNET_2_CIDR="10.0.2.0/24"

echo "=== Setting up ECS Cluster: $CLUSTER_NAME ==="

# Create VPC for ECS cluster
echo "1. Creating VPC..."
VPC_ID=$(aws ec2 create-vpc \
    --cidr-block $VPC_CIDR \
    --query 'Vpc.VpcId' \
    --output text \
    --region $REGION)

aws ec2 create-tags \
    --resources $VPC_ID \
    --tags Key=Name,Value=ecs-vpc \
    --region $REGION

echo "VPC created: $VPC_ID"

# Enable DNS hostname resolution
aws ec2 modify-vpc-attribute \
    --vpc-id $VPC_ID \
    --enable-dns-hostnames \
    --region $REGION

# Create Internet Gateway
echo "2. Creating Internet Gateway..."
IGW_ID=$(aws ec2 create-internet-gateway \
    --query 'InternetGateway.InternetGatewayId' \
    --output text \
    --region $REGION)

aws ec2 attach-internet-gateway \
    --vpc-id $VPC_ID \
    --internet-gateway-id $IGW_ID \
    --region $REGION

aws ec2 create-tags \
    --resources $IGW_ID \
    --tags Key=Name,Value=ecs-igw \
    --region $REGION

echo "Internet Gateway created: $IGW_ID"

# Get Availability Zones
AZ_1=$(aws ec2 describe-availability-zones \
    --region $REGION \
    --query 'AvailabilityZones[0].ZoneName' \
    --output text)

AZ_2=$(aws ec2 describe-availability-zones \
    --region $REGION \
    --query 'AvailabilityZones[1].ZoneName' \
    --output text)

# Create Subnets
echo "3. Creating subnets..."
SUBNET_1_ID=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block $SUBNET_1_CIDR \
    --availability-zone $AZ_1 \
    --query 'Subnet.SubnetId' \
    --output text \
    --region $REGION)

SUBNET_2_ID=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block $SUBNET_2_CIDR \
    --availability-zone $AZ_2 \
    --query 'Subnet.SubnetId' \
    --output text \
    --region $REGION)

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute \
    --subnet-id $SUBNET_1_ID \
    --map-public-ip-on-launch \
    --region $REGION

aws ec2 modify-subnet-attribute \
    --subnet-id $SUBNET_2_ID \
    --map-public-ip-on-launch \
    --region $REGION

# Tag subnets
aws ec2 create-tags \
    --resources $SUBNET_1_ID \
    --tags Key=Name,Value=ecs-subnet-1 \
    --region $REGION

aws ec2 create-tags \
    --resources $SUBNET_2_ID \
    --tags Key=Name,Value=ecs-subnet-2 \
    --region $REGION

echo "Subnets created: $SUBNET_1_ID, $SUBNET_2_ID"

# Create Route Table
echo "4. Creating route table..."
ROUTE_TABLE_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' \
    --output text \
    --region $REGION)

# Add route to Internet Gateway
aws ec2 create-route \
    --route-table-id $ROUTE_TABLE_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID \
    --region $REGION

# Associate subnets with route table
aws ec2 associate-route-table \
    --subnet-id $SUBNET_1_ID \
    --route-table-id $ROUTE_TABLE_ID \
    --region $REGION

aws ec2 associate-route-table \
    --subnet-id $SUBNET_2_ID \
    --route-table-id $ROUTE_TABLE_ID \
    --region $REGION

aws ec2 create-tags \
    --resources $ROUTE_TABLE_ID \
    --tags Key=Name,Value=ecs-route-table \
    --region $REGION

echo "Route table configured: $ROUTE_TABLE_ID"

# Create Security Group for ECS tasks
echo "5. Creating security groups..."
ECS_SG_ID=$(aws ec2 create-security-group \
    --group-name ecs-tasks-sg \
    --description "Security group for ECS tasks" \
    --vpc-id $VPC_ID \
    --query 'GroupId' \
    --output text \
    --region $REGION)

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
    --group-id $ECS_SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0 \
    --region $REGION

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
    --group-id $ECS_SG_ID \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0 \
    --region $REGION

# Allow custom application port
aws ec2 authorize-security-group-ingress \
    --group-id $ECS_SG_ID \
    --protocol tcp \
    --port 3000 \
    --cidr 0.0.0.0/0 \
    --region $REGION

aws ec2 create-tags \
    --resources $ECS_SG_ID \
    --tags Key=Name,Value=ecs-tasks-sg \
    --region $REGION

echo "Security group created: $ECS_SG_ID"

# Create ECS Cluster
echo "6. Creating ECS cluster..."
aws ecs create-cluster \
    --cluster-name $CLUSTER_NAME \
    --capacity-providers FARGATE EC2 \
    --default-capacity-provider-strategy capacityProvider=FARGATE,weight=1 \
    --region $REGION

echo "ECS cluster created: $CLUSTER_NAME"

# Create IAM role for ECS task execution
echo "7. Creating IAM roles..."

# ECS Task Execution Role
cat > trust-policy.json << 'TRUST'
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
TRUST

aws iam create-role \
    --role-name ecsTaskExecutionRole \
    --assume-role-policy-document file://trust-policy.json \
    --region $REGION 2>/dev/null || echo "Role already exists"

aws iam attach-role-policy \
    --role-name ecsTaskExecutionRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy \
    --region $REGION

# ECS Task Role (for application permissions)
aws iam create-role \
    --role-name ecsTaskRole \
    --assume-role-policy-document file://trust-policy.json \
    --region $REGION 2>/dev/null || echo "Role already exists"

# Clean up
rm trust-policy.json

echo "IAM roles created"

# Create CloudWatch Log Group
echo "8. Creating CloudWatch log group..."
aws logs create-log-group \
    --log-group-name "/ecs/$CLUSTER_NAME" \
    --region $REGION 2>/dev/null || echo "Log group already exists"

echo "CloudWatch log group created"

# Output configuration details
echo ""
echo "=== ECS Cluster Setup Complete ==="
echo "Cluster Name: $CLUSTER_NAME"
echo "Region: $REGION"
echo "VPC ID: $VPC_ID"
echo "Subnet IDs: $SUBNET_1_ID, $SUBNET_2_ID"
echo "Security Group ID: $ECS_SG_ID"
echo ""
echo "Save these values for task definition and service creation!"

# Save configuration to file
cat > ecs-config.env << CONFIG
CLUSTER_NAME=$CLUSTER_NAME
REGION=$REGION
VPC_ID=$VPC_ID
SUBNET_1_ID=$SUBNET_1_ID
SUBNET_2_ID=$SUBNET_2_ID
ECS_SG_ID=$ECS_SG_ID
CONFIG

echo "Configuration saved to ecs-config.env"
EOF

chmod +x ecs_cluster_setup.sh
```

### Task 5: ECS Task Definition and Service
**Create ECS task definitions and deploy services with proper configuration.**

**Solution:**
```json
{
  "family": "webapp-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "webapp",
      "image": "nginx:alpine",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-ecs-cluster",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "webapp"
        }
      },
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -f http://localhost/ || exit 1"
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      },
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:us-west-2:123456789012:secret:database-url-abc123"
        }
      ]
    }
  ]
}
```

```bash
# ECS task and service management script
cat > ecs_service_manager.sh << 'EOF'
#!/bin/bash

# Load configuration
if [[ -f "ecs-config.env" ]]; then
    source ecs-config.env
else
    echo "Error: ecs-config.env not found. Run ecs_cluster_setup.sh first."
    exit 1
fi

# Function to create task definition
create_task_definition() {
    local app_name=$1
    local image_uri=$2
    local cpu=${3:-256}
    local memory=${4:-512}
    
    echo "📝 Creating task definition for $app_name..."
    
    # Get account ID
    ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
    
    cat > "${app_name}-task-definition.json" << EOF
{
  "family": "${app_name}-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "${cpu}",
  "memory": "${memory}",
  "executionRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "${app_name}",
      "image": "${image_uri}",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/${CLUSTER_NAME}",
          "awslogs-region": "${REGION}",
          "awslogs-stream-prefix": "${app_name}"
        }
      },
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -f http://localhost/ || exit 1"
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      },
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        },
        {
          "name": "PORT",
          "value": "80"
        }
      ]
    }
  ]
}
EOF

    # Register task definition
    TASK_DEF_ARN=$(aws ecs register-task-definition \
        --cli-input-json "file://${app_name}-task-definition.json" \
        --region $REGION \
        --query 'taskDefinition.taskDefinitionArn' \
        --output text)
    
    echo "✅ Task definition created: $TASK_DEF_ARN"
    echo "$TASK_DEF_ARN" > "${app_name}-task-arn.txt"
}

# Function to create ECS service
create_service() {
    local app_name=$1
    local desired_count=${2:-2}
    
    echo "🚀 Creating ECS service for $app_name..."
    
    # Read task definition ARN
    if [[ ! -f "${app_name}-task-arn.txt" ]]; then
        echo "Error: Task definition ARN not found. Create task definition first."
        return 1
    fi
    
    TASK_DEF_ARN=$(cat "${app_name}-task-arn.txt")
    
    cat > "${app_name}-service.json" << EOF
{
  "serviceName": "${app_name}-service",
  "cluster": "${CLUSTER_NAME}",
  "taskDefinition": "${TASK_DEF_ARN}",
  "desiredCount": ${desired_count},
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["${SUBNET_1_ID}", "${SUBNET_2_ID}"],
      "securityGroups": ["${ECS_SG_ID}"],
      "assignPublicIp": "ENABLED"
    }
  },
  "deploymentConfiguration": {
    "maximumPercent": 200,
    "minimumHealthyPercent": 50
  },
  "healthCheckGracePeriodSeconds": 60
}
EOF

    # Create service
    aws ecs create-service \
        --cli-input-json "file://${app_name}-service.json" \
        --region $REGION

    echo "✅ ECS service created: ${app_name}-service"
}

# Function to create service with Application Load Balancer
create_service_with_alb() {
    local app_name=$1
    local desired_count=${2:-2}
    
    echo "🔗 Creating ECS service with ALB for $app_name..."
    
    # Create Application Load Balancer
    echo "Creating ALB..."
    ALB_ARN=$(aws elbv2 create-load-balancer \
        --name "${app_name}-alb" \
        --subnets $SUBNET_1_ID $SUBNET_2_ID \
        --security-groups $ECS_SG_ID \
        --region $REGION \
        --query 'LoadBalancers[0].LoadBalancerArn' \
        --output text)
    
    # Create target group
    echo "Creating target group..."
    TG_ARN=$(aws elbv2 create-target-group \
        --name "${app_name}-tg" \
        --protocol HTTP \
        --port 80 \
        --vpc-id $VPC_ID \
        --target-type ip \
        --health-check-path / \
        --health-check-interval-seconds 30 \
        --health-check-timeout-seconds 5 \
        --healthy-threshold-count 2 \
        --unhealthy-threshold-count 5 \
        --region $REGION \
        --query 'TargetGroups[0].TargetGroupArn' \
        --output text)
    
    # Create listener
    echo "Creating ALB listener..."
    aws elbv2 create-listener \
        --load-balancer-arn $ALB_ARN \
        --protocol HTTP \
        --port 80 \
        --default-actions Type=forward,TargetGroupArn=$TG_ARN \
        --region $REGION
    
    # Read task definition ARN
    TASK_DEF_ARN=$(cat "${app_name}-task-arn.txt")
    
    # Create service with load balancer
    cat > "${app_name}-service-alb.json" << EOF
{
  "serviceName": "${app_name}-service",
  "cluster": "${CLUSTER_NAME}",
  "taskDefinition": "${TASK_DEF_ARN}",
  "desiredCount": ${desired_count},
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["${SUBNET_1_ID}", "${SUBNET_2_ID}"],
      "securityGroups": ["${ECS_SG_ID}"],
      "assignPublicIp": "ENABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "${TG_ARN}",
      "containerName": "${app_name}",
      "containerPort": 80
    }
  ],
  "deploymentConfiguration": {
    "maximumPercent": 200,
    "minimumHealthyPercent": 50
  },
  "healthCheckGracePeriodSeconds": 60
}
EOF

    # Create service
    aws ecs create-service \
        --cli-input-json "file://${app_name}-service-alb.json" \
        --region $REGION

    # Get ALB DNS name
    ALB_DNS=$(aws elbv2 describe-load-balancers \
        --load-balancer-arns $ALB_ARN \
        --region $REGION \
        --query 'LoadBalancers[0].DNSName' \
        --output text)

    echo "✅ ECS service with ALB created"
    echo "🌐 Application URL: http://$ALB_DNS"
    
    # Save ALB info
    echo "ALB_ARN=$ALB_ARN" >> "${app_name}-alb-info.env"
    echo "TG_ARN=$TG_ARN" >> "${app_name}-alb-info.env"
    echo "ALB_DNS=$ALB_DNS" >> "${app_name}-alb-info.env"
}

# Function to update service
update_service() {
    local app_name=$1
    local new_image=$2
    
    echo "🔄 Updating service: $app_name with new image: $new_image"
    
    # Create new task definition with updated image
    create_task_definition "$app_name" "$new_image"
    
    # Update service
    NEW_TASK_DEF_ARN=$(cat "${app_name}-task-arn.txt")
    
    aws ecs update-service \
        --cluster $CLUSTER_NAME \
        --service "${app_name}-service" \
        --task-definition "$NEW_TASK_DEF_ARN" \
        --region $REGION
    
    echo "✅ Service update initiated"
}

# Function to scale service
scale_service() {
    local app_name=$1
    local desired_count=$2
    
    echo "📈 Scaling service: $app_name to $desired_count tasks"
    
    aws ecs update-service \
        --cluster $CLUSTER_NAME \
        --service "${app_name}-service" \
        --desired-count $desired_count \
        --region $REGION
    
    echo "✅ Service scaling initiated"
}

# Function to check service status
check_service_status() {
    local app_name=$1
    
    echo "📊 Checking service status: $app_name"
    
    aws ecs describe-services \
        --cluster $CLUSTER_NAME \
        --services "${app_name}-service" \
        --region $REGION \
        --query 'services[0].{Status:status,Running:runningCount,Pending:pendingCount,Desired:desiredCount}'
}

# Function to view service logs
view_logs() {
    local app_name=$1
    local lines=${2:-50}
    
    echo "📋 Viewing logs for: $app_name (last $lines lines)"
    
    aws logs tail "/ecs/${CLUSTER_NAME}" \
        --log-stream-name-prefix "$app_name" \
        --since 1h \
        --region $REGION
}

# Function to stop service
stop_service() {
    local app_name=$1
    
    echo "⏹️  Stopping service: $app_name"
    
    aws ecs update-service \
        --cluster $CLUSTER_NAME \
        --service "${app_name}-service" \
        --desired-count 0 \
        --region $REGION
    
    echo "✅ Service stopped"
}

# Function to delete service
delete_service() {
    local app_name=$1
    
    echo "🗑️  Deleting service: $app_name"
    
    # Scale down to 0 first
    stop_service "$app_name"
    
    # Wait for tasks to stop
    echo "Waiting for tasks to stop..."
    aws ecs wait services-stable \
        --cluster $CLUSTER_NAME \
        --services "${app_name}-service" \
        --region $REGION
    
    # Delete service
    aws ecs delete-service \
        --cluster $CLUSTER_NAME \
        --service "${app_name}-service" \
        --region $REGION
    
    echo "✅ Service deleted"
}

# Main command handler
case ${1:-help} in
    create-task)
        create_task_definition "$2" "$3" "$4" "$5"
        ;;
    create-service)
        create_service "$2" "$3"
        ;;
    create-with-alb)
        create_service_with_alb "$2" "$3"
        ;;
    update)
        update_service "$2" "$3"
        ;;
    scale)
        scale_service "$2" "$3"
        ;;
    status)
        check_service_status "$2"
        ;;
    logs)
        view_logs "$2" "$3"
        ;;
    stop)
        stop_service "$2"
        ;;
    delete)
        delete_service "$2"
        ;;
    *)
        echo "ECS Service Manager"
        echo "Usage: $0 {create-task|create-service|create-with-alb|update|scale|status|logs|stop|delete} [options]"
        echo ""
        echo "Commands:"
        echo "  create-task APP_NAME IMAGE_URI [CPU] [MEMORY]"
        echo "  create-service APP_NAME [DESIRED_COUNT]"
        echo "  create-with-alb APP_NAME [DESIRED_COUNT]"
        echo "  update APP_NAME NEW_IMAGE_URI"
        echo "  scale APP_NAME DESIRED_COUNT"
        echo "  status APP_NAME"
        echo "  logs APP_NAME [LINES]"
        echo "  stop APP_NAME"
        echo "  delete APP_NAME"
        ;;
esac
EOF

chmod +x ecs_service_manager.sh

echo "ECS task definition and service management completed!"
echo "Run ./ecs_service_manager.sh to manage ECS services"
```

### Task 6: Container Optimization
**Optimize Docker images and containers for performance and security.**

**Detailed Solution:**

#### 1. Multi-Stage Docker Builds
Create optimized Docker images with smaller size and better security:
```dockerfile
# Multi-stage build for Node.js application
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001
COPY --from=builder --chown=nextjs:nodejs /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
USER nextjs
CMD ["nginx", "-g", "daemon off;"]
```

#### 2. Dockerfile Optimization Best Practices
```dockerfile
# Use specific version tags, not 'latest'
FROM node:18.17.0-alpine3.18

# Combine RUN commands to reduce layers
RUN apk add --no-cache \
    curl \
    wget \
    && rm -rf /var/cache/apk/*

# Use .dockerignore to exclude unnecessary files
```

Create `.dockerignore`:
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.vscode
```

#### 3. Security Hardening
```dockerfile
# Don't run as root
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# Set proper permissions
COPY --chown=appuser:appgroup . /app
USER appuser

# Use read-only filesystem
docker run --read-only --tmpfs /tmp myapp:latest
```

#### 4. Image Scanning and Vulnerability Management
```bash
#!/bin/bash
# container_security_scan.sh

IMAGE_NAME=$1
if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

echo "🔍 Scanning image: $IMAGE_NAME"

# Scan with Trivy
echo "Running Trivy scan..."
trivy image --severity HIGH,CRITICAL --format table $IMAGE_NAME

# Scan with Docker Scout (if available)
if command -v docker &> /dev/null; then
    echo "Running Docker Scout scan..."
    docker scout cves $IMAGE_NAME
fi

# Analyze image layers
echo "Image layer analysis:"
docker history $IMAGE_NAME --format "table {{.CreatedBy}}\t{{.Size}}"

# Check for best practices
echo "Dockerfile best practices check:"
if docker run --rm -i hadolint/hadolint < Dockerfile; then
    echo "✅ Dockerfile follows best practices"
else
    echo "❌ Dockerfile has issues"
fi
```

### Task 7: Container Monitoring
**Set up comprehensive monitoring for ECS containers using CloudWatch and custom metrics.**

**Detailed Solution:**

#### 1. CloudWatch Logging Configuration
```json
{
  "family": "my-app-task",
  "containerDefinitions": [
    {
      "name": "my-app",
      "image": "my-app:latest",
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs",
          "awslogs-datetime-format": "%Y-%m-%d %H:%M:%S"
        }
      }
    }
  ]
}
```

#### 2. Custom Metrics with CloudWatch Agent
```bash
# Install CloudWatch agent sidecar
cat > cloudwatch-agent-config.json << 'EOF'
{
  "metrics": {
    "namespace": "ECS/ContainerMetrics",
    "metrics_collected": {
      "cpu": {
        "measurement": ["cpu_usage_idle", "cpu_usage_iowait"],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["used_percent"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      }
    }
  }
}
EOF
```

#### 3. Application-Level Monitoring
```javascript
// Node.js application with custom metrics
const express = require('express');
const prometheus = require('prom-client');
const AWS = require('aws-sdk');

const app = express();
const cloudwatch = new AWS.CloudWatch();

// Create metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Middleware to track metrics
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    httpRequestDuration
      .labels(req.method, req.route?.path || 'unknown', res.statusCode)
      .observe(duration);
      
    httpRequestTotal
      .labels(req.method, req.route?.path || 'unknown', res.statusCode)
      .inc();
      
    // Send custom metric to CloudWatch
    const params = {
      Namespace: 'MyApp/API',
      MetricData: [{
        MetricName: 'ResponseTime',
        Value: duration * 1000,
        Unit: 'Milliseconds',
        Dimensions: [{
          Name: 'Environment',
          Value: process.env.ENVIRONMENT || 'production'
        }]
      }]
    };
    
    cloudwatch.putMetricData(params).promise().catch(console.error);
  });
  
  next();
});

// Metrics endpoint for Prometheus
app.get('/metrics', (req, res) => {
  res.set('Content-Type', prometheus.register.contentType);
  res.end(prometheus.register.metrics());
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### 4. CloudWatch Dashboard Creation
```bash
#!/bin/bash
# create_ecs_dashboard.sh

CLUSTER_NAME=$1
SERVICE_NAME=$2
DASHBOARD_NAME="ECS-${CLUSTER_NAME}-${SERVICE_NAME}"

cat > dashboard_body.json << EOF
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/ECS", "CPUUtilization", "ServiceName", "$SERVICE_NAME", "ClusterName", "$CLUSTER_NAME"],
          [".", "MemoryUtilization", ".", ".", ".", "."]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-west-2",
        "title": "ECS Service Metrics"
      }
    },
    {
      "type": "log",
      "properties": {
        "query": "SOURCE '/ecs/my-app'\n| fields @timestamp, @message\n| sort @timestamp desc\n| limit 100",
        "region": "us-west-2",
        "title": "Recent Logs"
      }
    }
  ]
}
EOF

aws cloudwatch put-dashboard \
  --dashboard-name "$DASHBOARD_NAME" \
  --dashboard-body file://dashboard_body.json

echo "Dashboard created: $DASHBOARD_NAME"
```

### Task 8: CI/CD Integration
**Integrate ECS deployments with comprehensive CI/CD pipelines.**

**Detailed Solution:**

#### 1. CodePipeline with CodeBuild
```yaml
# buildspec.yml for CodeBuild
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"my-app","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
      - echo Running security scan...
      - trivy image --exit-code 1 --severity CRITICAL $REPOSITORY_URI:$IMAGE_TAG

artifacts:
  files:
    - imagedefinitions.json
    - taskdef.json
```

#### 2. Jenkins Pipeline for ECS
```groovy
pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = 'us-west-2'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        IMAGE_REPO_NAME = 'my-app'
        ECS_CLUSTER = 'my-cluster'
        ECS_SERVICE = 'my-service'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.BUILD_TAG = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm test'
                        publishTestResults testResultsPattern: 'test-results.xml'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'npm audit --audit-level high'
                    }
                }
            }
        }
        
        stage('Build & Push Image') {
            steps {
                script {
                    // Login to ECR
                    sh """
                        aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | \\
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                    
                    // Build image
                    def image = docker.build("${ECR_REGISTRY}/${IMAGE_REPO_NAME}:${BUILD_TAG}")
                    
                    // Security scan
                    sh "trivy image --exit-code 1 --severity CRITICAL ${ECR_REGISTRY}/${IMAGE_REPO_NAME}:${BUILD_TAG}"
                    
                    // Push image
                    image.push("${BUILD_TAG}")
                    image.push("latest")
                }
            }
        }
        
        stage('Deploy to ECS') {
            steps {
                script {
                    // Update task definition
                    sh """
                        aws ecs describe-task-definition --task-definition ${ECS_SERVICE} \\
                            --query taskDefinition > taskdef.json
                        
                        # Update image URI in task definition
                        cat taskdef.json | jq '.containerDefinitions[0].image="${ECR_REGISTRY}/${IMAGE_REPO_NAME}:${BUILD_TAG}"' > taskdef-updated.json
                        
                        # Register new task definition
                        aws ecs register-task-definition --cli-input-json file://taskdef-updated.json
                        
                        # Update service
                        aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} \\
                            --task-definition ${ECS_SERVICE}
                        
                        # Wait for deployment
                        aws ecs wait services-stable --cluster ${ECS_CLUSTER} --services ${ECS_SERVICE}
                    """
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    sleep(time: 30, unit: 'SECONDS')
                    
                    def healthCheck = sh(
                        script: """
                            aws elbv2 describe-target-health --target-group-arn ${TARGET_GROUP_ARN} \\
                                --query 'TargetHealthDescriptions[?TargetHealth.State==`healthy`]' \\
                                --output text | wc -l
                        """,
                        returnStdout: true
                    ).trim()
                    
                    if (healthCheck.toInteger() < 1) {
                        error("Health check failed - no healthy targets")
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "✅ Deployment successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: "❌ Deployment failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
```

#### 3. GitLab CI/CD for ECS
```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  AWS_DEFAULT_REGION: us-west-2
  ECR_REGISTRY: $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  IMAGE_NAME: my-app

test:
  stage: test
  image: node:18-alpine
  script:
    - npm ci
    - npm run test
    - npm run lint
  artifacts:
    reports:
      junit: test-results.xml

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - apk add --no-cache aws-cli
    - aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
    - docker build -t $ECR_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHA .
    - docker push $ECR_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHA
  only:
    - main

deploy:
  stage: deploy
  image: amazon/aws-cli:latest
  script:
    - |
      # Update ECS service
      aws ecs update-service \
        --cluster my-cluster \
        --service my-service \
        --force-new-deployment
      
      # Wait for deployment
      aws ecs wait services-stable \
        --cluster my-cluster \
        --services my-service
  only:
    - main
```

### Task 9: Auto-Scaling ECS Services
**Configure comprehensive auto-scaling for ECS services based on multiple metrics.**

**Detailed Solution:**

#### 1. Service Auto-Scaling Configuration
```bash
#!/bin/bash
# setup_ecs_autoscaling.sh

CLUSTER_NAME="my-cluster"
SERVICE_NAME="my-service"
MIN_CAPACITY=1
MAX_CAPACITY=10

# Register scalable target
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id "service/${CLUSTER_NAME}/${SERVICE_NAME}" \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity $MIN_CAPACITY \
  --max-capacity $MAX_CAPACITY

# CPU-based scaling policy
cat > cpu-scaling-policy.json << 'EOF'
{
  "TargetValue": 60.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
  },
  "ScaleOutCooldown": 300,
  "ScaleInCooldown": 300
}
EOF

aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id "service/${CLUSTER_NAME}/${SERVICE_NAME}" \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name "${SERVICE_NAME}-cpu-scaling" \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://cpu-scaling-policy.json

# Memory-based scaling policy
cat > memory-scaling-policy.json << 'EOF'
{
  "TargetValue": 70.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ECSServiceAverageMemoryUtilization"
  },
  "ScaleOutCooldown": 300,
  "ScaleInCooldown": 600
}
EOF

aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id "service/${CLUSTER_NAME}/${SERVICE_NAME}" \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name "${SERVICE_NAME}-memory-scaling" \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://memory-scaling-policy.json

# Custom metric scaling (ALB request count)
cat > alb-request-scaling-policy.json << 'EOF'
{
  "TargetValue": 1000.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ALBRequestCountPerTarget",
    "ResourceLabel": "app/my-alb/1234567890123456/targetgroup/my-targets/1234567890123456"
  },
  "ScaleOutCooldown": 300,
  "ScaleInCooldown": 300
}
EOF

aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id "service/${CLUSTER_NAME}/${SERVICE_NAME}" \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name "${SERVICE_NAME}-request-scaling" \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://alb-request-scaling-policy.json

echo "Auto-scaling configured for service: ${SERVICE_NAME}"
```

#### 2. Cluster Auto-Scaling (EC2)
```bash
#!/bin/bash
# setup_cluster_autoscaling.sh

CLUSTER_NAME="my-cluster"
ASG_NAME="my-ecs-asg"

# Create capacity provider
aws ecs create-capacity-provider \
  --name "${CLUSTER_NAME}-capacity-provider" \
  --auto-scaling-group-provider \
  autoScalingGroupArn="arn:aws:autoscaling:us-west-2:123456789012:autoScalingGroup:uuid:autoScalingGroupName/${ASG_NAME}",\
managedScaling="{\"status\":\"ENABLED\",\"targetCapacity\":80,\"minimumScalingStepSize\":1,\"maximumScalingStepSize\":10}",\
managedTerminationProtection="ENABLED"

# Associate capacity provider with cluster
aws ecs put-cluster-capacity-providers \
  --cluster $CLUSTER_NAME \
  --capacity-providers "${CLUSTER_NAME}-capacity-provider" \
  --default-capacity-provider-strategy \
  capacityProvider="${CLUSTER_NAME}-capacity-provider",weight=1
```

#### 3. Scheduled Scaling
```bash
#!/bin/bash
# setup_scheduled_scaling.sh

SERVICE_RESOURCE_ID="service/my-cluster/my-service"

# Scale up during business hours (8 AM)
aws application-autoscaling put-scheduled-action \
  --service-namespace ecs \
  --resource-id $SERVICE_RESOURCE_ID \
  --scalable-dimension ecs:service:DesiredCount \
  --scheduled-action-name "scale-up-business-hours" \
  --schedule "cron(0 8 ? * MON-FRI *)" \
  --scalable-target-action MinCapacity=3,MaxCapacity=10

# Scale down after business hours (8 PM)
aws application-autoscaling put-scheduled-action \
  --service-namespace ecs \
  --resource-id $SERVICE_RESOURCE_ID \
  --scalable-dimension ecs:service:DesiredCount \
  --scheduled-action-name "scale-down-after-hours" \
  --schedule "cron(0 20 ? * MON-FRI *)" \
  --scalable-target-action MinCapacity=1,MaxCapacity=3

# Weekend scaling
aws application-autoscaling put-scheduled-action \
  --service-namespace ecs \
  --resource-id $SERVICE_RESOURCE_ID \
  --scalable-dimension ecs:service:DesiredCount \
  --scheduled-action-name "scale-down-weekend" \
  --schedule "cron(0 0 ? * SAT *)" \
  --scalable-target-action MinCapacity=1,MaxCapacity=2
```

### Task 10: Service Discovery
**Enable comprehensive service discovery for ECS services using AWS Cloud Map.**

**Detailed Solution:**

#### 1. Create Cloud Map Namespace
```bash
#!/bin/bash
# setup_service_discovery.sh

VPC_ID="vpc-12345678"
NAMESPACE_NAME="myapp.local"

# Create private DNS namespace
NAMESPACE_ID=$(aws servicediscovery create-private-dns-namespace \
  --name $NAMESPACE_NAME \
  --vpc $VPC_ID \
  --description "Service discovery for my application" \
  --query 'OperationId' --output text)

# Wait for operation to complete
aws servicediscovery get-operation --operation-id $NAMESPACE_ID

# Get the namespace details
aws servicediscovery list-namespaces \
  --filters Name=TYPE,Values=DNS_PRIVATE \
  --query 'Namespaces[?Name==`'$NAMESPACE_NAME'`]'
```

#### 2. ECS Task Definition with Service Discovery
```json
{
  "family": "web-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "web-app",
      "image": "my-web-app:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "API_ENDPOINT",
          "value": "http://api-service.myapp.local:8080"
        },
        {
          "name": "DB_ENDPOINT", 
          "value": "database.myapp.local"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/web-service",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### 3. ECS Service with Service Registration
```bash
#!/bin/bash
# create_service_with_discovery.sh

CLUSTER_NAME="my-cluster"
SERVICE_NAME="web-service"
NAMESPACE_ID="ns-12345678"

# Create service in Cloud Map
SERVICE_REGISTRY_ID=$(aws servicediscovery create-service \
  --name $SERVICE_NAME \
  --namespace-id $NAMESPACE_ID \
  --dns-config NamespaceId=$NAMESPACE_ID,DnsRecords='[{Type=A,TTL=60}]' \
  --health-check-custom-config FailureThreshold=1 \
  --query 'Service.Id' --output text)

# Create ECS service with service discovery
cat > service-definition.json << EOF
{
  "serviceName": "$SERVICE_NAME",
  "cluster": "$CLUSTER_NAME",
  "taskDefinition": "$SERVICE_NAME",
  "desiredCount": 2,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-12345678", "subnet-87654321"],
      "securityGroups": ["sg-12345678"],
      "assignPublicIp": "ENABLED"
    }
  },
  "serviceRegistries": [
    {
      "registryArn": "arn:aws:servicediscovery:us-west-2:123456789012:service/$SERVICE_REGISTRY_ID"
    }
  ],
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-targets/1234567890123456",
      "containerName": "web-app",
      "containerPort": 80
    }
  ]
}
EOF

aws ecs create-service --cli-input-json file://service-definition.json
```

#### 4. Service Discovery Client Code
```javascript
// Node.js service discovery client
const AWS = require('aws-sdk');
const dns = require('dns');

class ServiceDiscovery {
  constructor() {
    this.servicediscovery = new AWS.ServiceDiscovery();
    this.cache = new Map();
    this.cacheTimeout = 60000; // 1 minute
  }

  async discoverService(serviceName, namespace = 'myapp.local') {
    const cacheKey = `${serviceName}.${namespace}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.endpoints;
    }

    try {
      // Method 1: DNS resolution
      const fqdn = `${serviceName}.${namespace}`;
      const addresses = await this.resolveDNS(fqdn);
      
      if (addresses.length > 0) {
        const endpoints = addresses.map(addr => ({
          host: addr,
          port: 80, // Default port
          healthy: true
        }));
        
        this.cache.set(cacheKey, {
          endpoints,
          timestamp: Date.now()
        });
        
        return endpoints;
      }

      // Method 2: AWS Service Discovery API
      return await this.discoverViaAPI(serviceName, namespace);
      
    } catch (error) {
      console.error('Service discovery failed:', error);
      return [];
    }
  }

  async resolveDNS(fqdn) {
    return new Promise((resolve, reject) => {
      dns.resolve4(fqdn, (err, addresses) => {
        if (err) reject(err);
        else resolve(addresses);
      });
    });
  }

  async discoverViaAPI(serviceName, namespace) {
    const params = {
      NamespaceName: namespace,
      ServiceName: serviceName
    };

    const result = await this.servicediscovery.discoverInstances(params).promise();
    
    const endpoints = result.Instances.map(instance => ({
      host: instance.Attributes.AWS_INSTANCE_IPV4,
      port: instance.Attributes.AWS_INSTANCE_PORT || 80,
      healthy: true
    }));

    return endpoints;
  }

  async makeRequest(serviceName, path, options = {}) {
    const endpoints = await this.discoverService(serviceName);
    
    if (endpoints.length === 0) {
      throw new Error(`No healthy endpoints found for service: ${serviceName}`);
    }

    // Simple round-robin load balancing
    const endpoint = endpoints[Math.floor(Math.random() * endpoints.length)];
    const url = `http://${endpoint.host}:${endpoint.port}${path}`;

    // Make HTTP request (using fetch or axios)
    const response = await fetch(url, options);
    return response;
  }
}

// Usage example
const serviceDiscovery = new ServiceDiscovery();

async function callAPIService() {
  try {
    const response = await serviceDiscovery.makeRequest('api-service', '/health');
    const data = await response.json();
    console.log('API response:', data);
  } catch (error) {
    console.error('API call failed:', error);
  }
}
```

## Intermediate Level Solutions (Tasks 11-15)

### Task 11: Multi-Region ECS Deployment
**Deploy ECS services across multiple AWS regions for high availability and disaster recovery.**

**Detailed Solution:**

#### 1. Multi-Region Infrastructure with Terraform
```hcl
# variables.tf
variable "regions" {
  description = "List of AWS regions for deployment"
  type        = list(string)
  default     = ["us-west-2", "us-east-1"]
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "production"
}

# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Provider configuration for multiple regions
provider "aws" {
  alias  = "us_west_2"
  region = "us-west-2"
}

provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"
}

# ECR repositories (global)
resource "aws_ecr_repository" "app" {
  provider = aws.us_west_2
  name     = "my-app"

  image_scanning_configuration {
    scan_on_push = true
  }

  lifecycle_policy {
    policy = jsonencode({
      rules = [{
        rulePriority = 1
        description  = "Keep last 10 images"
        selection = {
          tagStatus     = "tagged"
          countType     = "imageCountMoreThan"
          countNumber   = 10
        }
        action = {
          type = "expire"
        }
      }]
    })
  }
}

# Regional ECS clusters
module "ecs_us_west_2" {
  source = "./modules/ecs-cluster"
  providers = {
    aws = aws.us_west_2
  }
  region      = "us-west-2"
  environment = var.environment
  is_primary  = true
}

module "ecs_us_east_1" {
  source = "./modules/ecs-cluster"
  providers = {
    aws = aws.us_east_1
  }
  region      = "us-east-1"
  environment = var.environment
  is_primary  = false
}
```

#### 2. Cross-Region DNS Failover with Route 53
```hcl
# route53.tf
resource "aws_route53_zone" "main" {
  name = "myapp.com"

  tags = {
    Environment = var.environment
  }
}

# Health checks for primary region
resource "aws_route53_health_check" "primary" {
  fqdn                           = module.ecs_us_west_2.alb_dns_name
  port                           = 80
  type                           = "HTTP"
  resource_path                  = "/health"
  failure_threshold              = 3
  request_interval               = 30
  cloudwatch_alarm_region        = "us-west-2"
  cloudwatch_alarm_name          = "primary-health-check"
  insufficient_data_health_status = "Failure"

  tags = {
    Name = "Primary Region Health Check"
  }
}

# Primary region record
resource "aws_route53_record" "primary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.myapp.com"
  type    = "A"

  set_identifier = "primary"
  failover_routing_policy {
    type = "PRIMARY"
  }

  alias {
    name                   = module.ecs_us_west_2.alb_dns_name
    zone_id                = module.ecs_us_west_2.alb_zone_id
    evaluate_target_health = true
  }

  health_check_id = aws_route53_health_check.primary.id
}

# Secondary region record
resource "aws_route53_record" "secondary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.myapp.com"
  type    = "A"

  set_identifier = "secondary"
  failover_routing_policy {
    type = "SECONDARY"
  }

  alias {
    name                   = module.ecs_us_east_1.alb_dns_name
    zone_id                = module.ecs_us_east_1.alb_zone_id
    evaluate_target_health = true
  }
}
```

### Task 12: ECS with Service Mesh (AWS App Mesh)
**Integrate AWS App Mesh with ECS for advanced microservices communication and traffic management.**

**Detailed Solution:**

#### 1. App Mesh Setup
```bash
#!/bin/bash
# setup_app_mesh.sh

MESH_NAME="my-app-mesh"
NAMESPACE="myapp.local"

# Create App Mesh
aws appmesh create-mesh --mesh-name $MESH_NAME

# Create virtual services
for service in web-service api-service auth-service; do
  cat > virtual-service-${service}.json << EOF
{
  "meshName": "$MESH_NAME",
  "virtualServiceName": "${service}.${NAMESPACE}",
  "spec": {
    "provider": {
      "virtualNode": {
        "virtualNodeName": "$service"
      }
    }
  }
}
EOF

  aws appmesh create-virtual-service --cli-input-json file://virtual-service-${service}.json
done
```

#### 2. ECS Task Definition with Envoy Sidecar
```json
{
  "family": "web-service-with-mesh",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "proxyConfiguration": {
    "type": "APPMESH",
    "containerName": "envoy",
    "properties": [
      {
        "name": "IgnoredUID",
        "value": "1337"
      },
      {
        "name": "ProxyIngressPort",
        "value": "15000"
      },
      {
        "name": "ProxyEgressPort",
        "value": "15001"
      },
      {
        "name": "AppPorts",
        "value": "80"
      },
      {
        "name": "EgressIgnoredIPs",
        "value": "169.254.170.2,169.254.169.254"
      }
    ]
  },
  "containerDefinitions": [
    {
      "name": "web-app",
      "image": "my-web-app:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "API_ENDPOINT",
          "value": "http://api-service.myapp.local:8080"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/web-service",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "essential": true,
      "dependsOn": [
        {
          "containerName": "envoy",
          "condition": "HEALTHY"
        }
      ]
    },
    {
      "name": "envoy",
      "image": "840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.26.4.0-prod",
      "essential": true,
      "environment": [
        {
          "name": "APPMESH_VIRTUAL_NODE_NAME",
          "value": "mesh/my-app-mesh/virtualNode/web-service"
        }
      ],
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
        ],
        "startPeriod": 10,
        "interval": 5,
        "timeout": 2,
        "retries": 3
      },
      "user": "1337"
    }
  ]
}
```

### Task 13: Advanced ECS Monitoring and Observability
**Implement comprehensive monitoring, distributed tracing, and observability for ECS workloads.**

**Detailed Solution:**

#### 1. CloudWatch Container Insights with Custom Dashboard
```bash
#!/bin/bash
# setup_monitoring.sh

CLUSTER_NAME="my-cluster"
REGION="us-west-2"

# Enable Container Insights
aws ecs put-account-setting \
  --name "containerInsights" \
  --value "enabled" \
  --region $REGION

# Create custom dashboard
cat > dashboard-config.json << 'EOF'
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["ECS/ContainerInsights", "CpuUtilized", "ClusterName", "my-cluster"],
          [".", "MemoryUtilized", ".", "."],
          [".", "NetworkRxBytes", ".", "."],
          [".", "NetworkTxBytes", ".", "."]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-west-2",
        "title": "ECS Cluster Metrics"
      }
    }
  ]
}
EOF

aws cloudwatch put-dashboard \
  --dashboard-name "ECS-Advanced-Monitoring" \
  --dashboard-body file://dashboard-config.json
```

#### 2. Application with X-Ray Tracing
```javascript
// Node.js app with comprehensive monitoring
const AWSXRay = require('aws-xray-sdk-core');
const AWS = AWSXRay.captureAWS(require('aws-sdk'));
const express = require('express');
const prometheus = require('prom-client');

// Initialize X-Ray
AWSXRay.config([AWSXRay.plugins.ECSPlugin]);

const app = express();
app.use(AWSXRay.express.openSegment('my-app'));

// Custom metrics
const httpDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

// API with tracing
app.get('/api/users', async (req, res) => {
  const subsegment = AWSXRay.getSegment().addNewSubsegment('get_users');
  
  try {
    const dynamodb = new AWS.DynamoDB.DocumentClient();
    const result = await dynamodb.scan({
      TableName: 'Users'
    }).promise();
    
    subsegment.addMetadata('database', {
      operation: 'scan',
      count: result.Items.length
    });
    
    res.json(result.Items);
  } catch (error) {
    subsegment.addError(error);
    res.status(500).json({ error: 'Internal server error' });
  } finally {
    subsegment.close();
  }
});

app.use(AWSXRay.express.closeSegment());
```

### Task 14: Blue-Green Deployment with ECS and CodeDeploy
**Implement automated blue-green deployments for ECS services using AWS CodeDeploy.**

**Detailed Solution:**

#### 1. CodeDeploy Setup
```bash
#!/bin/bash
# setup_blue_green.sh

APP_NAME="my-ecs-app"
DEPLOYMENT_GROUP_NAME="my-deployment-group"

# Create CodeDeploy application
aws deploy create-application \
  --application-name $APP_NAME \
  --compute-platform ECS

# Create deployment group configuration
cat > deployment-group.json << 'EOF'
{
  "applicationName": "my-ecs-app",
  "deploymentGroupName": "my-deployment-group",
  "serviceRoleArn": "arn:aws:iam::123456789012:role/CodeDeployServiceRole",
  "deploymentConfigName": "CodeDeployDefault.ECSAllAtOnceBlueGreen",
  "blueGreenDeploymentConfiguration": {
    "terminateBlueInstancesOnDeploymentSuccess": {
      "action": "TERMINATE",
      "terminationWaitTimeInMinutes": 5
    },
    "deploymentReadyOption": {
      "actionOnTimeout": "CONTINUE_DEPLOYMENT"
    }
  },
  "loadBalancerInfo": {
    "targetGroupInfoList": [
      {
        "name": "my-targets-blue"
      },
      {
        "name": "my-targets-green"
      }
    ]
  },
  "ecsServices": [
    {
      "serviceName": "my-service",
      "clusterName": "my-cluster"
    }
  ]
}
EOF

aws deploy create-deployment-group --cli-input-json file://deployment-group.json
```

#### 2. AppSpec for Blue-Green Deployment
```yaml
# appspec.yml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: "my-app"
          ContainerPort: 80

Hooks:
  - BeforeInstall: "scripts/stop_application.sh"
  - AfterInstall: "scripts/start_application.sh"
  - BeforeAllowTraffic: "scripts/health_check.sh"
  - AfterAllowTraffic: "scripts/integration_test.sh"
```

#### 3. Deployment Validation Scripts
```bash
#!/bin/bash
# scripts/health_check.sh
set -e

echo "Running health checks..."

MAX_ATTEMPTS=30
ATTEMPT=1

while [ $ATTEMPT -le $MAX_ATTEMPTS ]; do
  HEALTHY_COUNT=$(aws elbv2 describe-target-health \
    --target-group-arn $TARGET_GROUP_ARN \
    --query 'TargetHealthDescriptions[?TargetHealth.State==`healthy`]' \
    --output text | wc -l)
  
  if [ $HEALTHY_COUNT -gt 0 ]; then
    echo "✅ Health check passed"
    exit 0
  fi
  
  echo "⏳ Waiting for healthy targets... ($ATTEMPT/$MAX_ATTEMPTS)"
  sleep 10
  ATTEMPT=$((ATTEMPT + 1))
done

echo "❌ Health check failed"
exit 1
```

### Task 15: ECS Secrets Management and Security
**Implement comprehensive secrets management and security best practices for ECS workloads.**

**Detailed Solution:**

#### 1. Secrets Manager Setup
```bash
#!/bin/bash
# setup_secrets.sh

# Create database credentials
aws secretsmanager create-secret \
  --name "myapp/database/credentials" \
  --description "Database credentials" \
  --secret-string '{
    "username": "admin",
    "password": "'$(openssl rand -base64 32)'",
    "host": "db.example.com",
    "port": "5432"
  }'

# Create API keys
aws secretsmanager create-secret \
  --name "myapp/api/keys" \
  --description "External API keys" \
  --secret-string '{
    "stripe_key": "sk_test_...",
    "jwt_secret": "'$(openssl rand -base64 64)'"
  }'
```

#### 2. Secure ECS Task Definition
```json
{
  "family": "secure-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "secure-app",
      "image": "my-secure-app:latest",
      "secrets": [
        {
          "name": "DB_USERNAME",
          "valueFrom": "arn:aws:secretsmanager:us-west-2:123456789012:secret:myapp/database/credentials:username::"
        },
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-west-2:123456789012:secret:myapp/database/credentials:password::"
        },
        {
          "name": "JWT_SECRET",
          "valueFrom": "arn:aws:secretsmanager:us-west-2:123456789012:secret:myapp/api/keys:jwt_secret::"
        }
      ],
      "readonlyRootFilesystem": true,
      "user": "1001:1001",
      "linuxParameters": {
        "capabilities": {
          "drop": ["ALL"]
        }
      },
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/secure-app",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### 3. Application with Secure Secrets Handling
```javascript
// Secure Node.js application
const AWS = require('aws-sdk');
const express = require('express');

class SecureSecretsManager {
  constructor() {
    this.secretsManager = new AWS.SecretsManager();
    this.cache = new Map();
    this.cacheTimeout = 300000; // 5 minutes
  }

  async getSecret(secretId) {
    const cached = this.cache.get(secretId);
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.value;
    }

    try {
      const result = await this.secretsManager.getSecretValue({
        SecretId: secretId
      }).promise();

      const secretValue = JSON.parse(result.SecretString);
      
      this.cache.set(secretId, {
        value: secretValue,
        timestamp: Date.now()
      });

      return secretValue;
    } catch (error) {
      console.error('Failed to retrieve secret:', error);
      throw error;
    }
  }
}

const secretsManager = new SecureSecretsManager();
const app = express();

// Secure database connection
async function connectToDatabase() {
  const credentials = await secretsManager.getSecret('myapp/database/credentials');
  
  return {
    host: credentials.host,
    username: credentials.username,
    password: credentials.password,
    port: credentials.port
  };
}

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

// Graceful shutdown with secret cleanup
process.on('SIGTERM', () => {
  secretsManager.cache.clear();
  process.exit(0);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Secure app running on port ${PORT}`);
});
```

---