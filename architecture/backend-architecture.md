# Backend Architecture

Django application structure and architecture for the Vitrin platform.

## Overview

The Vitrin backend is built with Django 4.2+ and Django REST Framework, following a modular architecture that supports scalability, maintainability, and testability.

## Technology Stack

### Core Technologies

- **Django 4.2+**: Web framework
- **Django REST Framework**: API framework
- **PostgreSQL**: Primary database
- **Redis**: Caching and session storage
- **MinIO**: Object storage
- **Celery**: Task queue
- **Gunicorn**: WSGI server

### Development Tools

- **pytest**: Testing framework
- **pytest-django**: Django testing utilities
- **factory-boy**: Test data generation
- **coverage**: Code coverage
- **black**: Code formatting
- **flake8**: Code linting

## Project Structure

```
vitrin-backend/
├── vitrin/                 # Django project settings
│   ├── __init__.py
│   ├── settings/           # Environment-specific settings
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── testing.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── apps/                   # Django applications
│   ├── __init__.py
│   ├── users/             # User management
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   ├── admin.py
│   │   ├── managers.py
│   │   ├── permissions.py
│   │   ├── signals.py
│   │   ├── tasks.py
│   │   ├── migrations/
│   │   └── tests/
│   ├── products/          # Product management
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   ├── admin.py
│   │   ├── managers.py
│   │   ├── permissions.py
│   │   ├── signals.py
│   │   ├── tasks.py
│   │   ├── migrations/
│   │   └── tests/
│   └── core/              # Core utilities
│       ├── __init__.py
│       ├── models.py
│       ├── views.py
│       ├── serializers.py
│       ├── urls.py
│       ├── permissions.py
│       ├── pagination.py
│       ├── filters.py
│       ├── exceptions.py
│       ├── middleware.py
│       └── utils.py
├── requirements/          # Dependencies
│   ├── base.txt
│   ├── development.txt
│   ├── production.txt
│   └── testing.txt
├── static/               # Static files
├── media/                # Media files
├── logs/                 # Log files
├── manage.py
├── pytest.ini
├── .env.example
└── README.md
```

## Django Settings

### Base Settings

```python
# settings/base.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

# Security
SECRET_KEY = os.environ.get('SECRET_KEY')
DEBUG = False
ALLOWED_HOSTS = []

# Application definition
DJANGO_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

THIRD_PARTY_APPS = [
    'rest_framework',
    'rest_framework_simplejwt',
    'corsheaders',
    'django_filters',
    'drf_spectacular',
]

LOCAL_APPS = [
    'apps.core',
    'apps.users',
    'apps.products',
]

INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'apps.core.middleware.RequestLoggingMiddleware',
]

ROOT_URLCONF = 'vitrin.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
    }
}

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

### Development Settings

```python
# settings/development.py
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# Database
DATABASES['default'].update({
    'NAME': 'vitrin_dev',
    'USER': 'vitrin',
    'PASSWORD': 'vitrin123',
    'HOST': 'localhost',
    'PORT': '5432',
})

# Email backend
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'root': {
        'handlers': ['console'],
        'level': 'INFO',
    },
}
```

### Production Settings

```python
# settings/production.py
from .base import *

DEBUG = False
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Security
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_SECONDS = 31536000
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

# Database
DATABASES['default'].update({
    'NAME': os.environ.get('DB_NAME'),
    'USER': os.environ.get('DB_USER'),
    'PASSWORD': os.environ.get('DB_PASSWORD'),
    'HOST': os.environ.get('DB_HOST'),
    'PORT': os.environ.get('DB_PORT'),
    'OPTIONS': {
        'sslmode': 'require',
    },
})

# Static files
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.StaticFilesStorage'

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
            'filename': BASE_DIR / 'logs' / 'django.log',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['file'],
        'level': 'INFO',
    },
}
```

## Django REST Framework

### DRF Configuration

```python
# settings/base.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'apps.core.pagination.StandardResultsSetPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',
        'user': '1000/hour'
    }
}

# JWT Settings
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(hours=1),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'UPDATE_LAST_LOGIN': True,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
    'VERIFYING_KEY': None,
    'AUDIENCE': None,
    'ISSUER': 'vitrin-api',
    'AUTH_HEADER_TYPES': ('Bearer',),
    'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',
    'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
    'TOKEN_TYPE_CLAIM': 'token_type',
}

# CORS Settings
CORS_ALLOWED_ORIGINS = [
    "https://vitrin.com",
    "https://www.vitrin.com",
    "https://app.vitrin.com",
]

CORS_ALLOW_CREDENTIALS = True
```

### API Schema

```python
# settings/base.py
SPECTACULAR_SETTINGS = {
    'TITLE': 'Vitrin API',
    'DESCRIPTION': 'API for the Vitrin product showcase platform',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
    'COMPONENT_SPLIT_REQUEST': True,
    'SCHEMA_PATH_PREFIX': '/api/v1/',
}
```

## Models

### Base Model

```python
# apps/core/models.py
from django.db import models
from django.utils import timezone

class BaseModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)

    class Meta:
        abstract = True

    def soft_delete(self):
        self.is_active = False
        self.save()
```

### User Model

```python
# apps/users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models
from apps.core.models import BaseModel

class User(AbstractUser):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=150)
    last_name = models.CharField(max_length=150)
    google_id = models.CharField(max_length=100, unique=True, null=True, blank=True)
    is_verified = models.BooleanField(default=False)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username', 'first_name', 'last_name']

    def __str__(self):
        return self.email

class UserProfile(BaseModel):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    location = models.CharField(max_length=255, blank=True)
    website = models.URLField(blank=True)
    phone = models.CharField(max_length=20, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    gender = models.CharField(
        max_length=10,
        choices=[
            ('male', 'Male'),
            ('female', 'Female'),
            ('other', 'Other'),
            ('prefer_not_to_say', 'Prefer not to say'),
        ],
        blank=True
    )
    is_private = models.BooleanField(default=False)

    def __str__(self):
        return f"{self.user.username}'s profile"
```

### Product Model

```python
# apps/products/models.py
from django.db import models
from django.contrib.auth import get_user_model
from apps.core.models import BaseModel

User = get_user_model()

class Category(BaseModel):
    name = models.CharField(max_length=255)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    parent = models.ForeignKey('self', on_delete=models.SET_NULL, null=True, blank=True)
    image = models.ImageField(upload_to='categories/', blank=True)

    class Meta:
        verbose_name_plural = 'categories'
        ordering = ['name']

    def __str__(self):
        return self.name

class Product(BaseModel):
    name = models.CharField(max_length=255)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    currency = models.CharField(max_length=3, default='USD')
    category = models.ForeignKey(Category, on_delete=models.PROTECT, related_name='products')
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='products')
    views_count = models.PositiveIntegerField(default=0)
    likes_count = models.PositiveIntegerField(default=0)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return self.name

class ProductImage(BaseModel):
    product = models.ForeignKey(Product, on_delete=models.CASCADE, related_name='images')
    image = models.ImageField(upload_to='products/')
    alt_text = models.CharField(max_length=255, blank=True)
    is_primary = models.BooleanField(default=False)
    order_index = models.PositiveIntegerField(default=0)

    class Meta:
        ordering = ['order_index', 'created_at']

    def __str__(self):
        return f"{self.product.name} - Image {self.order_index}"

class ProductTag(BaseModel):
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(unique=True)

    def __str__(self):
        return self.name

class ProductTagRelation(BaseModel):
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    tag = models.ForeignKey(ProductTag, on_delete=models.CASCADE)

    class Meta:
        unique_together = ['product', 'tag']

class Comment(BaseModel):
    product = models.ForeignKey(Product, on_delete=models.CASCADE, related_name='comments')
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='comments')
    parent = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True)
    content = models.TextField()
    is_approved = models.BooleanField(default=True)
    is_edited = models.BooleanField(default=False)
    likes_count = models.PositiveIntegerField(default=0)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return f"{self.user.username} - {self.product.name}"

class ProductLike(BaseModel):
    product = models.ForeignKey(Product, on_delete=models.CASCADE, related_name='likes')
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='product_likes')

    class Meta:
        unique_together = ['product', 'user']

    def __str__(self):
        return f"{self.user.username} likes {self.product.name}"
```

## Views

### ViewSets

```python
# apps/products/views.py
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter
from django.db.models import Q

