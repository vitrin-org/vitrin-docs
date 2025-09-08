# API Overview

The Vitrin API is a RESTful API built with Django REST Framework that provides endpoints for managing products, users, categories, and more.

## 🔗 **Base URL**

```
Development: http://localhost:8000/api/
Production: https://api.vitrin.app/api/
```

## 🔐 **Authentication**

The API uses JWT (JSON Web Tokens) for authentication. Include the token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

### Authentication Endpoints

- `POST /auth/login/` - Login with email and password
- `POST /auth/register/` - Register a new user
- `POST /auth/google/` - Login with Google OAuth
- `POST /auth/refresh/` - Refresh JWT token
- `POST /auth/logout/` - Logout and invalidate token

## 📊 **Response Format**

All API responses follow a consistent format:

### Success Response
```json
{
  "data": {
    // Response data
  },
  "message": "Success message",
  "status": 200,
  "success": true
}
```

### Error Response
```json
{
  "error": {
    "message": "Error description",
    "code": "ERROR_CODE",
    "details": {
      // Additional error details
    }
  },
  "status": 400,
  "success": false
}
```

### Paginated Response
```json
{
  "data": {
    "count": 100,
    "next": "http://api.vitrin.app/api/products/?page=2",
    "previous": null,
    "results": [
      // Array of items
    ]
  },
  "status": 200,
  "success": true
}
```

## 📝 **HTTP Status Codes**

| Code | Description |
|------|-------------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 204 | No Content - Request successful, no content returned |
| 400 | Bad Request - Invalid request data |
| 401 | Unauthorized - Authentication required |
| 403 | Forbidden - Access denied |
| 404 | Not Found - Resource not found |
| 422 | Unprocessable Entity - Validation error |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Server error |

## 🔍 **Query Parameters**

### Pagination
- `page` - Page number (default: 1)
- `page_size` - Items per page (default: 20, max: 100)

### Filtering
- `search` - Search query
- `category` - Filter by category ID
- `status` - Filter by status
- `featured` - Filter featured items (true/false)
- `author` - Filter by author ID
- `tags` - Filter by tags (comma-separated)

### Sorting
- `ordering` - Sort field (prefix with `-` for descending)

Examples:
```
GET /api/products/?page=2&page_size=10
GET /api/products/?search=react&category=1
GET /api/products/?ordering=-created_at
GET /api/products/?tags=javascript,react
```

## 📁 **Endpoints Overview**

### Authentication
- `POST /auth/login/` - User login
- `POST /auth/register/` - User registration
- `POST /auth/google/` - Google OAuth login
- `POST /auth/refresh/` - Refresh token
- `POST /auth/logout/` - User logout

### Users
- `GET /users/` - List users
- `GET /users/{id}/` - Get user details
- `GET /users/profile/` - Get current user profile
- `PUT /users/profile/` - Update user profile
- `POST /users/upload-avatar/` - Upload user avatar

### Products
- `GET /products/` - List products
- `POST /products/` - Create product
- `GET /products/{id}/` - Get product details
- `PUT /products/{id}/` - Update product
- `DELETE /products/{id}/` - Delete product
- `POST /products/{id}/upvote/` - Upvote product
- `DELETE /products/{id}/upvote/` - Remove upvote
- `POST /products/{id}/view/` - Record product view

### Categories
- `GET /categories/` - List categories
- `POST /categories/` - Create category
- `GET /categories/{id}/` - Get category details
- `PUT /categories/{id}/` - Update category
- `DELETE /categories/{id}/` - Delete category

### Comments
- `GET /products/{id}/comments/` - List product comments
- `POST /products/{id}/comments/` - Create comment
- `PUT /comments/{id}/` - Update comment
- `DELETE /comments/{id}/` - Delete comment
- `POST /comments/{id}/upvote/` - Upvote comment

### Search
- `GET /search/products/` - Search products
- `GET /search/users/` - Search users
- `GET /search/suggestions/` - Get search suggestions

### File Upload
- `POST /upload/image/` - Upload image
- `POST /upload/avatar/` - Upload avatar
- `POST /upload/gallery/` - Upload gallery images

## 🔒 **Rate Limiting**

The API implements rate limiting to prevent abuse:

- **Authentication endpoints**: 5 requests per minute
- **General endpoints**: 100 requests per minute
- **File upload endpoints**: 10 requests per minute

Rate limit headers are included in responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

## 📋 **Request Examples**

### Login
```bash
curl -X POST http://localhost:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

### Create Product
```bash
curl -X POST http://localhost:8000/api/products/ \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My Awesome Product",
    "short_description": "A brief description",
    "long_description": "A detailed description",
    "category": 1,
    "tags": ["react", "javascript"]
  }'
```

### Upload Image
```bash
curl -X POST http://localhost:8000/api/upload/image/ \
  -H "Authorization: Bearer <token>" \
  -F "file=@image.jpg" \
  -F "type=product_image"
```

## 🧪 **Testing the API**

### Using curl
```bash
# Test health endpoint
curl http://localhost:8000/api/health/

# Test authentication
curl -X POST http://localhost:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password"}'
```

### Using Postman
1. Import the API collection from `/api/postman/`
2. Set the base URL to your API endpoint
3. Configure authentication in the collection settings

### Using the API Documentation
Visit http://localhost:8000/api/docs/ for interactive API documentation.

## 📚 **SDK and Libraries**

### JavaScript/TypeScript
```typescript
import { VitrinAPI } from '@vitrin/api-client';

const api = new VitrinAPI({
  baseURL: 'http://localhost:8000/api',
  token: 'your-jwt-token'
});

// Get products
const products = await api.products.list();

// Create product
const product = await api.products.create({
  title: 'My Product',
  description: 'Product description',
  category: 1
});
```

### Python
```python
import requests

# Set up session
session = requests.Session()
session.headers.update({
    'Authorization': 'Bearer your-jwt-token',
    'Content-Type': 'application/json'
})

# Get products
response = session.get('http://localhost:8000/api/products/')
products = response.json()

# Create product
product_data = {
    'title': 'My Product',
    'description': 'Product description',
    'category': 1
}
response = session.post('http://localhost:8000/api/products/', json=product_data)
```

## 🔄 **Webhooks**

The API supports webhooks for real-time notifications:

### Available Events
- `product.created` - New product created
- `product.updated` - Product updated
- `product.deleted` - Product deleted
- `user.registered` - New user registered
- `comment.created` - New comment added

### Webhook Configuration
```json
{
  "url": "https://your-app.com/webhooks/vitrin",
  "events": ["product.created", "product.updated"],
  "secret": "your-webhook-secret"
}
```

## 📖 **Additional Resources**

- [Authentication Guide](authentication.md) - Detailed authentication documentation
- [Products API](products.md) - Product management endpoints
- [Users API](users.md) - User management endpoints
- [Error Handling](error-handling.md) - Error codes and handling
- [Rate Limiting](rate-limiting.md) - Rate limiting details

## 🆘 **Support**

- **Documentation**: [API Docs](https://docs.vitrin.app/api)
- **Issues**: [GitHub Issues](https://github.com/your-org/vitrin-backend/issues)
- **Community**: [Discord](https://discord.gg/vitrin)
- **Email**: api-support@vitrin.app
