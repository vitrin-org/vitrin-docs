# Authentication API

Authentication and authorization endpoints for the Vitrin platform.

## Overview

The Vitrin API uses JWT (JSON Web Tokens) for authentication and supports multiple authentication methods including email/password and Google OAuth.

## Authentication Methods

### 1. Email/Password Authentication

#### Register User
```http
POST /api/auth/register/
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123",
  "first_name": "John",
  "last_name": "Doe"
}
```

**Response:**
```json
{
  "user": {
    "id": 1,
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "is_active": true
  },
  "tokens": {
    "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
  }
}
```

#### Login User
```http
POST /api/auth/login/
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response:**
```json
{
  "user": {
    "id": 1,
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "is_active": true
  },
  "tokens": {
    "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
  }
}
```

### 2. Google OAuth Authentication

#### Google OAuth Login
```http
POST /api/auth/google/
Content-Type: application/json

{
  "access_token": "google_access_token_here"
}
```

**Response:**
```json
{
  "user": {
    "id": 1,
    "email": "user@gmail.com",
    "first_name": "John",
    "last_name": "Doe",
    "is_active": true,
    "google_id": "123456789"
  },
  "tokens": {
    "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
  }
}
```

## Token Management

### Refresh Token
```http
POST /api/auth/refresh/
Content-Type: application/json

{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

**Response:**
```json
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

### Logout
```http
POST /api/auth/logout/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

**Response:**
```json
{
  "message": "Successfully logged out"
}
```

## Protected Endpoints

All protected endpoints require the `Authorization` header with a valid JWT token:

```http
Authorization: Bearer <access_token>
```

## Error Responses

### 400 Bad Request
```json
{
  "error": "Invalid credentials",
  "details": "Email or password is incorrect"
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
  "details": "Insufficient permissions"
}
```

## Security Considerations

1. **Token Expiration**: Access tokens expire after 1 hour, refresh tokens after 7 days
2. **HTTPS Only**: All authentication endpoints must use HTTPS in production
3. **Rate Limiting**: Login attempts are rate-limited to prevent brute force attacks
4. **Password Requirements**: Minimum 8 characters with mixed case, numbers, and symbols
5. **Account Lockout**: Accounts are temporarily locked after 5 failed login attempts

## Implementation Notes

- JWT tokens are signed using HS256 algorithm
- Refresh tokens are stored securely and can be revoked
- Google OAuth requires proper client configuration
- All passwords are hashed using bcrypt
- CORS is configured for cross-origin requests