from .models import Product, Category, Comment, ProductLike
from .serializers import (
    ProductSerializer, CategorySerializer, CommentSerializer,
    ProductCreateSerializer, ProductUpdateSerializer
)
from .permissions import IsOwnerOrReadOnly
from .filters import ProductFilter

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.filter(is_active=True).select_related('user', 'category').prefetch_related('images', 'tags')
    permission_classes = [IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_class = ProductFilter
    search_fields = ['name', 'description']
    ordering_fields = ['name', 'price', 'created_at', 'views_count', 'likes_count']
    ordering = ['-created_at']

    def get_serializer_class(self):
        if self.action == 'create':
            return ProductCreateSerializer
        elif self.action in ['update', 'partial_update']:
            return ProductUpdateSerializer
        return ProductSerializer

    def get_queryset(self):
        queryset = super().get_queryset()
        if self.action == 'list':
            # Add user-specific data for authenticated users
            if self.request.user.is_authenticated:
                queryset = queryset.annotate(
                    is_liked=models.Exists(
                        ProductLike.objects.filter(
                            product=models.OuterRef('pk'),
                            user=self.request.user
                        )
                    )
                )
        return queryset

    def perform_create(self, serializer):
        serializer.save(user=self.request.user)

    @action(detail=True, methods=['post'])
    def like(self, request, pk=None):
        product = self.get_object()
        like, created = ProductLike.objects.get_or_create(
            product=product,
            user=request.user
        )
        
        if created:
            product.likes_count += 1
            product.save()
            return Response({'message': 'Product liked successfully'}, status=status.HTTP_201_CREATED)
        else:
            return Response({'message': 'Product already liked'}, status=status.HTTP_400_BAD_REQUEST)

    @action(detail=True, methods=['delete'])
    def unlike(self, request, pk=None):
        product = self.get_object()
        try:
            like = ProductLike.objects.get(product=product, user=request.user)
            like.delete()
            product.likes_count -= 1
            product.save()
            return Response({'message': 'Product unliked successfully'}, status=status.HTTP_200_OK)
        except ProductLike.DoesNotExist:
            return Response({'message': 'Product not liked'}, status=status.HTTP_400_BAD_REQUEST)

    @action(detail=True, methods=['get'])
    def comments(self, request, pk=None):
        product = self.get_object()
        comments = product.comments.filter(is_approved=True, parent=None)
        serializer = CommentSerializer(comments, many=True, context={'request': request})
        return Response(serializer.data)

class CategoryViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Category.objects.filter(is_active=True)
    serializer_class = CategorySerializer
    lookup_field = 'slug'

    @action(detail=True, methods=['get'])
    def products(self, request, slug=None):
        category = self.get_object()
        products = category.products.filter(is_active=True)
        serializer = ProductSerializer(products, many=True, context={'request': request})
        return Response(serializer.data)
```

## Serializers

### Product Serializers

```python
# apps/products/serializers.py
from rest_framework import serializers
from django.contrib.auth import get_user_model
from .models import Product, Category, Comment, ProductImage, ProductTag

User = get_user_model()

class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = ['id', 'name', 'slug', 'description', 'image']

class ProductImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = ProductImage
        fields = ['id', 'image', 'alt_text', 'is_primary', 'order_index']

class ProductTagSerializer(serializers.ModelSerializer):
    class Meta:
        model = ProductTag
        fields = ['id', 'name', 'slug']

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'first_name', 'last_name']

class ProductSerializer(serializers.ModelSerializer):
    category = CategorySerializer(read_only=True)
    images = ProductImageSerializer(many=True, read_only=True)
    tags = ProductTagSerializer(many=True, read_only=True)
    user = UserSerializer(read_only=True)
    is_liked = serializers.BooleanField(read_only=True)

    class Meta:
        model = Product
        fields = [
            'id', 'name', 'description', 'price', 'currency',
            'category', 'images', 'tags', 'user', 'is_liked',
            'views_count', 'likes_count', 'created_at', 'updated_at'
        ]

class ProductCreateSerializer(serializers.ModelSerializer):
    category_id = serializers.IntegerField(write_only=True)
    tags = serializers.ListField(
        child=serializers.CharField(),
        write_only=True,
        required=False
    )

    class Meta:
        model = Product
        fields = [
            'name', 'description', 'price', 'currency',
            'category_id', 'tags'
        ]

    def create(self, validated_data):
        tags_data = validated_data.pop('tags', [])
        product = Product.objects.create(**validated_data)
        
        # Create tags
        for tag_name in tags_data:
            tag, created = ProductTag.objects.get_or_create(
                name=tag_name,
                defaults={'slug': tag_name.lower().replace(' ', '-')}
            )
            product.tags.add(tag)
        
        return product

class ProductUpdateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ['name', 'description', 'price', 'currency']

class CommentSerializer(serializers.ModelSerializer):
    user = UserSerializer(read_only=True)
    replies = serializers.SerializerMethodField()

    class Meta:
        model = Comment
        fields = [
            'id', 'user', 'content', 'is_edited',
            'likes_count', 'created_at', 'updated_at', 'replies'
        ]

    def get_replies(self, obj):
        replies = obj.replies.filter(is_approved=True)
        return CommentSerializer(replies, many=True, context=self.context).data
```

## Permissions

### Custom Permissions

```python
# apps/products/permissions.py
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """
    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        if request.method in permissions.SAFE_METHODS:
            return True

        # Write permissions are only allowed to the owner of the object.
        return obj.user == request.user

class IsOwnerOrStaff(permissions.BasePermission):
    """
    Custom permission to only allow owners or staff to access an object.
    """
    def has_object_permission(self, request, view, obj):
        # Staff can access everything
        if request.user.is_staff:
            return True
        
        # Owners can access their own objects
        return obj.user == request.user

class IsModeratorOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow moderators to edit objects.
    """
    def has_permission(self, request, view):
        if request.method in permissions.SAFE_METHODS:
            return True
        
        return request.user.is_authenticated and (
            request.user.is_staff or 
            request.user.groups.filter(name='Moderators').exists()
        )
