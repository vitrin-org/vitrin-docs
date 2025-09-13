# Deployment Guide

Complete guide for deploying the Vitrin platform to production.

## Overview

This guide covers deploying the Vitrin platform to production environments, including cloud platforms, container orchestration, and traditional server deployments.

## Deployment Options

### 1. Docker Deployment (Recommended)

#### Prerequisites

- Docker and Docker Compose
- Domain name and SSL certificate
- Cloud provider account (AWS, GCP, Azure)

#### Production Configuration

```bash
# Clone infrastructure repository
git clone https://github.com/vitrin-org/vitrin-infrastructure.git
cd vitrin-infrastructure

# Configure environment
cp .env.production .env
```

Edit `.env` file:

```bash
# .env
# Django Settings
SECRET_KEY=your-production-secret-key
DEBUG=False
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com

# Database
POSTGRES_DB=vitrin_prod
POSTGRES_USER=vitrin
POSTGRES_PASSWORD=secure-password
POSTGRES_HOST=postgres
POSTGRES_PORT=5432

# Redis
REDIS_URL=redis://redis:6379/0

# MinIO
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=production-access-key
MINIO_SECRET_KEY=production-secret-key
MINIO_BUCKET_NAME=vitrin-prod

# Google OAuth
GOOGLE_OAUTH2_CLIENT_ID=your-google-client-id
GOOGLE_OAUTH2_CLIENT_SECRET=your-google-client-secret

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password
```

#### Deploy with Docker Compose

```bash
# Build and start services
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Run migrations
docker-compose exec backend python manage.py migrate

# Create superuser
docker-compose exec backend python manage.py createsuperuser

# Collect static files
docker-compose exec backend python manage.py collectstatic --noinput

# Load initial data
docker-compose exec backend python manage.py loaddata fixtures/initial_data.json
```

### 2. Cloud Platform Deployment

#### AWS Deployment

##### Using AWS ECS

```bash
# Build and push images
docker build -t your-account.dkr.ecr.region.amazonaws.com/vitrin-backend .
docker push your-account.dkr.ecr.region.amazonaws.com/vitrin-backend

# Deploy with ECS
aws ecs create-service \
  --cluster vitrin-cluster \
  --service-name vitrin-backend \
  --task-definition vitrin-backend:1 \
  --desired-count 2
```

##### Using AWS Elastic Beanstalk

```bash
# Install EB CLI
pip install awsebcli

# Initialize EB application
eb init vitrin-app

# Create environment
eb create production

# Deploy
eb deploy
```

#### Google Cloud Platform

##### Using Cloud Run

```bash
# Build and push image
gcloud builds submit --tag gcr.io/your-project/vitrin-backend

# Deploy to Cloud Run
gcloud run deploy vitrin-backend \
  --image gcr.io/your-project/vitrin-backend \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

##### Using Google Kubernetes Engine

```bash
# Create cluster
gcloud container clusters create vitrin-cluster \
  --num-nodes=3 \
  --zone=us-central1-a

# Deploy application
kubectl apply -f k8s/
```

#### Azure Deployment

##### Using Azure Container Instances

```bash
# Create resource group
az group create --name vitrin-rg --location eastus

# Deploy container
az container create \
  --resource-group vitrin-rg \
  --name vitrin-backend \
  --image your-registry.azurecr.io/vitrin-backend \
  --dns-name-label vitrin-backend \
  --ports 8000
```

### 3. Traditional Server Deployment

#### Ubuntu Server Setup

##### Install Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python
sudo apt install python3.11 python3.11-venv python3.11-dev

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Install Redis
sudo apt install redis-server

# Install Nginx
sudo apt install nginx

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs
```

##### Configure Database

```bash
# Create database and user
sudo -u postgres psql
CREATE DATABASE vitrin_prod;
CREATE USER vitrin WITH PASSWORD 'secure-password';
GRANT ALL PRIVILEGES ON DATABASE vitrin_prod TO vitrin;
\q
```

##### Deploy Backend

```bash
# Clone repository
git clone https://github.com/vitrin-org/vitrin-backend.git
cd vitrin-backend

# Create virtual environment
python3.11 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements/production.txt

# Configure environment
cp .env.production .env
# Edit .env with production settings

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Collect static files
python manage.py collectstatic --noinput

# Start with Gunicorn
gunicorn --bind 0.0.0.0:8000 vitrin.wsgi:application
```

