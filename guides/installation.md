# Installation Guide

Complete installation instructions for the Vitrin platform.

## Overview

This guide covers the installation of the complete Vitrin platform, including the backend API, frontend application, and infrastructure components.

## Prerequisites

### System Requirements

- **Operating System**: Linux (Ubuntu 20.04+), macOS (10.15+), or Windows 10+
- **Memory**: Minimum 4GB RAM, recommended 8GB+
- **Storage**: Minimum 10GB free space
- **Network**: Internet connection for downloading dependencies

### Required Software

- **Python**: 3.11 or higher
- **Node.js**: 18 or higher
- **PostgreSQL**: 13 or higher
- **Redis**: 6 or higher
- **Docker**: 20.10 or higher (optional)
- **Git**: Latest version

## Installation Methods

### Method 1: Docker Installation (Recommended)

#### 1. Clone the Repository

```bash
git clone https://github.com/vitrin-org/vitrin-infrastructure.git
cd vitrin-infrastructure
```

#### 2. Configure Environment

```bash
cp .env.example .env
# Edit .env with your configuration
```

#### 3. Start Services

```bash
# Development environment
docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d

# Production environment
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

#### 4. Initialize Database

```bash
# Run migrations
docker-compose exec backend python manage.py migrate

# Create superuser
docker-compose exec backend python manage.py createsuperuser

# Load initial data
docker-compose exec backend python manage.py loaddata fixtures/initial_data.json
```

### Method 2: Manual Installation

#### Backend Installation

##### 1. Clone Backend Repository

```bash
git clone https://github.com/vitrin-org/vitrin-backend.git
cd vitrin-backend
```

##### 2. Create Virtual Environment

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

##### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

##### 4. Configure Environment

```bash
cp .env.example .env
# Edit .env with your configuration
```

##### 5. Database Setup

```bash
# Create PostgreSQL database
createdb vitrin_dev

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Load initial data
python manage.py loaddata fixtures/initial_data.json
```

##### 6. Start Development Server

```bash
python manage.py runserver
```

#### Frontend Installation

##### 1. Clone Frontend Repository

```bash
git clone https://github.com/vitrin-org/vitrin-frontend.git
cd vitrin-frontend
```

##### 2. Install Dependencies

```bash
npm install
```

##### 3. Configure Environment

```bash
cp .env.example .env
# Edit .env with your configuration
```

##### 4. Start Development Server

```bash
npm run dev
```

## Configuration

### Environment Variables

#### Backend Configuration

```bash
# .env
# Django Settings
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DATABASE_URL=postgresql://vitrin:password@localhost:5432/vitrin_dev

# Redis
REDIS_URL=redis://localhost:6379/0

# MinIO Storage
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET_NAME=vitrin-dev

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

#### Frontend Configuration

```bash
# .env
# API Configuration
VITE_API_BASE_URL=http://localhost:8000/api
VITE_API_TIMEOUT=10000

# Google OAuth
VITE_GOOGLE_CLIENT_ID=your-google-client-id

# App Configuration
VITE_APP_NAME=Vitrin
VITE_APP_VERSION=1.0.0
```

### Database Configuration

#### PostgreSQL Setup

```bash
# Install PostgreSQL
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib

# Create database and user
sudo -u postgres psql
CREATE DATABASE vitrin_dev;
CREATE USER vitrin WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE vitrin_dev TO vitrin;
\q
```

#### Redis Setup

```bash
# Install Redis
sudo apt-get install redis-server

# Start Redis
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

### MinIO Setup

#### Install MinIO

```bash
# Download MinIO
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/

# Create data directory
sudo mkdir -p /data/minio
sudo chown -R $USER:$USER /data/minio