```

## URL Configuration

### Main URLs

```python
# vitrin/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('apps.core.urls')),
    path('api/v1/auth/', include('apps.users.urls')),
    path('api/v1/', include('apps.products.urls')),
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

### App URLs

```python
# apps/products/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ProductViewSet, CategoryViewSet, CommentViewSet

router = DefaultRouter()
router.register(r'products', ProductViewSet)
router.register(r'categories', CategoryViewSet)
router.register(r'comments', CommentViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

## Testing

### Test Configuration

```python
# pytest.ini
[tool:pytest]
DJANGO_SETTINGS_MODULE = vitrin.settings.testing
python_files = tests.py test_*.py *_tests.py
addopts = --tb=short --strict-markers
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
```

### Model Tests

```python
# apps/products/tests/test_models.py
import pytest
from django.test import TestCase
from django.contrib.auth import get_user_model
from factory import Faker
from factory.django import DjangoModelFactory

from apps.products.models import Product, Category

User = get_user_model()

class CategoryFactory(DjangoModelFactory):
    name = Faker('word')
    slug = Faker('slug')
    description = Faker('text')

    class Meta:
        model = Category

class ProductFactory(DjangoModelFactory):
    name = Faker('sentence', nb_words=3)
    description = Faker('text')
    price = Faker('pydecimal', left_digits=4, right_digits=2, positive=True)
    category = factory.SubFactory(CategoryFactory)
    user = factory.SubFactory('apps.users.tests.factories.UserFactory')

    class Meta:
        model = Product

class ProductModelTest(TestCase):
    def setUp(self):
        self.category = CategoryFactory()
        self.user = User.objects.create_user(
            email='test@example.com',
            username='testuser',
            password='testpass123'
        )
        self.product = ProductFactory(
            category=self.category,
            user=self.user
        )

    def test_product_creation(self):
        self.assertEqual(self.product.name, self.product.name)
        self.assertEqual(self.product.user, self.user)
        self.assertEqual(self.product.category, self.category)
        self.assertTrue(self.product.is_active)

    def test_product_str_representation(self):
        self.assertEqual(str(self.product), self.product.name)

    def test_product_soft_delete(self):
        self.product.soft_delete()
        self.assertFalse(self.product.is_active)
```

### View Tests

```python
# apps/products/tests/test_views.py
import pytest
from django.urls import reverse
from rest_framework.test import APITestCase
from rest_framework import status
from django.contrib.auth import get_user_model

from apps.products.models import Product, Category
from apps.products.tests.factories import ProductFactory, CategoryFactory

User = get_user_model()

class ProductViewSetTest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            email='test@example.com',
            username='testuser',
            password='testpass123'
        )
        self.category = CategoryFactory()
        self.product = ProductFactory(user=self.user, category=self.category)

    def test_list_products(self):
        url = reverse('product-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)

    def test_create_product_authenticated(self):
        self.client.force_authenticate(user=self.user)
        url = reverse('product-list')
        data = {
            'name': 'Test Product',
            'description': 'Test Description',
            'price': '99.99',
            'currency': 'USD',
            'category_id': self.category.id,
        }
        response = self.client.post(url, data)
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Product.objects.count(), 2)

    def test_create_product_unauthenticated(self):
        url = reverse('product-list')
        data = {
            'name': 'Test Product',
            'description': 'Test Description',
            'price': '99.99',
            'currency': 'USD',
            'category_id': self.category.id,
        }
        response = self.client.post(url, data)
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)

    def test_like_product(self):
        self.client.force_authenticate(user=self.user)
        url = reverse('product-like', kwargs={'pk': self.product.pk})
        response = self.client.post(url)
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.product.refresh_from_db()
        self.assertEqual(self.product.likes_count, 1)
```

## Deployment

### Docker Configuration

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' vitrin
USER vitrin

# Expose port
EXPOSE 8000

# Run application
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "vitrin.wsgi:application"]
```

### Gunicorn Configuration

```python
# gunicorn.conf.py
bind = "0.0.0.0:8000"
workers = 4
worker_class = "sync"
worker_connections = 1000
timeout = 30
keepalive = 2
max_requests = 1000
max_requests_jitter = 100
preload_app = True
```

### Environment Variables

```bash
# .env.production
SECRET_KEY=your-secret-key
DEBUG=False
ALLOWED_HOSTS=api.vitrin.com,www.vitrin.com
DB_NAME=vitrin_prod
DB_USER=vitrin
DB_PASSWORD=secure-password
DB_HOST=localhost
DB_PORT=5432
REDIS_URL=redis://localhost:6379/0
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=access-key
MINIO_SECRET_KEY=secret-key
MINIO_BUCKET_NAME=vitrin-prod
```