##### Deploy Frontend

```bash
# Clone repository
git clone https://github.com/vitrin-org/vitrin-frontend.git
cd vitrin-frontend

# Install dependencies
npm install

# Build for production
npm run build

# Serve with Nginx
sudo cp -r dist/* /var/www/html/
```

##### Configure Nginx

```nginx
# /etc/nginx/sites-available/vitrin
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # Frontend
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }

    # Backend API
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Static files
    location /static/ {
        alias /path/to/vitrin-backend/staticfiles/;
    }

    # Media files
    location /media/ {
        alias /path/to/vitrin-backend/media/;
    }
}
```

## Environment Configuration

### Production Settings

#### Django Settings

```python
# settings/production.py
import os
from .base import *

# Security
SECRET_KEY = os.environ.get('SECRET_KEY')
DEBUG = False
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
        'OPTIONS': {
            'sslmode': 'require',
        },
    }
}

# Static files
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.StaticFilesStorage'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

# Media files
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')
AWS_S3_REGION_NAME = os.environ.get('AWS_S3_REGION_NAME')
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'

# Email
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = os.environ.get('EMAIL_HOST')
EMAIL_PORT = int(os.environ.get('EMAIL_PORT', 587))
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': os.path.join(BASE_DIR, 'logs', 'django.log'),
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['file', 'console'],
        'level': 'INFO',
    },
}
```

#### Frontend Configuration

```bash
# .env.production
VITE_API_BASE_URL=https://api.yourdomain.com/api
VITE_GOOGLE_CLIENT_ID=your-google-client-id
VITE_APP_NAME=Vitrin
VITE_APP_VERSION=1.0.0
```

## SSL/TLS Configuration

### Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

### Cloudflare SSL

```bash
# Configure Cloudflare
# 1. Add domain to Cloudflare
# 2. Update nameservers
# 3. Enable SSL/TLS encryption
# 4. Configure page rules
```

## Database Setup

### PostgreSQL Configuration

```bash
# /etc/postgresql/13/main/postgresql.conf
listen_addresses = 'localhost'
port = 5432
max_connections = 100
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
```

### Database Backup

```bash
# Automated backup script
#!/bin/bash
# /usr/local/bin/backup-db.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"
DB_NAME="vitrin_prod"
DB_USER="vitrin"

# Create backup
pg_dump -h localhost -U $DB_USER -d $DB_NAME > $BACKUP_DIR/vitrin_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/vitrin_$DATE.sql

# Remove old backups (keep 30 days)
find $BACKUP_DIR -name "vitrin_*.sql.gz" -mtime +30 -delete

# Upload to S3
aws s3 cp $BACKUP_DIR/vitrin_$DATE.sql.gz s3://your-backup-bucket/
```

## Monitoring and Logging

### Application Monitoring

#### Sentry Integration

```python
# settings/production.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn="https://your-sentry-dsn@sentry.io/project-id",
    integrations=[DjangoIntegration()],
    traces_sample_rate=1.0,
    send_default_pii=True
)
```

#### Health Checks

```python
# apps/core/views.py
from django.http import JsonResponse
from django.db import connection
from django.core.cache import cache

def health_check(request):
    try:
        # Check database
        with connection.cursor() as cursor:
            cursor.execute("SELECT 1")
        
        # Check cache
        cache.set('health_check', 'ok', 10)
        cache.get('health_check')
        
        return JsonResponse({
            'status': 'healthy',
            'timestamp': timezone.now().isoformat(),
            'version': '1.0.0'
        })
    except Exception as e:
        return JsonResponse({
            'status': 'unhealthy',
            'error': str(e)
        }, status=500)
```

### Server Monitoring

#### System Monitoring

```bash
# Install monitoring tools
sudo apt install htop iotop nethogs

# Configure log rotation
sudo nano /etc/logrotate.d/vitrin
```

```bash
# /etc/logrotate.d/vitrin
/path/to/vitrin-backend/logs/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 644 www-data www-data
}
```

## Performance Optimization

### Database Optimization

```python
# Database indexes
class Product(models.Model):
    name = models.CharField(max_length=255, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    price = models.DecimalField(max_digits=10, decimal_places=2, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['category', 'created_at']),
            models.Index(fields=['user', 'is_active']),
        ]
```