# Start MinIO
minio server /data/minio --console-address ":9001"
```

#### Configure MinIO

```bash
# Create bucket
mc alias set local http://localhost:9000 minioadmin minioadmin
mc mb local/vitrin-dev
mc policy set public local/vitrin-dev
```

## Verification

### Backend Verification

#### 1. Check API Health

```bash
curl http://localhost:8000/api/health/
```

Expected response:
```json
{
  "status": "healthy",
  "timestamp": "2024-01-15T10:30:00Z",
  "version": "1.0.0"
}
```

#### 2. Check API Documentation

Visit: http://localhost:8000/api/docs/

#### 3. Test Authentication

```bash
# Register user
curl -X POST http://localhost:8000/api/auth/register/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "testpass123",
    "first_name": "Test",
    "last_name": "User"
  }'

# Login user
curl -X POST http://localhost:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "testpass123"
  }'
```

### Frontend Verification

#### 1. Check Application

Visit: http://localhost:3000

#### 2. Test Features

- User registration and login
- Product creation and viewing
- Image upload
- Search functionality

## Troubleshooting

### Common Issues

#### 1. Database Connection Error

```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Check database exists
psql -h localhost -U vitrin -d vitrin_dev -c "\l"
```

#### 2. Redis Connection Error

```bash
# Check Redis status
sudo systemctl status redis-server

# Test Redis connection
redis-cli ping
```

#### 3. MinIO Connection Error

```bash
# Check MinIO status
ps aux | grep minio

# Test MinIO connection
curl http://localhost:9000/minio/health/live
```

#### 4. Frontend Build Error

```bash
# Clear node modules
rm -rf node_modules package-lock.json
npm install

# Check Node.js version
node --version
npm --version
```

#### 5. Python Dependencies Error

```bash
# Update pip
pip install --upgrade pip

# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

### Log Files

#### Backend Logs

```bash
# Django logs
tail -f logs/django.log

# Gunicorn logs
tail -f logs/gunicorn.log

# Nginx logs
tail -f /var/log/nginx/error.log
```

#### Frontend Logs

```bash
# Development server logs
npm run dev

# Build logs
npm run build
```

### Performance Issues

#### 1. Slow Database Queries

```bash
# Enable query logging
# In settings.py
LOGGING = {
    'loggers': {
        'django.db.backends': {
            'level': 'DEBUG',
        },
    },
}
```

#### 2. High Memory Usage

```bash
# Check memory usage
free -h
ps aux --sort=-%mem | head

# Optimize Gunicorn workers
# In gunicorn.conf.py
workers = 2  # Reduce number of workers
```

#### 3. Slow Frontend Build

```bash
# Use faster build tools
npm install -g esbuild

# Enable caching
npm config set cache-min 86400
```

## Production Deployment

### Security Checklist

- [ ] Change default passwords
- [ ] Enable HTTPS
- [ ] Configure firewall
- [ ] Set up monitoring
- [ ] Enable backups
- [ ] Configure logging
- [ ] Set up error tracking

### Performance Optimization

- [ ] Enable database indexing
- [ ] Configure Redis caching
- [ ] Set up CDN
- [ ] Optimize images
- [ ] Enable compression
- [ ] Configure load balancing

### Monitoring Setup

- [ ] Application monitoring
- [ ] Database monitoring
- [ ] Server monitoring
- [ ] Log aggregation
- [ ] Alert configuration
- [ ] Performance metrics

## Support

### Getting Help

1. **Documentation**: Check the [API documentation](api/overview.md)
2. **Issues**: Create an issue in the appropriate repository
3. **Community**: Join our community discussions
4. **Email**: Contact support@vitrin.com

### Useful Commands

```bash
# Backend
python manage.py check
python manage.py collectstatic
python manage.py test

# Frontend
npm run lint
npm run test
npm run build

# Docker
docker-compose logs -f
docker-compose ps
docker-compose restart
```

### Next Steps

After successful installation:

1. Read the [Configuration Guide](configuration.md)
2. Set up [Development Environment](development-setup.md)
3. Review [API Documentation](api/overview.md)
4. Check [Deployment Guide](deployment.md)
