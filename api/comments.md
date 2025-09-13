# Comments API

Comment system endpoints for the Vitrin platform.

## Overview

The Comments API provides endpoints for managing product comments, including creating, updating, deleting, and moderating comments. Comments support threading and moderation features.

## Comment Model

```json
{
  "id": 1,
  "product": {
    "id": 1,
    "name": "iPhone 15 Pro",
    "slug": "iphone-15-pro"
  },
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe",
    "profile": {
      "avatar": "https://example.com/avatar.jpg"
    }
  },
  "parent": null,
  "content": "This is an amazing product! The camera quality is outstanding.",
  "is_approved": true,
  "is_edited": false,
  "likes_count": 5,
  "replies_count": 2,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "replies": [
    {
      "id": 2,
      "user": {
        "id": 2,
        "username": "jane_smith",
        "first_name": "Jane",
        "last_name": "Smith",
        "profile": {
          "avatar": "https://example.com/jane-avatar.jpg"
        }
      },
      "parent": 1,
      "content": "I agree! The battery life is also impressive.",
      "is_approved": true,
      "is_edited": false,
      "likes_count": 2,
      "replies_count": 0,
      "created_at": "2024-01-15T11:00:00Z",
      "updated_at": "2024-01-15T11:00:00Z"
    }
  ]
}
```

## Endpoints

### List Comments

#### Get Product Comments
```http
GET /api/products/{product_id}/comments/
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20, max: 100)
- `parent` (int): Filter by parent comment ID (null for top-level comments)
- `ordering` (string): Sort order (created_at, -created_at, likes_count, -likes_count)
- `include_replies` (boolean): Include nested replies (default: true)

**Example:**
```http
GET /api/products/1/comments/?page=1&page_size=20&parent=null&ordering=-created_at&include_replies=true
```

**Response:**
```json
{
  "count": 45,
  "next": "http://api.example.com/api/products/1/comments/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "product": {
        "id": 1,
        "name": "iPhone 15 Pro",
        "slug": "iphone-15-pro"
      },
      "user": {
        "id": 1,
        "username": "john_doe",
        "first_name": "John",
        "last_name": "Doe",
        "profile": {
          "avatar": "https://example.com/avatar.jpg"
        }
      },
      "parent": null,
      "content": "This is an amazing product! The camera quality is outstanding.",
      "is_approved": true,
      "is_edited": false,
      "likes_count": 5,
      "replies_count": 2,
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z",
      "replies": [
        {
          "id": 2,
          "user": {
            "id": 2,
            "username": "jane_smith",
            "first_name": "Jane",
            "last_name": "Smith",
            "profile": {
              "avatar": "https://example.com/jane-avatar.jpg"
            }
          },
          "parent": 1,
          "content": "I agree! The battery life is also impressive.",
          "is_approved": true,
          "is_edited": false,
          "likes_count": 2,
          "replies_count": 0,
          "created_at": "2024-01-15T11:00:00Z",
          "updated_at": "2024-01-15T11:00:00Z"
        }
      ]
    }
  ]
}
```

#### Get User Comments
```http
GET /api/users/{user_id}/comments/
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `ordering` (string): Sort order (created_at, -created_at)

