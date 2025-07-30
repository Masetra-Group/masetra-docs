---
layout: default
title: "Deployment Guide"
description: "Comprehensive guide for deploying Masetra in various environments including cloud, on-premises, and hybrid configurations"
---

## 🚀 Deployment Options

Masetra offers multiple deployment options to meet different organizational needs and compliance requirements.

### Cloud Deployment (Recommended)

**Managed Service:** Fully managed by Masetra

- **Automatic Updates:** Zero-downtime deployments
- **99.9% Uptime SLA:** Enterprise-grade reliability
- **Global Infrastructure:** Multi-region availability
- **Professional Support:** 24/7 support team
- **Scalability:** Automatic resource scaling

### Self-Hosted Deployment

**On-Premises:** Deployed within your infrastructure

- **Complete Control:** Full administrative control
- **Data Sovereignty:** Data remains within your network
- **Custom Integration:** Deep system integration
- **Compliance:** Meet specific regulatory requirements
- **Cost Control:** Predictable licensing costs

### Hybrid Deployment

**Mixed Approach:** Combination of cloud and on-premises

- **Flexible Architecture:** Choose components for each environment
- **Gradual Migration:** Phased transition approach
- **Optimized Performance:** Critical data on-premises
- **Cost Efficiency:** Balance of control and cost

## ☁️ Cloud Deployment

### AWS Deployment

#### Prerequisites

- AWS account with appropriate permissions
- Domain name registered in Route 53 (optional)
- SSL certificate in Certificate Manager

#### Deployment Steps

1. **Launch CloudFormation Stack**

```bash
# Download CloudFormation template
wget https://downloads.masetra.com/aws/masetra-aws-template.yaml

# Deploy using AWS CLI
aws cloudformation create-stack \
  --stack-name masetra-production \
  --template-body file://masetra-aws-template.yaml \
  --parameters ParameterKey=DatabasePassword,ParameterValue=YourSecurePassword \
  --capabilities CAPABILITY_IAM
```

2. **Configure Environment Variables**

```bash
# Set environment variables
export AWS_REGION=us-east-1
export ENVIRONMENT=production
export DATABASE_URL=postgresql://masetra:password@db.masetra.internal:5432/masetra
export REDIS_URL=redis://redis.masetra.internal:6379/0
```

3. **Deploy Application Containers**

```bash
# Pull latest images
docker pull masetra/web:latest
docker pull masetra/api:latest
docker pull masetra/worker:latest

# Deploy to ECS
aws ecs update-service \
  --cluster masetra-cluster \
  --service masetra-web-service \
  --task-definition masetra-web:1 \
  --desired-count 2
```

4. **Configure Load Balancer**

```bash
# Update target groups
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/masetra-tg/1234567890123456 \
  --attributes Key=deregistration_delay.timeout_seconds,Value=30
```

AWS Resources Created

- EC2 Instances: Application servers
- RDS PostgreSQL: Managed database
- ElastiCache Redis: Caching layer
- S3 Buckets: File storage
- Elastic Load Balancer: Traffic distribution
- CloudFront: Content delivery network
- Route 53: DNS management
- CloudWatch: Monitoring and logging

Azure Deployment
Prerequisites

- Azure subscription with appropriate permissions
- Resource group created
- Domain name configured in Azure DNS

Deployment Steps

1. Deploy ARM Template

```bash
# Deploy using Azure CLI
az deployment group create \
  --resource-group masetra-rg \
  --template-file masetra-azure-template.json \
  --parameters @masetra-parameters.json
```

2. Configure Application Settings

```bash
# Set application settings
az webapp config appsettings set \
  --name masetra-app \
  --resource-group masetra-rg \
  --settings \
    DATABASE_URL="postgresql://masetra:password@db.masetra.azure.com:5432/masetra" \
    REDIS_URL="redis://redis.masetra.azure.com:6379/0"
```

3. Deploy Container Images

```bash
# Configure container settings
az webapp config container set \
  --name masetra-app \
  --resource-group masetra-rg \
  --docker-custom-image-name masetra/web:latest \
  --docker-registry-server-url https://index.docker.io
```

🏢 On-Premises Deployment
System Requirements
Hardware Requirements

