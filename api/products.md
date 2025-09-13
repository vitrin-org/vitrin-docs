# Products API

Product management endpoints for the Vitrin platform.

## Overview

The Products API provides comprehensive CRUD operations for managing products, including product information, images, categories, and user interactions.

## Product Model

```json
{
  "id": 1,
  "name": "iPhone 15 Pro",
  "description": "Latest iPhone with advanced camera system",
  "price": 999.99,
  "currency": "USD",
  "category": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics"
  },
  "images": [
    {
      "id": 1,
      "url": "https://example.com/image1.jpg",
      "alt_text": "iPhone 15 Pro front view",
      "is_primary": true
    }
  ],
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "is_active": true,
  "tags": ["smartphone", "apple", "premium"]
}
```

## Endpoints

### List Products

#### Get All Products
```http
GET /api/products/
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20, max: 100)
- `category` (int): Filter by category ID
- `user` (int): Filter by user ID
- `search` (string): Search in product name and description
- `min_price` (float): Minimum price filter
- `max_price` (float): Maximum price filter
- `tags` (string): Filter by tags (comma-separated)
- `ordering` (string): Sort order (name, price, created_at, -name, -price, -created_at)

**Example:**
```http
GET /api/products/?page=1&page_size=20&category=1&search=iphone&min_price=500&max_price=1500&ordering=-created_at
```

**Response:**
```json
{
  "count": 150,
  "next": "http://api.example.com/api/products/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "iPhone 15 Pro",
      "description": "Latest iPhone with advanced camera system",
      "price": 999.99,
      "currency": "USD",
      "category": {
        "id": 1,
        "name": "Electronics",
        "slug": "electronics"
      },
      "images": [
        {
          "id": 1,
          "url": "https://example.com/image1.jpg",
          "alt_text": "iPhone 15 Pro front view",
          "is_primary": true
        }
      ],
      "user": {
        "id": 1,
        "username": "john_doe",
        "first_name": "John",
        "last_name": "Doe"
      },
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z",
      "is_active": true,
      "tags": ["smartphone", "apple", "premium"]
    }
  ]
}
```

### Get Product Details

#### Get Single Product
```http
GET /api/products/{id}/
```

**Response:**
```json
{
  "id": 1,
  "name": "iPhone 15 Pro",
  "description": "Latest iPhone with advanced camera system",
  "price": 999.99,
  "currency": "USD",
  "category": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics"
  },
  "images": [
    {
      "id": 1,
      "url": "https://example.com/image1.jpg",
      "alt_text": "iPhone 15 Pro front view",
      "is_primary": true
    },
    {
      "id": 2,
      "url": "https://example.com/image2.jpg",
      "alt_text": "iPhone 15 Pro back view",
      "is_primary": false
    }
  ],
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "is_active": true,
  "tags": ["smartphone", "apple", "premium"],
  "views_count": 1250,
  "likes_count": 45
}
```

### Create Product

#### Create New Product
```http
POST /api/products/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "name": "MacBook Pro M3",
  "description": "Powerful laptop for professionals",
  "price": 1999.99,
  "currency": "USD",
  "category": 1,
  "tags": ["laptop", "apple", "professional"]
}
```

**Response:**
```json
{
  "id": 2,
  "name": "MacBook Pro M3",
  "description": "Powerful laptop for professionals",
  "price": 1999.99,
  "currency": "USD",
  "category": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics"
  },
  "images": [],
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe"
  },
  "created_at": "2024-01-15T11:00:00Z",
  "updated_at": "2024-01-15T11:00:00Z",
  "is_active": true,
  "tags": ["laptop", "apple", "professional"]
}
```

### Update Product

#### Update Existing Product
```http
PUT /api/products/{id}/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "name": "MacBook Pro M3 (Updated)",
  "description": "Powerful laptop for professionals - Updated description",
  "price": 1899.99,
  "currency": "USD",
  "category": 1,
  "tags": ["laptop", "apple", "professional", "updated"]
}
```

**Response:**
```json
{
  "id": 2,
  "name": "MacBook Pro M3 (Updated)",
  "description": "Powerful laptop for professionals - Updated description",
  "price": 1899.99,
  "currency": "USD",
  "category": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics"
  },
  "images": [],
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe"
  },
  "created_at": "2024-01-15T11:00:00Z",
  "updated_at": "2024-01-15T11:30:00Z",
  "is_active": true,
  "tags": ["laptop", "apple", "professional", "updated"]
}
```

### Delete Product

#### Delete Product
```http
DELETE /api/products/{id}/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Product deleted successfully"
}
```

## Image Management

### Upload Product Image
```http
POST /api/products/{id}/images/
Authorization: Bearer <access_token>
Content-Type: multipart/form-data

{
  "image": <file>,
  "alt_text": "Product image description",
  "is_primary": false
}
```

**Response:**
```json
{
  "id": 3,
  "url": "https://example.com/image3.jpg",
  "alt_text": "Product image description",
  "is_primary": false,
  "created_at": "2024-01-15T12:00:00Z"
}
```

### Delete Product Image
```http
DELETE /api/products/{id}/images/{image_id}/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Image deleted successfully"
}
```

## Product Interactions

### Like Product
```http
POST /api/products/{id}/like/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Product liked successfully",
  "likes_count": 46
}
```

### Unlike Product
```http
DELETE /api/products/{id}/like/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Product unliked successfully",
  "likes_count": 45
}
```

## Error Responses

### 400 Bad Request
```json
{
  "error": "Validation error",
  "details": {
    "name": ["This field is required."],
    "price": ["Price must be greater than 0."]
  }
}
```

### 401 Unauthorized
```json
{
  "error": "Authentication required",
  "details": "Token is missing or invalid"
}
```

### 403 Forbidden
```json
{
  "error": "Access denied",
  "details": "You can only modify your own products"
}
```

### 404 Not Found
```json
{
  "error": "Product not found",
  "details": "The requested product does not exist"
}
```

## Rate Limiting

- **Read operations**: 1000 requests per hour
- **Write operations**: 100 requests per hour
- **Image uploads**: 50 requests per hour

## Implementation Notes

- All prices are stored as decimal values with 2 decimal places
- Images are stored in MinIO object storage
- Product search uses full-text search on name and description
- Pagination is implemented using Django REST framework pagination
- All timestamps are in ISO 8601 format (UTC)
- Product views are tracked for analytics
