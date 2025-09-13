# Search API

Search and filtering endpoints for the Vitrin platform.

## Overview

The Search API provides comprehensive search functionality across products, users, and categories with advanced filtering, sorting, and faceted search capabilities.

## Search Endpoints

### Global Search

#### Search All Content
```http
GET /api/search/
```

**Query Parameters:**
- `q` (string): Search query (required)
- `type` (string): Content type filter (products, users, categories, all)
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20, max: 100)
- `ordering` (string): Sort order (relevance, created_at, -created_at, name, -name)

**Example:**
```http
GET /api/search/?q=iphone&type=products&page=1&page_size=20&ordering=relevance
```

**Response:**
```json
{
  "query": "iphone",
  "type": "products",
  "count": 25,
  "next": "http://api.example.com/api/search/?q=iphone&type=products&page=2",
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
      "tags": ["smartphone", "apple", "premium"],
      "views_count": 1250,
      "likes_count": 45,
      "relevance_score": 0.95
    }
  ],
  "facets": {
    "categories": [
      {"id": 1, "name": "Electronics", "count": 15},
      {"id": 2, "name": "Smartphones", "count": 10}
    ],
    "price_ranges": [
      {"min": 0, "max": 500, "count": 5},
      {"min": 500, "max": 1000, "count": 15},
      {"min": 1000, "max": 2000, "count": 5}
    ],
    "tags": [
      {"tag": "smartphone", "count": 10},
      {"tag": "apple", "count": 8},
      {"tag": "premium", "count": 6}
    ]
  }
}
```

### Product Search

#### Search Products
```http
GET /api/search/products/
```

**Query Parameters:**
- `q` (string): Search query
- `category` (int): Filter by category ID
- `user` (int): Filter by user ID
- `min_price` (float): Minimum price filter
- `max_price` (float): Maximum price filter
- `tags` (string): Filter by tags (comma-separated)
- `is_active` (boolean): Filter by active status
- `created_after` (date): Filter by creation date (ISO 8601)
- `created_before` (date): Filter by creation date (ISO 8601)
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `ordering` (string): Sort order (relevance, name, price, created_at, -name, -price, -created_at)

**Example:**
```http
GET /api/search/products/?q=smartphone&category=1&min_price=500&max_price=1500&tags=apple,premium&ordering=relevance
```

**Response:**
```json
{
  "query": "smartphone",
  "filters": {
    "category": 1,
    "min_price": 500,
    "max_price": 1500,
    "tags": ["apple", "premium"]
  },
  "count": 15,
  "next": "http://api.example.com/api/search/products/?q=smartphone&category=1&page=2",
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
      "tags": ["smartphone", "apple", "premium"],
      "views_count": 1250,
      "likes_count": 45,
      "relevance_score": 0.95
    }
  ],
  "facets": {
    "categories": [
      {"id": 1, "name": "Electronics", "count": 15}
    ],
    "price_ranges": [
      {"min": 500, "max": 1000, "count": 10},
      {"min": 1000, "max": 1500, "count": 5}
    ],
    "tags": [
      {"tag": "smartphone", "count": 15},
      {"tag": "apple", "count": 8},
      {"tag": "premium", "count": 6}
    ]
  }
}
```

### User Search

#### Search Users
```http
GET /api/search/users/
```

**Query Parameters:**
- `q` (string): Search query
- `location` (string): Filter by location
- `is_active` (boolean): Filter by active status
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `ordering` (string): Sort order (relevance, username, created_at, -username, -created_at)

**Example:**
```http
GET /api/search/users/?q=john&location=San Francisco&ordering=relevance
```

**Response:**
```json
{
  "query": "john",
  "filters": {
    "location": "San Francisco"
  },
  "count": 8,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "username": "john_doe",
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "is_active": true,
      "date_joined": "2024-01-15T10:30:00Z",
      "profile": {
        "bio": "Product enthusiast and tech lover",
        "avatar": "https://example.com/avatar.jpg",
        "location": "San Francisco, CA",
        "website": "https://johndoe.com"
      },
      "stats": {
        "products_count": 25,
        "followers_count": 150,
        "following_count": 75,
        "likes_received": 320
      },
      "relevance_score": 0.98
    }
  ]
}
```

### Category Search

#### Search Categories
```http
GET /api/search/categories/
```

**Query Parameters:**
- `q` (string): Search query
- `parent` (int): Filter by parent category ID
- `is_active` (boolean): Filter by active status
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `ordering` (string): Sort order (relevance, name, products_count, -name, -products_count)

