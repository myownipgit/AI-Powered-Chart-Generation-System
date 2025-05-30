# 🚀 Deployment Guide

Complete guide for deploying the AI-Powered Chart Generation System in production environments. This guide covers everything from local development to enterprise-scale deployments.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Local Development](#local-development)
- [Docker Deployment](#docker-deployment)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Cloud Platform Deployments](#cloud-platform-deployments)
- [Configuration Management](#configuration-management)
- [Monitoring & Observability](#monitoring--observability)
- [Security Configuration](#security-configuration)
- [Performance Optimization](#performance-optimization)
- [Backup & Recovery](#backup--recovery)
- [Troubleshooting](#troubleshooting)

## ⚙️ Prerequisites

### System Requirements

#### Minimum Requirements (Development)
- **CPU**: 2 cores
- **RAM**: 4 GB
- **Storage**: 20 GB SSD
- **Network**: 1 Mbps

#### Recommended (Production)
- **CPU**: 8+ cores
- **RAM**: 16+ GB
- **Storage**: 100+ GB SSD (with backup)
- **Network**: 1 Gbps

### Software Dependencies

#### Required
- **Docker**: 24.0+
- **Docker Compose**: 2.20+
- **Kubernetes**: 1.28+ (for K8s deployment)
- **kubectl**: 1.28+
- **Helm**: 3.12+ (recommended)

#### Optional
- **Terraform**: 1.5+ (for infrastructure as code)
- **Ansible**: 2.15+ (for configuration management)
- **Git**: 2.40+

### External Services

#### Required Services
- **PostgreSQL**: 15+ (managed or self-hosted)
- **Redis**: 7+ (managed or self-hosted)
- **OpenAI API**: Account with API key

#### Optional Services
- **AWS S3**: For file storage
- **CloudFlare**: For CDN and DDoS protection
- **Datadog/New Relic**: For advanced monitoring

## 🔧 Environment Setup

### 1. Clone Repository

```bash
git clone https://github.com/myownipgit/AI-Powered-Chart-Generation-System.git
cd AI-Powered-Chart-Generation-System
```

### 2. Environment Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit configuration
nano .env
```

### 3. Environment Variables

Create a `.env` file with the following configuration:

```bash
# =====================================================
# ENVIRONMENT CONFIGURATION
# =====================================================

# Application
ENVIRONMENT=production
DEBUG=false
API_VERSION=v1
SECRET_KEY=your-super-secret-key-here

# Database
DATABASE_URL=postgresql://chart_user:secure_password@postgres:5432/chart_system
DATABASE_POOL_SIZE=20
DATABASE_MAX_OVERFLOW=30

# Redis
REDIS_URL=redis://redis:6379/0
REDIS_CACHE_TTL=3600

# AI/ML Services
OPENAI_API_KEY=sk-your-openai-api-key-here
OPENAI_MODEL=gpt-4
SPACY_MODEL=en_core_web_sm

# Authentication
JWT_SECRET_KEY=your-jwt-secret-key
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=60
JWT_REFRESH_TOKEN_EXPIRE_DAYS=30

# File Storage
STORAGE_BACKEND=s3  # or 'local' for development
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_S3_BUCKET_NAME=chartgen-storage
AWS_REGION=us-east-1

# Email (for notifications)
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=noreply@chartgen.ai
SMTP_PASSWORD=your-app-password
SMTP_FROM_EMAIL=noreply@chartgen.ai

# Monitoring
PROMETHEUS_ENABLED=true
JAEGER_ENABLED=true
LOG_LEVEL=INFO

# Rate Limiting
RATE_LIMIT_ENABLED=true
FREE_TIER_LIMIT=100
PRO_TIER_LIMIT=1000
ENTERPRISE_TIER_LIMIT=10000

# Security
ALLOWED_HOSTS=chartgen.ai,www.chartgen.ai,api.chartgen.ai
CORS_ORIGINS=https://chartgen.ai,https://www.chartgen.ai

# Performance
WORKER_PROCESSES=4
WORKER_CONNECTIONS=1000
KEEPALIVE_TIMEOUT=65
```

## 💻 Local Development

### Quick Start with Docker Compose

```bash
# Start all services
docker-compose -f docker-compose.dev.yml up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f api

# Stop services
docker-compose down
```

### Manual Setup (for development)

#### Backend Setup
```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run database migrations
python manage.py migrate

# Start development server
python main.py
```

#### Frontend Setup
```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

### Development Services

| Service | URL | Description |
|---------|-----|-------------|
| Frontend | http://localhost:3000 | React development server |
| API | http://localhost:5000 | FastAPI backend |
| API Docs | http://localhost:5000/docs | Swagger UI |
| Database | localhost:5432 | PostgreSQL |
| Redis | localhost:6379 | Redis cache |
| Mailhog | http://localhost:8025 | Email testing |

## 🐳 Docker Deployment

### Production Docker Compose

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  # Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
      - static_files:/var/www/static
    depends_on:
      - api
      - frontend
    restart: unless-stopped

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=https://api.chartgen.ai
    restart: unless-stopped

  # API Backend
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      - ENVIRONMENT=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - static_files:/app/static
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 1G
          cpus: '1.0'

  # WebSocket Server
  websocket:
    build:
      context: ./websocket
      dockerfile: Dockerfile
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    ports:
      - "8765:8765"
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  # Database
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=chart_system
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    restart: unless-stopped

  # Cache
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    restart: unless-stopped

  # Background Workers
  worker:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: celery -A app.celery worker --loglevel=info
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    deploy:
      replicas: 2

  # Task Scheduler
  scheduler:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: celery -A app.celery beat --loglevel=info
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  static_files:
```

### Deployment Commands

```bash
# Build and deploy
docker-compose -f docker-compose.prod.yml up -d --build

# Scale services
docker-compose -f docker-compose.prod.yml up -d --scale api=5 --scale worker=3

# Update specific service
docker-compose -f docker-compose.prod.yml up -d --no-deps api

# View logs
docker-compose -f docker-compose.prod.yml logs -f --tail=100 api

# Health check
docker-compose -f docker-compose.prod.yml exec api python health_check.py
```

## ☸️ Kubernetes Deployment

### Helm Chart Deployment (Recommended)

```bash
# Add Helm repository
helm repo add chartgen https://charts.chartgen.ai
helm repo update

# Install with custom values
helm install chartgen chartgen/ai-chart-generator \
  --namespace chartgen \
  --create-namespace \
  --values values.prod.yaml
```

### Custom Values (values.prod.yaml)

```yaml
# Production values for Helm chart
replicaCount:
  api: 3
  websocket: 2
  worker: 2

image:
  repository: ghcr.io/myownipgit/ai-chart-generator
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "1000"
  hosts:
    - host: api.chartgen.ai
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: chartgen-tls
      hosts:
        - api.chartgen.ai

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

resources:
  limits:
    cpu: 1000m
    memory: 2Gi
  requests:
    cpu: 500m
    memory: 1Gi

postgresql:
  enabled: true
  auth:
    postgresPassword: "secure-password"
    database: "chart_system"
  primary:
    persistence:
      enabled: true
      size: 100Gi

redis:
  enabled: true
  auth:
    enabled: true
    password: "redis-password"
  master:
    persistence:
      enabled: true
      size: 10Gi

monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
```

### Manual Kubernetes Deployment

```bash
# Create namespace
kubectl create namespace chartgen

# Apply configurations
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl apply -f k8s/hpa.yaml

# Check deployment status
kubectl get pods -n chartgen
kubectl rollout status deployment/chartgen-api -n chartgen
```

### Kubernetes Manifest Example

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chartgen-api
  namespace: chartgen
spec:
  replicas: 3
  selector:
    matchLabels:
      app: chartgen-api
  template:
    metadata:
      labels:
        app: chartgen-api
    spec:
      containers:
      - name: api
        image: ghcr.io/myownipgit/ai-chart-generator-api:latest
        ports:
        - containerPort: 5000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: chartgen-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: chartgen-secrets
              key: redis-url
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 5
```

## ☁️ Cloud Platform Deployments

### AWS Deployment

#### Using EKS (Elastic Kubernetes Service)

```bash
# Create EKS cluster
eksctl create cluster \
  --name chartgen-prod \
  --version 1.28 \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 10 \
  --managed

# Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name chartgen-prod

# Deploy application
helm install chartgen ./helm/chartgen -f values.aws.yaml
```

#### Using ECS (Elastic Container Service)

```json
{
  "family": "chartgen-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "api",
      "image": "ghcr.io/myownipgit/ai-chart-generator-api:latest",
      "portMappings": [
        {
          "containerPort": 5000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ENVIRONMENT",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:account:secret:chartgen/database-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/chartgen-api",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### Terraform Configuration

```hcl
# terraform/aws/main.tf
provider "aws" {
  region = "us-east-1"
}

# VPC
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "chartgen-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = true
}

# EKS Cluster
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  
  cluster_name    = "chartgen-prod"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  node_groups = {
    main = {
      desired_capacity = 3
      max_capacity     = 10
      min_capacity     = 1
      
      instance_types = ["t3.medium"]
      
      k8s_labels = {
        Environment = "production"
        Application = "chartgen"
      }
    }
  }
}

# RDS Database
resource "aws_db_instance" "main" {
  identifier = "chartgen-prod"
  
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.r6g.large"
  
  allocated_storage     = 100
  max_allocated_storage = 1000
  storage_encrypted     = true
  
  db_name  = "chart_system"
  username = var.db_username
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 30
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  deletion_protection = true
  
  tags = {
    Name = "chartgen-prod"
  }
}

# ElastiCache Redis
resource "aws_elasticache_cluster" "redis" {
  cluster_id           = "chartgen-prod"
  engine               = "redis"
  node_type            = "cache.r6g.large"
  num_cache_nodes      = 1
  parameter_group_name = "default.redis7"
  port                 = 6379
  subnet_group_name    = aws_elasticache_subnet_group.main.name
  security_group_ids   = [aws_security_group.redis.id]
}
```

### Google Cloud Platform (GCP)

```bash
# Create GKE cluster
gcloud container clusters create chartgen-prod \
  --zone=us-central1-a \
  --num-nodes=3 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --machine-type=e2-standard-4

# Get credentials
gcloud container clusters get-credentials chartgen-prod --zone=us-central1-a

# Deploy application
kubectl apply -f k8s/gcp/
```

### Microsoft Azure

```bash
# Create AKS cluster
az aks create \
  --resource-group chartgen-rg \
  --name chartgen-prod \
  --node-count 3 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group chartgen-rg --name chartgen-prod

# Deploy application
helm install chartgen ./helm/chartgen -f values.azure.yaml
```

## 🔧 Configuration Management

### Environment-specific Configurations

#### Development
```yaml
# config/development.yaml
debug: true
log_level: DEBUG
database:
  pool_size: 5
  echo: true
redis:
  db: 0
rate_limiting:
  enabled: false
```

#### Staging
```yaml
# config/staging.yaml
debug: false
log_level: INFO
database:
  pool_size: 10
  echo: false
redis:
  db: 1
rate_limiting:
  enabled: true
  free_limit: 50
```

#### Production
```yaml
# config/production.yaml
debug: false
log_level: WARNING
database:
  pool_size: 20
  echo: false
redis:
  db: 0
rate_limiting:
  enabled: true
  free_limit: 100
security:
  force_https: true
  hsts_max_age: 31536000
```

### Secrets Management

#### Using Kubernetes Secrets
```bash
# Create secrets
kubectl create secret generic chartgen-secrets \
  --from-literal=database-url="postgresql://user:pass@host:5432/db" \
  --from-literal=redis-url="redis://host:6379/0" \
  --from-literal=openai-api-key="sk-..." \
  --namespace=chartgen

# Use in deployment
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chartgen-api
spec:
  template:
    spec:
      containers:
      - name: api
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: chartgen-secrets
              key: database-url
EOF
```

#### Using HashiCorp Vault
```bash
# Store secrets in Vault
vault kv put secret/chartgen/prod \
  database_url="postgresql://..." \
  redis_url="redis://..." \
  openai_api_key="sk-..."

# Use with Vault Agent or CSI driver
```

## 📊 Monitoring & Observability

### Prometheus Configuration

```yaml
# monitoring/prometheus.yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'chartgen-api'
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
            - chartgen
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### Grafana Dashboards

```json
{
  "dashboard": {
    "title": "ChartGen API Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ]
      }
    ]
  }
}
```

### Alerting Rules

```yaml
# monitoring/alert_rules.yml
groups:
- name: chartgen-alerts
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: High error rate detected
      
  - alert: DatabaseDown
    expr: up{job="postgresql"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Database is down
```

## 🔒 Security Configuration

### SSL/TLS Setup

```bash
# Using Let's Encrypt with cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Create ClusterIssuer
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@chartgen.ai
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

### Network Policies

```yaml
# security/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: chartgen-api-netpol
  namespace: chartgen
spec:
  podSelector:
    matchLabels:
      app: chartgen-api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 5000
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
  - to: []
    ports:
    - protocol: TCP
      port: 5432  # PostgreSQL
    - protocol: TCP
      port: 6379  # Redis
```

### Security Scanning

```bash
# Container vulnerability scanning
trivy image ghcr.io/myownipgit/ai-chart-generator-api:latest

# Kubernetes security scanning
kube-bench run --targets node

# Network security
nmap -sV -sC chartgen.ai
```

## ⚡ Performance Optimization

### Database Optimization

```sql
-- Create indexes for better performance
CREATE INDEX CONCURRENTLY idx_chart_configs_user_created 
ON chart_configs(user_id, created_at DESC);

CREATE INDEX CONCURRENTLY idx_chart_configs_search 
ON chart_configs USING gin(to_tsvector('english', user_prompt));

-- Analyze query performance
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM chart_configs 
WHERE user_id = 'user_123' 
ORDER BY created_at DESC 
LIMIT 20;

-- Configure PostgreSQL
ALTER SYSTEM SET shared_buffers = '1GB';
ALTER SYSTEM SET effective_cache_size = '3GB';
ALTER SYSTEM SET maintenance_work_mem = '256MB';
ALTER SYSTEM SET checkpoint_completion_target = 0.9;
```

### Redis Optimization

```bash
# Redis configuration
redis-cli CONFIG SET maxmemory 2gb
redis-cli CONFIG SET maxmemory-policy allkeys-lru
redis-cli CONFIG SET save "900 1 300 10 60 10000"

# Monitor Redis performance
redis-cli --latency-history
redis-cli INFO memory
```

### Application Performance

```python
# Connection pooling
from sqlalchemy.pool import QueuePool

engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,
    max_overflow=30,
    pool_pre_ping=True,
    pool_recycle=3600
)

# Async processing
async def process_batch_charts(requests: List[ChartRequest]):
    tasks = [generate_chart(req) for req in requests]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return results
```

## 💾 Backup & Recovery

### Database Backup

```bash
#!/bin/bash
# scripts/backup_database.sh

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="chartgen_backup_${DATE}.sql"

# Create backup
pg_dump $DATABASE_URL > "${BACKUP_DIR}/${BACKUP_FILE}"

# Compress backup
gzip "${BACKUP_DIR}/${BACKUP_FILE}"

# Upload to S3
aws s3 cp "${BACKUP_DIR}/${BACKUP_FILE}.gz" s3://chartgen-backups/database/

# Clean old backups (keep 30 days)
find $BACKUP_DIR -name "chartgen_backup_*.sql.gz" -mtime +30 -delete
```

### Kubernetes Backup

```bash
# Backup using Velero
velero backup create chartgen-backup-$(date +%Y%m%d) \
  --include-namespaces chartgen \
  --storage-location default

# Schedule regular backups
velero schedule create chartgen-daily \
  --schedule="0 2 * * *" \
  --include-namespaces chartgen
```

### Recovery Procedures

```bash
# Database recovery
psql $DATABASE_URL < backup_file.sql

# Kubernetes recovery
velero restore create --from-backup chartgen-backup-20241130

# Application recovery
kubectl rollout undo deployment/chartgen-api -n chartgen
```

## 🔧 Troubleshooting

### Common Issues

#### High Memory Usage
```bash
# Check memory usage
kubectl top pods -n chartgen

# Scale down if needed
kubectl scale deployment chartgen-api --replicas=2 -n chartgen

# Check for memory leaks
kubectl exec -it chartgen-api-pod -- python -c "
import gc
import psutil
print(f'Memory: {psutil.virtual_memory().percent}%')
print(f'Objects: {len(gc.get_objects())}')
"
```

#### Database Connection Issues
```bash
# Check database connectivity
kubectl exec -it chartgen-api-pod -- python -c "
import psycopg2
try:
    conn = psycopg2.connect('$DATABASE_URL')
    print('Database connection successful')
except Exception as e:
    print(f'Connection failed: {e}')
"

# Check connection pool
kubectl logs -f deployment/chartgen-api | grep "pool"
```

#### Performance Issues
```bash
# Check API latency
curl -w "@curl-format.txt" -o /dev/null -s https://api.chartgen.ai/health

# Check database performance
kubectl exec -it postgres-pod -- psql -c "
SELECT query, mean_exec_time, calls 
FROM pg_stat_statements 
ORDER BY mean_exec_time DESC 
LIMIT 10;
"
```

### Monitoring Commands

```bash
# Check all pods
kubectl get pods -n chartgen -o wide

# Check events
kubectl get events -n chartgen --sort-by='.lastTimestamp'

# Check resource usage
kubectl top nodes
kubectl top pods -n chartgen

# Check logs
kubectl logs -f deployment/chartgen-api -n chartgen --tail=100

# Debug networking
kubectl exec -it chartgen-api-pod -- nslookup postgres
kubectl exec -it chartgen-api-pod -- nc -zv postgres 5432
```

### Performance Tuning Checklist

- [ ] Database indexes optimized
- [ ] Connection pooling configured
- [ ] Redis caching implemented
- [ ] CDN configured for static assets
- [ ] Gzip compression enabled
- [ ] Keep-alive connections enabled
- [ ] Resource limits set appropriately
- [ ] Auto-scaling configured
- [ ] Database queries optimized
- [ ] Async processing for heavy operations

This comprehensive deployment guide provides everything needed to successfully deploy and maintain the AI-Powered Chart Generation System in production environments.