### Caching Configuration

```python
# settings/production.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': os.environ.get('REDIS_URL'),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Session configuration
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
```

### CDN Configuration

```python
# settings/production.py
STATIC_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/static/'
MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/media/'

# CloudFront configuration
AWS_S3_CUSTOM_DOMAIN = 'd1234567890.cloudfront.net'
```

## Security Configuration

### Firewall Setup

```bash
# Configure UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### Security Headers

```python
# settings/production.py
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_SECONDS = 31536000
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
X_FRAME_OPTIONS = 'DENY'
```

### Rate Limiting

```python
# settings/production.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',
        'user': '1000/hour',
        'login': '5/min',
        'register': '10/hour'
    }
}
```

## Backup and Recovery

### Automated Backups

```bash
# Database backup script
#!/bin/bash
# /usr/local/bin/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"
DB_NAME="vitrin_prod"

# Database backup
pg_dump -h localhost -U vitrin -d $DB_NAME | gzip > $BACKUP_DIR/db_$DATE.sql.gz

# File backup
tar -czf $BACKUP_DIR/files_$DATE.tar.gz /path/to/vitrin-backend/media/

# Upload to cloud storage
aws s3 sync $BACKUP_DIR s3://your-backup-bucket/backups/

# Cleanup local backups
find $BACKUP_DIR -name "*.gz" -mtime +7 -delete
```

### Disaster Recovery

```bash
# Recovery script
#!/bin/bash
# /usr/local/bin/recover.sh

BACKUP_DATE=$1
BACKUP_DIR="/backups"

# Restore database
gunzip -c $BACKUP_DIR/db_$BACKUP_DATE.sql.gz | psql -h localhost -U vitrin -d vitrin_prod

# Restore files
tar -xzf $BACKUP_DIR/files_$BACKUP_DATE.tar.gz -C /

# Restart services
sudo systemctl restart nginx
sudo systemctl restart gunicorn
```

## Deployment Checklist

### Pre-deployment

- [ ] Code review completed
- [ ] Tests passing
- [ ] Security scan completed
- [ ] Performance testing done
- [ ] Database migrations ready
- [ ] Environment variables configured
- [ ] SSL certificates obtained
- [ ] Domain DNS configured

### Deployment

- [ ] Backup current system
- [ ] Deploy new version
- [ ] Run database migrations
- [ ] Update static files
- [ ] Restart services
- [ ] Verify deployment
- [ ] Run smoke tests
- [ ] Monitor logs

### Post-deployment

- [ ] Health checks passing
- [ ] Performance metrics normal
- [ ] Error rates acceptable
- [ ] User feedback positive
- [ ] Monitoring alerts configured
- [ ] Documentation updated

## Troubleshooting

### Common Issues

#### 1. Database Connection Issues

```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Check connection
psql -h localhost -U vitrin -d vitrin_prod

# Check logs
sudo tail -f /var/log/postgresql/postgresql-13-main.log
```

#### 2. Application Errors

```bash
# Check application logs
tail -f /path/to/vitrin-backend/logs/django.log

# Check Gunicorn logs
tail -f /var/log/gunicorn/error.log

# Check Nginx logs
tail -f /var/log/nginx/error.log
```

#### 3. Performance Issues

```bash
# Check system resources
htop
iotop
nethogs

# Check database performance
sudo -u postgres psql -d vitrin_prod -c "SELECT * FROM pg_stat_activity;"

# Check slow queries
sudo -u postgres psql -d vitrin_prod -c "SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;"
```

### Rollback Procedures

```bash
# Rollback to previous version
git checkout previous-version
pip install -r requirements/production.txt
python manage.py migrate
sudo systemctl restart gunicorn

# Rollback database
psql -h localhost -U vitrin -d vitrin_prod < backup.sql
```

## Maintenance

### Regular Maintenance

- [ ] Update dependencies
- [ ] Apply security patches
- [ ] Monitor disk space
- [ ] Check log files
- [ ] Review performance metrics
- [ ] Backup verification
- [ ] SSL certificate renewal

### Monitoring Alerts

- [ ] High error rates
- [ ] Slow response times
- [ ] High memory usage
- [ ] Disk space low
- [ ] Database connection issues
- [ ] SSL certificate expiration
