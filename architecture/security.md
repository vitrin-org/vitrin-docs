# Security Architecture

Security considerations and implementation for the Vitrin platform.

## Overview

The Vitrin platform implements comprehensive security measures to protect user data, prevent unauthorized access, and ensure system integrity. Security is implemented at multiple layers including authentication, authorization, data protection, and infrastructure security.

## Security Principles

### Defense in Depth

- **Multiple layers**: Security controls at every layer
- **Fail secure**: System fails in a secure state
- **Least privilege**: Users have minimum necessary permissions
- **Separation of duties**: Different roles for different functions

### Security by Design

- **Secure defaults**: Secure configuration by default
- **Input validation**: Validate all input data
- **Output encoding**: Encode output to prevent injection
- **Error handling**: Secure error handling and logging

## Authentication

### Multi-Factor Authentication

#### JWT Authentication

```python
# JWT token structure
{
  "user_id": 123,
  "email": "user@example.com",
  "exp": 1640995200,
  "iat": 1640908800,
  "jti": "unique-token-id"
}
```

#### Google OAuth Integration

```python
# OAuth flow
1. User clicks "Login with Google"
2. Redirect to Google OAuth
3. User authorizes application
4. Google returns authorization code
5. Exchange code for access token
6. Verify token and create user session
```

### Password Security

#### Password Requirements

- **Minimum length**: 8 characters
- **Complexity**: Mixed case, numbers, symbols
- **History**: Cannot reuse last 5 passwords
- **Expiration**: 90 days (configurable)

#### Password Hashing

```python
# Using bcrypt for password hashing
import bcrypt

def hash_password(password):
    salt = bcrypt.gensalt()
    hashed = bcrypt.hashpw(password.encode('utf-8'), salt)
    return hashed.decode('utf-8')

def verify_password(password, hashed):
    return bcrypt.checkpw(password.encode('utf-8'), hashed.encode('utf-8'))
```

### Session Management

#### Session Security

- **Secure cookies**: HTTPS only, HttpOnly, SameSite
- **Session timeout**: 30 minutes of inactivity
- **Session rotation**: New session on login
- **Concurrent sessions**: Limit to 5 active sessions

#### Token Management

```python
# JWT token configuration
JWT_SETTINGS = {
    'ACCESS_TOKEN_LIFETIME': timedelta(hours=1),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': settings.SECRET_KEY,
    'VERIFYING_KEY': None,
    'AUDIENCE': None,
    'ISSUER': 'vitrin-api',
    'AUTH_HEADER_TYPES': ('Bearer',),
    'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
}
```

## Authorization

### Role-Based Access Control (RBAC)

#### User Roles

```python
class UserRole(models.TextChoices):
    USER = 'user', 'Regular User'
    MODERATOR = 'moderator', 'Moderator'
    ADMIN = 'admin', 'Administrator'
    SUPERUSER = 'superuser', 'Super User'
```

#### Permission System

```python
# Django permissions
class ProductPermissions:
    CAN_VIEW = 'products.view_product'
    CAN_ADD = 'products.add_product'
    CAN_CHANGE = 'products.change_product'
    CAN_DELETE = 'products.delete_product'
    CAN_MODERATE = 'products.moderate_product'
```

### Resource-Level Permissions

#### Object-Level Permissions

```python
class ProductPermission(BasePermission):
    def has_object_permission(self, request, view, obj):
        # Users can only edit their own products
        if request.method in ['PUT', 'PATCH', 'DELETE']:
            return obj.user == request.user
        return True
```

#### API Endpoint Protection

```python
# View-level permissions
class ProductViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated, ProductPermission]
    
    def get_queryset(self):
        if self.request.user.is_staff:
            return Product.objects.all()
        return Product.objects.filter(user=self.request.user)
```

## Data Protection

### Encryption

#### Data at Rest

- **Database encryption**: PostgreSQL TDE (Transparent Data Encryption)
- **File encryption**: MinIO server-side encryption
- **Backup encryption**: Encrypted database backups
- **Key management**: AWS KMS or similar service

#### Data in Transit

- **HTTPS**: TLS 1.3 for all communications
- **API encryption**: JWT tokens for API authentication
- **Database connections**: SSL/TLS encrypted connections
- **File uploads**: Encrypted file transfers

### Personal Data Protection

#### GDPR Compliance

```python
# Data anonymization
def anonymize_user_data(user):
    user.email = f"anonymous_{user.id}@deleted.com"
    user.first_name = "Anonymous"
    user.last_name = "User"
    user.profile.bio = None
    user.profile.avatar = None
    user.save()
```

#### Data Retention

- **User data**: Retained for account lifetime
- **Product data**: Retained for 7 years after deletion
- **Log data**: Retained for 1 year
- **Backup data**: Retained for 3 years

### Input Validation

#### Data Sanitization

```python
# Input validation
from django.core.validators import validate_email, URLValidator
from django.utils.html import escape

def sanitize_input(data):
    # Escape HTML content
    if isinstance(data, str):
        data = escape(data)
    
    # Validate email
    if 'email' in data:
        validate_email(data['email'])
    
    # Validate URLs
    if 'website' in data:
        URLValidator()(data['website'])
    
    return data
```

#### File Upload Security

