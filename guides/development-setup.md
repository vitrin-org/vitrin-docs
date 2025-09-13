# Development Setup

Complete guide for setting up a local development environment for the Vitrin platform.

## Overview

This guide covers setting up a local development environment for the Vitrin platform, including all necessary tools, configurations, and best practices for development.

## Prerequisites

### Required Software

- **Python**: 3.11 or higher
- **Node.js**: 18 or higher
- **PostgreSQL**: 13 or higher
- **Redis**: 6 or higher
- **Git**: Latest version
- **Docker**: 20.10 or higher (optional)

### Development Tools

- **VS Code**: Recommended IDE
- **Postman**: API testing
- **pgAdmin**: Database management
- **Redis Commander**: Redis management
- **MinIO Client**: Object storage management

## Environment Setup

### 1. Clone Repositories

```bash
# Create project directory
mkdir vitrin-dev
cd vitrin-dev

# Clone all repositories
git clone https://github.com/vitrin-org/vitrin-backend.git
git clone https://github.com/vitrin-org/vitrin-frontend.git
git clone https://github.com/vitrin-org/vitrin-shared.git
git clone https://github.com/vitrin-org/vitrin-docs.git
git clone https://github.com/vitrin-org/vitrin-infrastructure.git
```

### 2. Backend Setup

#### Create Virtual Environment

```bash
cd vitrin-backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

#### Install Dependencies

```bash
pip install -r requirements/development.txt
```

#### Configure Environment

```bash
cp .env.example .env
```

Edit `.env` file:

```bash
# .env
# Django Settings
SECRET_KEY=dev-secret-key-change-in-production
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DATABASE_URL=postgresql://vitrin:vitrin123@localhost:5432/vitrin_dev

# Redis
REDIS_URL=redis://localhost:6379/0

# MinIO Storage
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET_NAME=vitrin-dev

# Google OAuth (for testing)
GOOGLE_OAUTH2_CLIENT_ID=your-google-client-id
GOOGLE_OAUTH2_CLIENT_SECRET=your-google-client-secret

# Email (for testing)
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
```

#### Database Setup

```bash
# Create database
createdb vitrin_dev

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Load test data
python manage.py loaddata fixtures/test_data.json
```

#### Start Development Server

```bash
python manage.py runserver
```

### 3. Frontend Setup

#### Install Dependencies

```bash
cd ../vitrin-frontend
npm install
```

#### Configure Environment

```bash
cp .env.example .env
```

Edit `.env` file:

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

#### Start Development Server

```bash
npm run dev
```

### 4. Infrastructure Setup

#### Start Services with Docker

```bash
cd ../vitrin-infrastructure
docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

This will start:
- PostgreSQL database
- Redis cache
- MinIO object storage
- Nginx reverse proxy

## Development Tools Configuration

### VS Code Setup

#### Install Extensions

```bash
# Python
code --install-extension ms-python.python
code --install-extension ms-python.flake8
code --install-extension ms-python.black-formatter

# TypeScript/JavaScript
code --install-extension ms-vscode.vscode-typescript-next
code --install-extension esbenp.prettier-vscode
code --install-extension dbaeumer.vscode-eslint

# Django
code --install-extension batisteo.vscode-django

# React
code --install-extension dsznajder.es7-react-js-snippets

# Git
code --install-extension eamodio.gitlens
```

#### Workspace Configuration

Create `.vscode/settings.json`:

```json
{
  "python.defaultInterpreterPath": "./vitrin-backend/venv/bin/python",
  "python.linting.enabled": true,
  "python.linting.flake8Enabled": true,
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "files.exclude": {
    "**/__pycache__": true,
    "**/node_modules": true,
    "**/.git": true
  }
}
```

#### Launch Configuration

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Django Debug",
      "type": "python",
      "request": "launch",
      "program": "${workspaceFolder}/vitrin-backend/manage.py",
      "args": ["runserver"],
      "django": true,
      "justMyCode": false
    },
    {
      "name": "Django Tests",
      "type": "python",
      "request": "launch",
      "program": "${workspaceFolder}/vitrin-backend/manage.py",
      "args": ["test"],
      "django": true,
      "justMyCode": false
    }
  ]
}
```

### Git Configuration

#### Git Hooks

```bash
# Install pre-commit hooks
cd vitrin-backend
pip install pre-commit
pre-commit install

cd ../vitrin-frontend
npm install husky --save-dev
npx husky install
npx husky add .husky/pre-commit "npm run lint"
```

#### Git Aliases

```bash
# Add useful git aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## Development Workflow

### 1. Feature Development

#### Create Feature Branch

```bash
git checkout -b feature/new-feature
```

#### Make Changes

```bash
# Backend changes
cd vitrin-backend
# Make your changes
python manage.py test
python manage.py check

# Frontend changes
cd ../vitrin-frontend
# Make your changes
npm run test
npm run lint
```

#### Commit Changes

```bash
git add .
git commit -m "feat: add new feature"
```

#### Push and Create PR

```bash
git push origin feature/new-feature
# Create pull request on GitHub
```

### 2. Testing

#### Backend Testing

```bash
cd vitrin-backend

# Run all tests
python manage.py test

# Run specific test
python manage.py test apps.products.tests.test_models

# Run with coverage
coverage run --source='.' manage.py test
coverage report
coverage html
```

#### Frontend Testing

```bash
cd vitrin-frontend

# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

### 3. Code Quality

#### Backend Code Quality

```bash
cd vitrin-backend

# Format code
black .

# Lint code
flake8 .

# Type checking
mypy .

