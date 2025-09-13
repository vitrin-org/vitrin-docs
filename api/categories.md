# Categories API

Category management endpoints for the Vitrin platform.

## Overview

The Categories API provides endpoints for managing product categories, including hierarchical category structures and category-specific product listings.

## Category Model

```json
{
  "id": 1,
  "name": "Electronics",
  "slug": "electronics",
  "description": "Electronic devices and gadgets",
  "parent": null,
  "children": [
    {
      "id": 2,
      "name": "Smartphones",
      "slug": "smartphones",
      "description": "Mobile phones and accessories",
      "parent": 1,
      "children": []
    }
  ],
  "image": "https://example.com/category-image.jpg",
  "is_active": true,
  "products_count": 150,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

## Endpoints

### List Categories

#### Get All Categories
```http
GET /api/categories/
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20, max: 100)
- `parent` (int): Filter by parent category ID
- `search` (string): Search in category name and description
- `is_active` (boolean): Filter by active status
- `ordering` (string): Sort order (name, created_at, products_count, -name, -created_at, -products_count)

**Example:**
```http
GET /api/categories/?page=1&page_size=20&parent=1&search=electronics&is_active=true&ordering=-products_count
```

**Response:**
```json
{
  "count": 25,
  "next": "http://api.example.com/api/categories/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Electronics",
      "slug": "electronics",
      "description": "Electronic devices and gadgets",
      "parent": null,
      "children": [
        {
          "id": 2,
          "name": "Smartphones",
          "slug": "smartphones",
          "description": "Mobile phones and accessories",
          "parent": 1,
          "children": []
        }
      ],
      "image": "https://example.com/category-image.jpg",
      "is_active": true,
      "products_count": 150,
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

#### Get Category Tree
```http
GET /api/categories/tree/
```

**Response:**
```json
{
  "categories": [
    {
      "id": 1,
      "name": "Electronics",
      "slug": "electronics",
      "description": "Electronic devices and gadgets",
      "parent": null,
      "children": [
        {
          "id": 2,
          "name": "Smartphones",
          "slug": "smartphones",
          "description": "Mobile phones and accessories",
          "parent": 1,
          "children": [
            {
              "id": 3,
              "name": "iPhone",
              "slug": "iphone",
              "description": "Apple iPhone devices",
              "parent": 2,
              "children": []
            }
          ]
        },
        {
          "id": 4,
          "name": "Laptops",
          "slug": "laptops",
          "description": "Portable computers",
          "parent": 1,
          "children": []
        }
      ],
      "image": "https://example.com/electronics.jpg",
      "is_active": true,
      "products_count": 150
    },
    {
      "id": 5,
      "name": "Fashion",
      "slug": "fashion",
      "description": "Clothing and accessories",
      "parent": null,
      "children": [
        {
          "id": 6,
          "name": "Men's Clothing",
          "slug": "mens-clothing",
          "description": "Clothing for men",
          "parent": 5,
          "children": []
        }
      ],
      "image": "https://example.com/fashion.jpg",
      "is_active": true,
      "products_count": 89
    }
  ]
}
```

### Get Category Details

#### Get Single Category
```http
GET /api/categories/{id}/
```

**Response:**
```json
{
  "id": 1,
  "name": "Electronics",
  "slug": "electronics",
  "description": "Electronic devices and gadgets",
  "parent": null,
  "children": [
    {
      "id": 2,
      "name": "Smartphones",
      "slug": "smartphones",
      "description": "Mobile phones and accessories",
      "parent": 1,
      "children": []
    }
  ],
  "image": "https://example.com/category-image.jpg",
  "is_active": true,
  "products_count": 150,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

#### Get Category by Slug
```http
GET /api/categories/slug/{slug}/
```

**Response:**
```json
{
  "id": 1,
  "name": "Electronics",
  "slug": "electronics",
  "description": "Electronic devices and gadgets",
  "parent": null,
  "children": [
    {
      "id": 2,
      "name": "Smartphones",
      "slug": "smartphones",
      "description": "Mobile phones and accessories",
      "parent": 1,
      "children": []
    }
  ],
  "image": "https://example.com/category-image.jpg",
  "is_active": true,
  "products_count": 150,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### Create Category

#### Create New Category
```http
POST /api/categories/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "name": "Gaming",
  "slug": "gaming",
  "description": "Gaming devices and accessories",
  "parent": 1,
  "image": "https://example.com/gaming.jpg"
}
```

**Response:**
```json
{
  "id": 7,
  "name": "Gaming",
  "slug": "gaming",
  "description": "Gaming devices and accessories",
  "parent": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics"
  },
  "children": [],
  "image": "https://example.com/gaming.jpg",
  "is_active": true,
  "products_count": 0,
  "created_at": "2024-01-20T15:30:00Z",
  "updated_at": "2024-01-20T15:30:00Z"
}
```

### Update Category

#### Update Existing Category
```http
PUT /api/categories/{id}/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "name": "Gaming & Entertainment",
  "slug": "gaming-entertainment",
  "description": "Gaming devices, consoles, and entertainment accessories",
  "parent": 1,
  "image": "https://example.com/gaming-entertainment.jpg"
}
```

**Response:**
```json
{
  "id": 7,
  "name": "Gaming & Entertainment",
  "slug": "gaming-entertainment",
  "description": "Gaming devices, consoles, and entertainment accessories",
  "parent": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics"
  },
  "children": [],
  "image": "https://example.com/gaming-entertainment.jpg",
  "is_active": true,
  "products_count": 0,
  "created_at": "2024-01-20T15:30:00Z",
  "updated_at": "2024-01-20T16:00:00Z"
}
```

### Delete Category

#### Delete Category
```http
DELETE /api/categories/{id}/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Category deleted successfully"
}
```

### Category Products

#### Get Products in Category
```http
GET /api/categories/{id}/products/
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `include_subcategories` (boolean): Include products from subcategories (default: false)
- `search` (string): Search in product name and description
- `min_price` (float): Minimum price filter
- `max_price` (float): Maximum price filter
- `ordering` (string): Sort order (name, price, created_at, -name, -price, -created_at)