```python
# File upload validation
ALLOWED_IMAGE_TYPES = ['image/jpeg', 'image/png', 'image/gif', 'image/webp']
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB

def validate_file_upload(file):
    if file.content_type not in ALLOWED_IMAGE_TYPES:
        raise ValidationError("Invalid file type")
    
    if file.size > MAX_FILE_SIZE:
        raise ValidationError("File too large")
    
    # Scan for malware
    if not scan_file_for_malware(file):
        raise ValidationError("File contains malware")
```

## API Security

### Rate Limiting

#### Implementation

```python
# Django REST Framework throttling
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

#### Custom Throttling

```python
class LoginRateThrottle(AnonRateThrottle):
    scope = 'login'
    
    def get_cache_key(self, request, view):
        if request.method == 'POST' and 'login' in request.path:
            return f"login_{self.get_ident(request)}"
        return None
```

### CORS Configuration

```python
# CORS settings
CORS_ALLOWED_ORIGINS = [
    "https://vitrin.com",
    "https://www.vitrin.com",
    "https://app.vitrin.com",
]

CORS_ALLOW_CREDENTIALS = True
CORS_ALLOW_HEADERS = [
    'accept',
    'accept-encoding',
    'authorization',
    'content-type',
    'dnt',
    'origin',
    'user-agent',
    'x-csrftoken',
    'x-requested-with',
]
```

### Security Headers

```python
# Security middleware
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_SECONDS = 31536000
SECURE_REDIRECT_EXEMPT = []
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
X_FRAME_OPTIONS = 'DENY'
```

## Infrastructure Security

### Network Security

#### Firewall Configuration

```bash
# UFW firewall rules
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

#### Network Segmentation

- **DMZ**: Public-facing servers
- **Application tier**: Application servers
- **Database tier**: Database servers
- **Management tier**: Administrative access

### Server Security

#### System Hardening

```bash
# Disable unnecessary services
systemctl disable bluetooth
systemctl disable cups
systemctl disable avahi-daemon

# Configure automatic security updates
apt install unattended-upgrades
dpkg-reconfigure unattended-upgrades
```

#### User Access Control

```bash
# Create application user
useradd -r -s /bin/false vitrin
usermod -L vitrin

# Configure sudo access
echo "vitrin ALL=(ALL) NOPASSWD: /bin/systemctl restart vitrin" >> /etc/sudoers
```

### Container Security

#### Docker Security

```dockerfile
# Use non-root user
RUN adduser --disabled-password --gecos '' vitrin
USER vitrin

# Use specific image tags
FROM python:3.11-slim

# Remove unnecessary packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*
```

#### Kubernetes Security

```yaml
# Security context
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
  containers:
  - name: vitrin-backend
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

## Monitoring and Logging

### Security Monitoring

#### Log Analysis

```python
# Security event logging
import logging

security_logger = logging.getLogger('security')

def log_security_event(event_type, user, details):
    security_logger.warning(f"Security event: {event_type} - User: {user} - Details: {details}")
```

#### Intrusion Detection

- **Failed login attempts**: Monitor and alert
- **Unusual access patterns**: Detect anomalies
- **Privilege escalation**: Monitor permission changes
- **Data exfiltration**: Monitor large data transfers

### Audit Logging

#### Audit Trail

```python
# Audit log model
class AuditLog(models.Model):
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    action = models.CharField(max_length=100)
    resource_type = models.CharField(max_length=50)
    resource_id = models.CharField(max_length=100)
    details = models.JSONField()
    ip_address = models.GenericIPAddressField()
    user_agent = models.TextField()
    timestamp = models.DateTimeField(auto_now_add=True)
```

#### Compliance Reporting

- **GDPR compliance**: Data processing reports
- **SOC 2**: Security control reports
- **PCI DSS**: Payment card industry compliance
- **ISO 27001**: Information security management

## Incident Response

### Security Incident Plan

#### Incident Classification

- **Critical**: Data breach, system compromise
- **High**: Unauthorized access, privilege escalation
- **Medium**: Failed attacks, suspicious activity
- **Low**: Policy violations, minor security issues

#### Response Procedures

1. **Detection**: Identify security incident
2. **Assessment**: Evaluate severity and impact
3. **Containment**: Isolate affected systems
4. **Eradication**: Remove threat and vulnerabilities
5. **Recovery**: Restore normal operations
6. **Lessons learned**: Improve security measures

### Backup and Recovery

#### Data Backup

```bash
# Database backup
pg_dump -h localhost -U vitrin -d vitrin_db > backup_$(date +%Y%m%d_%H%M%S).sql

# File backup
rsync -av /var/www/vitrin/media/ /backup/media/
```

#### Disaster Recovery

- **RTO**: Recovery Time Objective (4 hours)
- **RPO**: Recovery Point Objective (1 hour)
- **Backup testing**: Monthly restore tests
- **Failover procedures**: Automated failover systems

## Security Testing

### Vulnerability Assessment

#### Automated Scanning

```bash
# OWASP ZAP scanning
docker run -t owasp/zap2docker-stable zap-baseline.py -t https://api.vitrin.com

# Dependency scanning
safety check --json
bandit -r . -f json
```

#### Penetration Testing

- **Annual testing**: Professional penetration testing
- **Bug bounty**: Public bug bounty program
- **Internal testing**: Regular internal security tests
- **Code review**: Security-focused code reviews

### Security Training

#### Developer Training

- **Secure coding**: OWASP secure coding practices
- **Security awareness**: Regular security training
- **Incident response**: Security incident procedures
- **Compliance**: Regulatory compliance training
