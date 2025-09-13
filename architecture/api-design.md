# API Design

API architecture and design patterns for the Vitrin platform.

## Overview

The Vitrin API follows RESTful principles with a focus on simplicity, consistency, and developer experience. The API is built using Django REST Framework and provides comprehensive endpoints for all platform functionality.

## Design Principles

### RESTful Architecture

- **Resource-based URLs**: URLs represent resources, not actions
- **HTTP methods**: Use appropriate HTTP methods (GET, POST, PUT, DELETE)
- **Stateless**: Each request contains all necessary information
- **Cacheable**: Responses can be cached when appropriate
- **Uniform interface**: Consistent API design across all endpoints

### API Design Standards

- **Consistent naming**: Use consistent naming conventions
- **Versioning**: API versioning for backward compatibility
- **Error handling**: Comprehensive error responses
- **Documentation**: Clear and comprehensive documentation
- **Rate limiting**: Protect against abuse and ensure fair usage

## URL Structure

### Base URL

```
https://api.vitrin.com/v1/
```

### URL Patterns

```
# Resource collections
GET    /api/v1/products/           # List products
POST   /api/v1/products/           # Create product

# Individual resources
GET    /api/v1/products/{id}/      # Get product
PUT    /api/v1/products/{id}/      # Update product
DELETE /api/v1/products/{id}/      # Delete product

# Nested resources
GET    /api/v1/products/{id}/comments/     # Get product comments
POST   /api/v1/products/{id}/comments/     # Create comment
GET    /api/v1/products/{id}/images/       # Get product images
POST   /api/v1/products/{id}/images/       # Upload image

# Actions
POST   /api/v1/products/{id}/like/         # Like product
DELETE /api/v1/products/{id}/like/         # Unlike product
POST   /api/v1/comments/{id}/like/         # Like comment
DELETE /api/v1/comments/{id}/like/         # Unlike comment
```

## HTTP Methods

### GET

Used for retrieving resources.

```http
GET /api/v1/products/
GET /api/v1/products/123/
GET /api/v1/users/profile/
```

### POST

Used for creating new resources.

```http
POST /api/v1/products/
POST /api/v1/auth/login/
POST /api/v1/products/123/comments/
```

### PUT

Used for updating entire resources.

```http
PUT /api/v1/products/123/
PUT /api/v1/users/profile/
```

### PATCH

Used for partial updates.

```http
PATCH /api/v1/products/123/
PATCH /api/v1/users/profile/
```

### DELETE

Used for deleting resources.

```http
DELETE /api/v1/products/123/
DELETE /api/v1/comments/456/
```

## Request/Response Format

### Request Headers

```http
Content-Type: application/json
Authorization: Bearer <access_token>
Accept: application/json
User-Agent: VitrinApp/1.0
```

### Response Headers

```http
Content-Type: application/json
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### JSON Response Format

```json
{
  "data": {
    "id": 1,
    "name": "iPhone 15 Pro",
    "price": 999.99,
    "created_at": "2024-01-15T10:30:00Z"
  },
  "meta": {
    "count": 1,
    "next": null,
    "previous": null
  }
}
```

## Authentication

### JWT Authentication

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
```

### Token Refresh

```http
POST /api/v1/auth/refresh/
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

### Google OAuth

```http
POST /api/v1/auth/google/
{
  "access_token": "google_access_token_here"
}
```

## Pagination

### Cursor-based Pagination

```json
{
  "results": [...],
  "next": "http://api.vitrin.com/v1/products/?cursor=eyJpZCI6MTIzfQ",
  "previous": "http://api.vitrin.com/v1/products/?cursor=eyJpZCI6MTIwfQ"
}
```

### Page-based Pagination

```json
{
  "count": 150,
  "next": "http://api.vitrin.com/v1/products/?page=2",
  "previous": null,
  "results": [...]
}
```

### Pagination Parameters

- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20, max: 100)
- `cursor`: Cursor for cursor-based pagination

## Filtering and Sorting

### Filtering

```http
GET /api/v1/products/?category=1&min_price=100&max_price=500&tags=electronics
```

### Sorting

```http
GET /api/v1/products/?ordering=-created_at
GET /api/v1/products/?ordering=name,-price
```

### Search

```http
GET /api/v1/search/?q=iphone&type=products
```

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request data is invalid",
    "details": {
      "name": ["This field is required."],
      "price": ["Price must be greater than 0."]
    }
  }
}
```