# Security check
bandit -r .
```

#### Frontend Code Quality

```bash
cd vitrin-frontend

# Format code
npm run format

# Lint code
npm run lint

# Type checking
npm run type-check
```

## Database Management

### Database Migrations

```bash
cd vitrin-backend

# Create migration
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Show migration status
python manage.py showmigrations

# Rollback migration
python manage.py migrate app_name 0001
```

### Database Seeding

```bash
# Load initial data
python manage.py loaddata fixtures/initial_data.json

# Create test data
python manage.py shell
>>> from apps.products.tests.factories import ProductFactory
>>> ProductFactory.create_batch(10)
```

### Database Backup

```bash
# Backup database
pg_dump -h localhost -U vitrin -d vitrin_dev > backup.sql

# Restore database
psql -h localhost -U vitrin -d vitrin_dev < backup.sql
```

## API Development

### API Testing

#### Using Postman

1. Import API collection from `vitrin-docs/postman/`
2. Set environment variables
3. Run tests

#### Using curl

```bash
# Test API health
curl http://localhost:8000/api/health/

# Test authentication
curl -X POST http://localhost:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "testpass123"}'

# Test product creation
curl -X POST http://localhost:8000/api/products/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"name": "Test Product", "price": "99.99"}'
```

### API Documentation

Visit: http://localhost:8000/api/docs/

## Frontend Development

### Component Development

#### Create New Component

```bash
cd vitrin-frontend/src/components
mkdir NewComponent
cd NewComponent
touch NewComponent.tsx
touch NewComponent.test.tsx
touch index.ts
```

#### Component Template

```typescript
// NewComponent.tsx
import React from 'react';
import { cn } from '@/utils/cn';

interface NewComponentProps {
  className?: string;
  children?: React.ReactNode;
}

export const NewComponent: React.FC<NewComponentProps> = ({
  className,
  children,
}) => {
  return (
    <div className={cn('new-component', className)}>
      {children}
    </div>
  );
};
```

### State Management

#### Using React Query

```typescript
// hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '@/services/api';

export const useProducts = (filters?: ProductFilters) => {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => api.products.list(filters),
  });
};

export const useCreateProduct = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: api.products.create,
    onSuccess: () => {
      queryClient.invalidateQueries(['products']);
    },
  });
};
```

## Debugging

### Backend Debugging

#### Django Debug Toolbar

```bash
# Install
pip install django-debug-toolbar

# Add to settings
INSTALLED_APPS = [
    'debug_toolbar',
    # ... other apps
]

MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    # ... other middleware
]

# Add to urls.py
if settings.DEBUG:
    import debug_toolbar
    urlpatterns = [
        path('__debug__/', include(debug_toolbar.urls)),
    ] + urlpatterns
```

#### Logging Configuration

```python
# settings/development.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
        'file': {
            'class': 'logging.FileHandler',
            'filename': 'logs/django.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
        },
        'apps': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
        },
    },
}
```

### Frontend Debugging

#### React Developer Tools

Install browser extension for React debugging.

#### Redux DevTools

```typescript
// For state management debugging
import { devToolsEnhancer } from '@redux-devtools/extension';

const store = createStore(
  rootReducer,
  devToolsEnhancer()
);
```

## Performance Optimization

### Backend Optimization

#### Database Optimization

```python
# Use select_related and prefetch_related
products = Product.objects.select_related('user', 'category').prefetch_related('images')

# Use database indexes
class Product(models.Model):
    name = models.CharField(max_length=255, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
```

#### Caching

```python
# Redis caching
from django.core.cache import cache

def get_products():
    products = cache.get('products')
    if not products:
        products = Product.objects.all()
        cache.set('products', products, 300)  # 5 minutes
    return products
```

### Frontend Optimization

#### Code Splitting

```typescript
// Lazy load components
const ProductDetail = lazy(() => import('./ProductDetail'));

// Route-based splitting
const App = () => (
  <Suspense fallback={<Loading />}>
    <Routes>
      <Route path="/products/:id" element={<ProductDetail />} />
    </Routes>
  </Suspense>
);
```

#### Image Optimization

```typescript
// Optimized image component
const OptimizedImage = ({ src, alt, ...props }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"
      onLoad={() => setIsLoaded(true)}
      style={{ opacity: isLoaded ? 1 : 0 }}
      {...props}
    />
  );
};
```

## Troubleshooting

### Common Issues

#### 1. Database Connection Issues

```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Check connection
psql -h localhost -U vitrin -d vitrin_dev
```

#### 2. Redis Connection Issues

```bash
# Check Redis status
sudo systemctl status redis-server

# Test connection
redis-cli ping
```

#### 3. Frontend Build Issues

```bash
# Clear cache
rm -rf node_modules package-lock.json
npm install

# Check Node version
node --version
```

#### 4. Python Import Issues

```bash
# Check virtual environment
which python
pip list

# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

### Debug Commands

```bash
# Backend
python manage.py check
python manage.py shell
python manage.py dbshell

# Frontend
npm run build
npm run analyze
npm run type-check
```

## Best Practices

### Code Organization

- Keep components small and focused
- Use TypeScript for type safety
- Follow naming conventions
- Write tests for new features
- Document complex logic

### Git Workflow

- Use descriptive commit messages
- Create feature branches
- Review code before merging
- Keep commits atomic
- Use conventional commits

### Performance

- Optimize database queries
- Use caching appropriately
- Minimize bundle size
- Optimize images
- Monitor performance metrics

### Security

- Never commit secrets
- Use environment variables
- Validate all inputs
- Use HTTPS in production
- Keep dependencies updated