Minimum Configuration:

- CPU: 4 cores (8 cores recommended)
- RAM: 16GB (32GB recommended)
- Storage: 100GB SSD (500GB recommended)
- Network: 1Gbps network interface

High Availability Configuration:

- CPU: 8 cores
- RAM: 32GB
- Storage: 500GB SSD
- Redundancy: RAID 1 or better

Software Requirements

- Operating System: Ubuntu 20.04+ or CentOS 8+
- Container Runtime: Docker 20.10+
- Container Orchestration: Docker Compose or Kubernetes
- Database: PostgreSQL 13+
- Cache: Redis 6+
- Web Server: Nginx or Apache


Installation Steps

1. Prepare Environment

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

2. Download Installation Package

```bash
# Download Masetra deployment package
wget https://downloads.masetra.com/on-premises/masetra-onprem-latest.tar.gz
tar -xzf masetra-onprem-latest.tar.gz
cd masetra-deployment
```

3. Configure Environment

```bash
# Copy and edit configuration
cp .env.example .env
nano .env

# Example .env configuration
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=masetra
DATABASE_USER=masetra_user
DATABASE_PASSWORD=secure_password
REDIS_HOST=localhost
REDIS_PORT=6379
SECRET_KEY=your_very_secret_key_here
DEBUG=False
```

4. Start Services

```bash
# Start all services
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f
```

5. Initialize Application

```bash
# Run database migrations
docker-compose exec web python manage.py migrate

# Create superuser
docker-compose exec web python manage.py createsuperuser

# Load initial data
docker-compose exec web python manage.py loaddata initial_data.
```

6. Configure Reverse Proxy

```nginx
# Nginx configuration example
server {
    listen 443 ssl;
    server_name masetra.yourcompany.com;

    ssl_certificate /path/to/ssl/certificate.crt;
    ssl_certificate_key /path/to/ssl/private.key;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static/ {
        alias /var/www/masetra/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

High Availability Setup
Database Clustering

```bash
# PostgreSQL replication setup
# Primary server configuration
postgresql.conf:
wal_level = replica
max_wal_senders = 3
wal_keep_size = 512MB

pg_hba.conf:
host replication replicator 192.168.1.101/32 md5
```

Load Balancer Configuration

```bash
# HAProxy configuration
frontend masetra_frontend
    bind *:443 ssl crt /etc/ssl/certs/masetra.pem
    mode http
    default_backend masetra_backend

backend masetra_backend
    mode http
    balance roundrobin
    server app1 192.168.1.10:8000 check
    server app2 192.168.1.11:8000 check
    server app3 192.168.1.12:8000 check
```

⚙️ Configuration Management
Environment Variables
Core Configuration

```env
# Application Settings
SECRET_KEY=your_secret_key_here
DEBUG=False
ALLOWED_HOSTS=masetra.yourcompany.com,localhost
ENVIRONMENT=production

# Database Configuration
DATABASE_URL=postgresql://user:password@host:port/database
DATABASE_POOL_TIMEOUT=30
DATABASE_CONN_MAX_AGE=600

# Cache Configuration
REDIS_URL=redis://localhost:6379/0
CACHE_TIMEOUT=300

# Email Configuration
EMAIL_HOST=smtp.yourcompany.com
EMAIL_PORT=587
EMAIL_HOST_USER=notifications@masetra.com
EMAIL_HOST_PASSWORD=secure_password
EMAIL_USE_TLS=True

# Security Settings
SECURE_SSL_REDIRECT=True
SECURE_HSTS_SECONDS=31536000
SECURE_CONTENT_TYPE_NOSNIFF=True
SECURE_BROWSER_XSS_FILTER=True
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
```

Advanced Configuration

```env
# Performance Settings
WORKER_PROCESSES=4
MAX_REQUESTS=1000
MAX_REQUESTS_JITTER=100

# Logging Configuration
LOG_LEVEL=INFO
LOG_FORMAT=json
LOG_FILE=/var/log/masetra/application.log

# Feature Flags
FEATURE_AUDIT_TRAIL=true
FEATURE_MULTI_TENANCY=false
FEATURE_ADVANCED_REPORTING=true