**Response:**
```json
{
  "count": 25,
  "next": "http://api.example.com/api/users/1/comments/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "product": {
        "id": 1,
        "name": "iPhone 15 Pro",
        "slug": "iphone-15-pro"
      },
      "user": {
        "id": 1,
        "username": "john_doe",
        "first_name": "John",
        "last_name": "Doe"
      },
      "parent": null,
      "content": "This is an amazing product! The camera quality is outstanding.",
      "is_approved": true,
      "is_edited": false,
      "likes_count": 5,
      "replies_count": 2,
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

### Get Comment Details

#### Get Single Comment
```http
GET /api/comments/{id}/
```

**Response:**
```json
{
  "id": 1,
  "product": {
    "id": 1,
    "name": "iPhone 15 Pro",
    "slug": "iphone-15-pro"
  },
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe",
    "profile": {
      "avatar": "https://example.com/avatar.jpg"
    }
  },
  "parent": null,
  "content": "This is an amazing product! The camera quality is outstanding.",
  "is_approved": true,
  "is_edited": false,
  "likes_count": 5,
  "replies_count": 2,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "replies": [
    {
      "id": 2,
      "user": {
        "id": 2,
        "username": "jane_smith",
        "first_name": "Jane",
        "last_name": "Smith",
        "profile": {
          "avatar": "https://example.com/jane-avatar.jpg"
        }
      },
      "parent": 1,
      "content": "I agree! The battery life is also impressive.",
      "is_approved": true,
      "is_edited": false,
      "likes_count": 2,
      "replies_count": 0,
      "created_at": "2024-01-15T11:00:00Z",
      "updated_at": "2024-01-15T11:00:00Z"
    }
  ]
}
```

### Create Comment

#### Create New Comment
```http
POST /api/products/{product_id}/comments/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "content": "Great product! I've been using it for a week now and it's fantastic.",
  "parent": null
}
```

**Response:**
```json
{
  "id": 3,
  "product": {
    "id": 1,
    "name": "iPhone 15 Pro",
    "slug": "iphone-15-pro"
  },
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe",
    "profile": {
      "avatar": "https://example.com/avatar.jpg"
    }
  },
  "parent": null,
  "content": "Great product! I've been using it for a week now and it's fantastic.",
  "is_approved": true,
  "is_edited": false,
  "likes_count": 0,
  "replies_count": 0,
  "created_at": "2024-01-20T15:30:00Z",
  "updated_at": "2024-01-20T15:30:00Z"
}
```

#### Create Reply to Comment
```http
POST /api/products/{product_id}/comments/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "content": "Thanks for sharing your experience!",
  "parent": 1
}
```

**Response:**
```json
{
  "id": 4,
  "product": {
    "id": 1,
    "name": "iPhone 15 Pro",
    "slug": "iphone-15-pro"
  },
  "user": {
    "id": 2,
    "username": "jane_smith",
    "first_name": "Jane",
    "last_name": "Smith",
    "profile": {
      "avatar": "https://example.com/jane-avatar.jpg"
    }
  },
  "parent": 1,
  "content": "Thanks for sharing your experience!",
  "is_approved": true,
  "is_edited": false,
  "likes_count": 0,
  "replies_count": 0,
  "created_at": "2024-01-20T16:00:00Z",
  "updated_at": "2024-01-20T16:00:00Z"
}
```

### Update Comment

#### Update Existing Comment
```http
PUT /api/comments/{id}/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "content": "Great product! I've been using it for a week now and it's fantastic. The camera quality is especially impressive."
}
```

**Response:**
```json
{
  "id": 3,
  "product": {
    "id": 1,
    "name": "iPhone 15 Pro",
    "slug": "iphone-15-pro"
  },
  "user": {
    "id": 1,
    "username": "john_doe",
    "first_name": "John",
    "last_name": "Doe",
    "profile": {
      "avatar": "https://example.com/avatar.jpg"
    }
  },
  "parent": null,
  "content": "Great product! I've been using it for a week now and it's fantastic. The camera quality is especially impressive.",
  "is_approved": true,
  "is_edited": true,
  "likes_count": 0,
  "replies_count": 0,
  "created_at": "2024-01-20T15:30:00Z",
  "updated_at": "2024-01-20T16:30:00Z"
}
```

### Delete Comment

#### Delete Comment
```http
DELETE /api/comments/{id}/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Comment deleted successfully"
}
```

### Comment Interactions

#### Like Comment
```http
POST /api/comments/{id}/like/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Comment liked successfully",
  "likes_count": 6
}
```

#### Unlike Comment
```http
DELETE /api/comments/{id}/like/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Comment unliked successfully",
  "likes_count": 5
}
```

### Comment Moderation

#### Approve Comment (Staff Only)
```http
POST /api/comments/{id}/approve/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Comment approved successfully",
  "is_approved": true
}
```

#### Reject Comment (Staff Only)
```http
POST /api/comments/{id}/reject/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Comment rejected successfully",
  "is_approved": false
}
```

#### Report Comment
```http
POST /api/comments/{id}/report/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "reason": "spam",
  "description": "This comment appears to be spam"
}
```

**Response:**
```json
{
  "message": "Comment reported successfully"
}
```

## Error Responses

### 400 Bad Request
```json
{
  "error": "Validation error",
  "details": {
    "content": ["This field is required."],
    "content": ["Comment must be between 1 and 1000 characters."]
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
  "details": "You can only modify your own comments"
}
```

### 404 Not Found
```json
{
  "error": "Comment not found",
  "details": "The requested comment does not exist"
}
```

### 409 Conflict
```json
{
  "error": "Comment conflict",
  "details": "Cannot delete comment with replies"
}
```

## Rate Limiting

- **Comment creation**: 50 requests per hour
- **Comment updates**: 20 requests per hour
- **Comment likes**: 100 requests per hour
- **Comment reports**: 10 requests per hour

## Implementation Notes

- Comments support unlimited nesting levels
- Comment approval is automatic for authenticated users
- Comment editing is allowed within 24 hours of creation
- Comment deletion is prevented if it has replies
- All timestamps are in ISO 8601 format (UTC)
- Comment content is sanitized to prevent XSS attacks
- Spam detection is implemented using content analysis
- Comment notifications are sent to product owners