**Example:**
```http
GET /api/search/categories/?q=electronics&ordering=relevance
```

**Response:**
```json
{
  "query": "electronics",
  "filters": {},
  "count": 5,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Electronics",
      "slug": "electronics",
      "description": "Electronic devices and gadgets",
      "parent": null,
      "image": "https://example.com/category-image.jpg",
      "is_active": true,
      "products_count": 150,
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z",
      "relevance_score": 0.99
    }
  ]
}
```

### Advanced Search

#### Advanced Product Search
```http
GET /api/search/advanced/
```

**Query Parameters:**
- `q` (string): Search query
- `fields` (string): Search in specific fields (name, description, tags, all)
- `category` (int): Filter by category ID
- `user` (int): Filter by user ID
- `min_price` (float): Minimum price filter
- `max_price` (float): Maximum price filter
- `tags` (string): Filter by tags (comma-separated)
- `created_after` (date): Filter by creation date (ISO 8601)
- `created_before` (date): Filter by creation date (ISO 8601)
- `min_views` (int): Minimum views filter
- `min_likes` (int): Minimum likes filter
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `ordering` (string): Sort order (relevance, name, price, created_at, views_count, likes_count, -name, -price, -created_at, -views_count, -likes_count)

**Example:**
```http
GET /api/search/advanced/?q=smartphone&fields=name,description&category=1&min_price=500&min_views=100&ordering=-likes_count
```

**Response:**
```json
{
  "query": "smartphone",
  "fields": ["name", "description"],
  "filters": {
    "category": 1,
    "min_price": 500,
    "min_views": 100
  },
  "count": 12,
  "next": null,
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
      "tags": ["smartphone", "apple", "premium"],
      "views_count": 1250,
      "likes_count": 45,
      "relevance_score": 0.95
    }
  ],
  "facets": {
    "categories": [
      {"id": 1, "name": "Electronics", "count": 12}
    ],
    "price_ranges": [
      {"min": 500, "max": 1000, "count": 8},
      {"min": 1000, "max": 1500, "count": 4}
    ],
    "tags": [
      {"tag": "smartphone", "count": 12},
      {"tag": "apple", "count": 6},
      {"tag": "premium", "count": 4}
    ]
  }
}
```

### Search Suggestions

#### Get Search Suggestions
```http
GET /api/search/suggestions/
```

**Query Parameters:**
- `q` (string): Search query (required)
- `type` (string): Content type filter (products, users, categories, all)
- `limit` (int): Maximum number of suggestions (default: 10, max: 50)

**Example:**
```http
GET /api/search/suggestions/?q=iph&type=products&limit=10
```

**Response:**
```json
{
  "query": "iph",
  "type": "products",
  "suggestions": [
    {
      "text": "iPhone 15 Pro",
      "type": "product",
      "count": 1
    },
    {
      "text": "iPhone 14",
      "type": "product",
      "count": 3
    },
    {
      "text": "iPhone 13",
      "type": "product",
      "count": 2
    }
  ]
}
```

### Search Analytics

#### Get Search Statistics
```http
GET /api/search/stats/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "total_searches": 15420,
  "unique_queries": 3240,
  "popular_queries": [
    {"query": "iphone", "count": 1250},
    {"query": "laptop", "count": 980},
    {"query": "smartphone", "count": 750}
  ],
  "search_by_type": {
    "products": 12000,
    "users": 2500,
    "categories": 920
  },
  "average_results_per_query": 15.5,
  "zero_result_queries": 120
}
```

## Error Responses

### 400 Bad Request
```json
{
  "error": "Invalid search parameters",
  "details": {
    "q": ["Search query is required."],
    "page_size": ["Page size must be between 1 and 100."]
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

### 429 Too Many Requests
```json
{
  "error": "Rate limit exceeded",
  "details": "Too many search requests. Please try again later."
}
```

## Rate Limiting

- **Search requests**: 1000 requests per hour
- **Advanced search**: 500 requests per hour
- **Search suggestions**: 2000 requests per hour
- **Search analytics**: 100 requests per hour

## Implementation Notes

- Search uses full-text search with relevance scoring
- Search results are cached for performance
- Faceted search provides filtering options
- Search suggestions are generated from popular queries
- All timestamps are in ISO 8601 format (UTC)
- Search analytics track usage patterns
- Search results include relevance scores
- Advanced search supports complex filtering