# Integration Settings
INTEGRATION_ERP_ENABLED=true
INTEGRATION_LIMS_ENABLED=false
INTEGRATION_EMAIL_PROVIDER=sendgrid
```

Configuration Files
Docker Compose Configuration

```yaml
version: '3.8'

services:
  web:
    image: masetra/web:latest
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - SECRET_KEY=${SECRET_KEY}
    depends_on:
      - db
      - redis
    volumes:
      - static_volume:/app/static
      - media_volume:/app/media

  api:
    image: masetra/api:latest
    ports:
      - "8001:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - SECRET_KEY=${SECRET_KEY}
    depends_on:
      - db
      - redis

  worker:
    image: masetra/worker:latest
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    depends_on:
      - db
      - redis

  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=${DATABASE_NAME}
      - POSTGRES_USER=${DATABASE_USER}
      - POSTGRES_PASSWORD=${DATABASE_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
  static_volume:
  media_volume:
```

🔧 Maintenance and Operations
Backup and Recovery
Automated Backup Script

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/masetra"
S3_BUCKET="s3://masetra-backups"

# Database backup
pg_dump -h localhost -U masetra_user masetra > $BACKUP_DIR/db_backup_$DATE.sql

# File backup
tar -czf $BACKUP_DIR/files_backup_$DATE.tar.gz /var/www/masetra/media/

# Upload to S3
aws s3 cp $BACKUP_DIR/db_backup_$DATE.sql $S3_BUCKET/db/
aws s3 cp $BACKUP_DIR/files_backup_$DATE.tar.gz $S3_BUCKET/files/

# Cleanup old backups (keep 30 days)
find $BACKUP_DIR -name "*.sql" -mtime +30 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete
```

Restore Process

```bash
# Restore database
psql -h localhost -U masetra_user masetra < db_backup_20240115_143000.sql

# Restore files
tar -xzf files_backup_20240115_143000.tar.gz -C /
```

Monitoring and Health Checks
Health Check Endpoints

```bash
# Application health
curl -f https://masetra.yourcompany.com/health/

# Database health
curl -f https://masetra.yourcompany.com/health/database/

# Cache health
curl -f https://masetra.yourcompany.com/health/cache/

# API health
curl -f https://masetra.yourcompany.com/api/health/
```

Monitoring Configuration

```yaml
# Prometheus configuration
scrape_configs:
  - job_name: 'masetra'
    static_configs:
      - targets: ['masetra.yourcompany.com:8000']
    metrics_path: '/metrics'
    scrape_interval: 30s
```

Update and Upgrade Process
Version Upgrade Steps

1. Backup: Create full system backup

2. Maintenance Mode: Enable maintenance mode

3. Database Migration: Apply database schema changes

4. Application Update: Deploy new application version

5. Service Restart: Restart all services

6. Health Check: Verify system functionality

7. Maintenance Mode Off: Disable maintenance mode

```bash
# Example upgrade script
#!/bin/bash

# Enable maintenance mode
touch /var/www/masetra/maintenance.flag

# Pull new images
docker-compose pull

# Stop services
docker-compose down

# Apply migrations
docker-compose run --rm web python manage.py migrate

# Start services
docker-compose up -d

# Wait for services to start
sleep 30

# Verify health
curl -f https://masetra.yourcompany.com/health/

# Disable maintenance mode
rm /var/www/masetra/maintenance.flag
```


🔒 Security Considerations
Network Security
Firewall Rules: Restrict access to necessary ports only
SSL/TLS: Enforce HTTPS for all communications
Intrusion Detection: Deploy IDS/IPS systems
Regular Scanning: Perform vulnerability assessments

Data Security
Encryption: Enable encryption at rest and in transit
Access Control: Implement principle of least privilege
Audit Logging: Maintain comprehensive audit trails
Data Retention: Configure appropriate retention policies

Compliance
21 CFR Part 11: Ensure electronic signature compliance
GDPR: Implement data protection measures
ISO Standards: Follow quality management standards
Local Regulations: Comply with Uganda and East Africa regulations

📞 Support
For deployment assistance, contact info@masetra.com or call +256-760-927180.

Last updated: {{ site.time | date: '%B %d, %Y' }}