**Example:**
```http
GET /api/categories/1/products/?page=1&page_size=20&include_subcategories=true&search=iphone&min_price=500&ordering=-created_at
```

**Response:**
```json
{
  "category": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics",
    "description": "Electronic devices and gadgets"
  },
  "count": 150,
  "next": "http://api.example.com/api/categories/1/products/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "iPhone 15 Pro",
      "description": "Latest iPhone with advanced camera system",
      "price": 999.99,
      "currency": "USD",
      "category": {
        "id": 2,
        "name": "Smartphones",
        "slug": "smartphones"
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
      "tags": ["smartphone", "apple", "premium"],
      "views_count": 1250,
      "likes_count": 45
    }
  ]
}
```

### Category Statistics

#### Get Category Statistics
```http
GET /api/categories/{id}/stats/
```

**Response:**
```json
{
  "category": {
    "id": 1,
    "name": "Electronics",
    "slug": "electronics"
  },
  "stats": {
    "total_products": 150,
    "active_products": 145,
    "total_subcategories": 5,
    "average_price": 299.99,
    "price_range": {
      "min": 19.99,
      "max": 1999.99
    },
    "recent_products": 12,
    "popular_tags": [
      {"tag": "smartphone", "count": 45},
      {"tag": "laptop", "count": 32},
      {"tag": "tablet", "count": 28}
    ]
  }
}
```

## Error Responses

### 400 Bad Request
```json
{
  "error": "Validation error",
  "details": {
    "name": ["This field is required."],
    "slug": ["This field is required."],
    "parent": ["Cannot set parent to self or descendant."]
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
  "details": "Only staff members can manage categories"
}
```

### 404 Not Found
```json
{
  "error": "Category not found",
  "details": "The requested category does not exist"
}
```

### 409 Conflict
```json
{
  "error": "Category conflict",
  "details": "Cannot delete category with products or subcategories"
}
```

## Rate Limiting

- **Read operations**: 1000 requests per hour
- **Write operations**: 100 requests per hour
- **Category tree**: 500 requests per hour

## Implementation Notes

- Categories support hierarchical structure with unlimited depth
- Slug generation is automatic from the name if not provided
- Category deletion is prevented if it has products or subcategories
- Product counts are calculated in real-time
- Category images are stored in MinIO object storage
- All timestamps are in ISO 8601 format (UTC)
- Category tree endpoint provides complete hierarchy in one request
