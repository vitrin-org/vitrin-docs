# Users API

User management endpoints for the Vitrin platform.

## Overview

The Users API provides endpoints for managing user profiles, preferences, and user-related data. All user endpoints require authentication.

## User Model

```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "is_staff": false,
  "is_superuser": false,
  "date_joined": "2024-01-15T10:30:00Z",
  "last_login": "2024-01-20T14:30:00Z",
  "profile": {
    "bio": "Product enthusiast and tech lover",
    "avatar": "https://example.com/avatar.jpg",
    "location": "San Francisco, CA",
    "website": "https://johndoe.com",
    "phone": "+1234567890",
    "birth_date": "1990-05-15",
    "gender": "male"
  },
  "stats": {
    "products_count": 25,
    "followers_count": 150,
    "following_count": 75,
    "likes_received": 320
  }
}
```

## Endpoints

### Get User Profile

#### Get Current User Profile
```http
GET /api/users/profile/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "date_joined": "2024-01-15T10:30:00Z",
  "last_login": "2024-01-20T14:30:00Z",
  "profile": {
    "bio": "Product enthusiast and tech lover",
    "avatar": "https://example.com/avatar.jpg",
    "location": "San Francisco, CA",
    "website": "https://johndoe.com",
    "phone": "+1234567890",
    "birth_date": "1990-05-15",
    "gender": "male"
  },
  "stats": {
    "products_count": 25,
    "followers_count": 150,
    "following_count": 75,
    "likes_received": 320
  }
}
```

#### Get User Profile by ID
```http
GET /api/users/{id}/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "id": 2,
  "username": "jane_smith",
  "first_name": "Jane",
  "last_name": "Smith",
  "is_active": true,
  "date_joined": "2024-01-10T09:15:00Z",
  "profile": {
    "bio": "Fashion and lifestyle blogger",
    "avatar": "https://example.com/jane-avatar.jpg",
    "location": "New York, NY",
    "website": "https://janesmith.com"
  },
  "stats": {
    "products_count": 18,
    "followers_count": 89,
    "following_count": 120,
    "likes_received": 156
  }
}
```

### Update User Profile

#### Update Current User Profile
```http
PUT /api/users/profile/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "first_name": "John",
  "last_name": "Doe",
  "profile": {
    "bio": "Updated bio - Product enthusiast and tech lover",
    "location": "San Francisco, CA",
    "website": "https://johndoe.com",
    "phone": "+1234567890",
    "birth_date": "1990-05-15",
    "gender": "male"
  }
}
```

**Response:**
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "date_joined": "2024-01-15T10:30:00Z",
  "last_login": "2024-01-20T14:30:00Z",
  "profile": {
    "bio": "Updated bio - Product enthusiast and tech lover",
    "avatar": "https://example.com/avatar.jpg",
    "location": "San Francisco, CA",
    "website": "https://johndoe.com",
    "phone": "+1234567890",
    "birth_date": "1990-05-15",
    "gender": "male"
  },
  "stats": {
    "products_count": 25,
    "followers_count": 150,
    "following_count": 75,
    "likes_received": 320
  }
}
```

### Upload Avatar

#### Upload User Avatar
```http
POST /api/users/profile/avatar/
Authorization: Bearer <access_token>
Content-Type: multipart/form-data

{
  "avatar": <file>
}
```

**Response:**
```json
{
  "message": "Avatar uploaded successfully",
  "avatar_url": "https://example.com/new-avatar.jpg"
}
```

### Change Password

#### Change User Password
```http
POST /api/users/change-password/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "old_password": "oldpassword123",
  "new_password": "newpassword123",
  "confirm_password": "newpassword123"
}
```

**Response:**
```json
{
  "message": "Password changed successfully"
}
```

### User Following System

#### Follow User
```http
POST /api/users/{id}/follow/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "User followed successfully",
  "following_count": 76
}
```

#### Unfollow User
```http
DELETE /api/users/{id}/follow/
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "User unfollowed successfully",
  "following_count": 75
}
```

#### Get User Followers
```http
GET /api/users/{id}/followers/
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)

**Response:**
```json
{
  "count": 150,
  "next": "http://api.example.com/api/users/1/followers/?page=2",
  "previous": null,
  "results": [
    {
      "id": 3,
      "username": "follower1",
      "first_name": "Follower",
      "last_name": "One",
      "profile": {
        "avatar": "https://example.com/follower1-avatar.jpg",
        "bio": "Following great products"
      },
      "followed_at": "2024-01-18T10:30:00Z"
    }
  ]
}
```

#### Get User Following
```http
GET /api/users/{id}/following/
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)

**Response:**
```json
{
  "count": 75,
  "next": "http://api.example.com/api/users/1/following/?page=2",
  "previous": null,
  "results": [
    {
      "id": 4,
      "username": "following1",
      "first_name": "Following",
      "last_name": "One",
      "profile": {
        "avatar": "https://example.com/following1-avatar.jpg",
        "bio": "Great product creator"
      },
      "followed_at": "2024-01-16T14:20:00Z"
    }
  ]
}
```

### User Products

#### Get User Products
```http
GET /api/users/{id}/products/
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `ordering` (string): Sort order (name, price, created_at, -name, -price, -created_at)

**Response:**
```json
{
  "count": 25,
  "next": "http://api.example.com/api/users/1/products/?page=2",
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

### User Activity

#### Get User Activity Feed
```http
GET /api/users/{id}/activity/
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `page` (int): Page number for pagination (default: 1)
- `page_size` (int): Number of items per page (default: 20)
- `activity_type` (string): Filter by activity type (product_created, product_liked, user_followed)

**Response:**
```json
{
  "count": 50,
  "next": "http://api.example.com/api/users/1/activity/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "activity_type": "product_created",
      "description": "Created a new product: iPhone 15 Pro",
      "created_at": "2024-01-15T10:30:00Z",
      "metadata": {
        "product_id": 1,
        "product_name": "iPhone 15 Pro"
      }
    },
    {
      "id": 2,
      "activity_type": "product_liked",
      "description": "Liked product: MacBook Pro M3",
      "created_at": "2024-01-15T09:15:00Z",
      "metadata": {
        "product_id": 2,
        "product_name": "MacBook Pro M3"
      }
    }
  ]
}
```

## Error Responses

### 400 Bad Request
```json
{
  "error": "Validation error",
  "details": {
    "first_name": ["This field is required."],
    "profile.bio": ["Bio must be less than 500 characters."]
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
  "details": "You can only view your own private profile"
}
```

### 404 Not Found
```json
{
  "error": "User not found",
  "details": "The requested user does not exist"
}
```

## Rate Limiting

- **Profile updates**: 10 requests per hour
- **Password changes**: 5 requests per hour
- **Avatar uploads**: 20 requests per hour
- **Follow/unfollow**: 100 requests per hour

## Implementation Notes

- User profiles are public by default
- Private profiles can only be viewed by the user themselves
- Avatar images are stored in MinIO object storage
- All timestamps are in ISO 8601 format (UTC)
- User statistics are calculated in real-time
- Activity feeds are paginated for performance
- Password changes require the old password for security