### HTTP Status Codes

- **200 OK**: Successful request
- **201 Created**: Resource created successfully
- **400 Bad Request**: Invalid request data
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource conflict
- **429 Too Many Requests**: Rate limit exceeded
- **500 Internal Server Error**: Server error

### Error Codes

- `VALIDATION_ERROR`: Request validation failed
- `AUTHENTICATION_REQUIRED`: Authentication required
- `ACCESS_DENIED`: Insufficient permissions
- `RESOURCE_NOT_FOUND`: Resource does not exist
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `INTERNAL_ERROR`: Internal server error

## Rate Limiting

### Rate Limit Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### Rate Limits

- **Authenticated users**: 1000 requests per hour
- **Unauthenticated users**: 100 requests per hour
- **Write operations**: 100 requests per hour
- **Search requests**: 500 requests per hour

## API Versioning

### URL Versioning

```
https://api.vitrin.com/v1/products/
https://api.vitrin.com/v2/products/
```

### Header Versioning

```http
Accept: application/vnd.vitrin.v1+json
```

### Version Deprecation

```http
X-API-Deprecation-Date: 2024-12-31
X-API-Deprecation-Info: https://docs.vitrin.com/api/deprecation
```

## Caching

### Cache Headers

```http
Cache-Control: public, max-age=3600
ETag: "abc123"
Last-Modified: Wed, 15 Jan 2024 10:30:00 GMT
```

### Cache Strategies

- **Public resources**: Cache for 1 hour
- **User-specific resources**: Cache for 5 minutes
- **Dynamic content**: No caching
- **Static assets**: Cache for 1 year

## Webhooks

### Webhook Events

```json
{
  "event": "product.created",
  "data": {
    "id": 123,
    "name": "iPhone 15 Pro",
    "user_id": 456
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### Supported Events

- `product.created`
- `product.updated`
- `product.deleted`
- `user.registered`
- `comment.created`
- `comment.updated`

## API Documentation

### OpenAPI Specification

The API is documented using OpenAPI 3.0 specification:

```yaml
openapi: 3.0.0
info:
  title: Vitrin API
  version: 1.0.0
  description: API for the Vitrin product showcase platform
servers:
  - url: https://api.vitrin.com/v1
paths:
  /products:
    get:
      summary: List products
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  results:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
```

### Interactive Documentation

- **Swagger UI**: Available at `/api/docs/`
- **ReDoc**: Available at `/api/redoc/`
- **Postman Collection**: Available for download

## SDK and Libraries

### Official SDKs

- **JavaScript/TypeScript**: `@vitrin/api-client`
- **Python**: `vitrin-api-client`
- **PHP**: `vitrin/api-client`
- **Java**: `com.vitrin.api-client`

### Usage Example

```javascript
import { VitrinAPI } from '@vitrin/api-client';

const api = new VitrinAPI({
  baseURL: 'https://api.vitrin.com/v1',
  apiKey: 'your-api-key'
});

// Get products
const products = await api.products.list({
  page: 1,
  pageSize: 20
});

// Create product
const product = await api.products.create({
  name: 'iPhone 15 Pro',
  price: 999.99,
  category: 1
});
```

## Testing

### API Testing

- **Unit tests**: Test individual endpoints
- **Integration tests**: Test complete workflows
- **Load tests**: Test performance under load
- **Contract tests**: Test API contracts

### Test Data

- **Fixtures**: Predefined test data
- **Factories**: Dynamic test data generation
- **Mocking**: Mock external dependencies

## Monitoring and Analytics

### Metrics

- **Request count**: Total API requests
- **Response time**: Average response time
- **Error rate**: Percentage of failed requests
- **Usage by endpoint**: Most used endpoints

### Logging

- **Request logging**: Log all API requests
- **Error logging**: Log errors and exceptions
- **Performance logging**: Log slow queries
- **Audit logging**: Log sensitive operations

### Alerts

- **High error rate**: Alert when error rate exceeds threshold
- **Slow responses**: Alert when response time is high
- **Rate limit exceeded**: Alert when rate limits are hit
- **Unusual traffic**: Alert for unusual traffic patterns
