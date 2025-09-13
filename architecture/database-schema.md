# Database Schema

Database design and relationships for the Vitrin platform.

## Overview

The Vitrin platform uses PostgreSQL as the primary database with a well-structured schema that supports user management, product catalog, categories, comments, and social features.

## Database Design Principles

- **Normalization**: Proper normalization to reduce data redundancy
- **Referential Integrity**: Foreign key constraints to maintain data consistency
- **Indexing**: Strategic indexing for optimal query performance
- **Scalability**: Design that supports horizontal scaling
- **Data Types**: Appropriate data types for each field

## Core Tables

### Users Table

```sql
CREATE TABLE users_user (
    id SERIAL PRIMARY KEY,
    username VARCHAR(150) UNIQUE NOT NULL,
    email VARCHAR(254) UNIQUE NOT NULL,
    first_name VARCHAR(150) NOT NULL,
    last_name VARCHAR(150) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    is_staff BOOLEAN DEFAULT FALSE,
    is_superuser BOOLEAN DEFAULT FALSE,
    date_joined TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login TIMESTAMP WITH TIME ZONE,
    password VARCHAR(128) NOT NULL,
    google_id VARCHAR(100) UNIQUE NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_users_email ON users_user(email);
CREATE INDEX idx_users_username ON users_user(username);
CREATE INDEX idx_users_google_id ON users_user(google_id);
CREATE INDEX idx_users_is_active ON users_user(is_active);
```

### User Profiles Table

```sql
CREATE TABLE users_userprofile (
    id SERIAL PRIMARY KEY,
    user_id INTEGER UNIQUE NOT NULL REFERENCES users_user(id) ON DELETE CASCADE,
    bio TEXT,
    avatar VARCHAR(500),
    location VARCHAR(255),
    website VARCHAR(255),
    phone VARCHAR(20),
    birth_date DATE,
    gender VARCHAR(10) CHECK (gender IN ('male', 'female', 'other', 'prefer_not_to_say')),
    is_private BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_userprofile_user_id ON users_userprofile(user_id);
CREATE INDEX idx_userprofile_location ON users_userprofile(location);
```

### Categories Table

```sql
CREATE TABLE products_category (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    parent_id INTEGER REFERENCES products_category(id) ON DELETE SET NULL,
    image VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_category_slug ON products_category(slug);
CREATE INDEX idx_category_parent_id ON products_category(parent_id);
CREATE INDEX idx_category_is_active ON products_category(is_active);
CREATE INDEX idx_category_name ON products_category(name);
```

### Products Table

```sql
CREATE TABLE products_product (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    category_id INTEGER NOT NULL REFERENCES products_category(id) ON DELETE RESTRICT,
    user_id INTEGER NOT NULL REFERENCES users_user(id) ON DELETE CASCADE,
    is_active BOOLEAN DEFAULT TRUE,
    views_count INTEGER DEFAULT 0,
    likes_count INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_product_category_id ON products_product(category_id);
CREATE INDEX idx_product_user_id ON products_product(user_id);
CREATE INDEX idx_product_is_active ON products_product(is_active);
CREATE INDEX idx_product_price ON products_product(price);
CREATE INDEX idx_product_created_at ON products_product(created_at);
CREATE INDEX idx_product_views_count ON products_product(views_count);
CREATE INDEX idx_product_likes_count ON products_product(likes_count);

-- Full-text search index
CREATE INDEX idx_product_search ON products_product USING gin(to_tsvector('english', name || ' ' || description));
```

### Product Images Table

```sql
CREATE TABLE products_productimage (
    id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products_product(id) ON DELETE CASCADE,
    image VARCHAR(500) NOT NULL,
    alt_text VARCHAR(255),
    is_primary BOOLEAN DEFAULT FALSE,
    order_index INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_productimage_product_id ON products_productimage(product_id);
CREATE INDEX idx_productimage_is_primary ON products_productimage(is_primary);
CREATE INDEX idx_productimage_order_index ON products_productimage(order_index);
```

### Product Tags Table

```sql
CREATE TABLE products_producttag (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE products_product_tags (
    id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products_product(id) ON DELETE CASCADE,
    tag_id INTEGER NOT NULL REFERENCES products_producttag(id) ON DELETE CASCADE,
    UNIQUE(product_id, tag_id)
);

-- Indexes
CREATE INDEX idx_producttag_slug ON products_producttag(slug);
CREATE INDEX idx_product_tags_product_id ON products_product_tags(product_id);
CREATE INDEX idx_product_tags_tag_id ON products_product_tags(tag_id);
```

### Comments Table

```sql
CREATE TABLE products_comment (
    id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products_product(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users_user(id) ON DELETE CASCADE,
    parent_id INTEGER REFERENCES products_comment(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    is_approved BOOLEAN DEFAULT TRUE,
    is_edited BOOLEAN DEFAULT FALSE,
    likes_count INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_comment_product_id ON products_comment(product_id);
CREATE INDEX idx_comment_user_id ON products_comment(user_id);
CREATE INDEX idx_comment_parent_id ON products_comment(parent_id);
CREATE INDEX idx_comment_is_approved ON products_comment(is_approved);
CREATE INDEX idx_comment_created_at ON products_comment(created_at);
```

### User Following Table

```sql
CREATE TABLE users_userfollowing (
    id SERIAL PRIMARY KEY,
    follower_id INTEGER NOT NULL REFERENCES users_user(id) ON DELETE CASCADE,
    following_id INTEGER NOT NULL REFERENCES users_user(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(follower_id, following_id)
);

-- Indexes
CREATE INDEX idx_userfollowing_follower_id ON users_userfollowing(follower_id);
CREATE INDEX idx_userfollowing_following_id ON users_userfollowing(following_id);
```

### Product Likes Table

```sql
CREATE TABLE products_productlike (
    id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products_product(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users_user(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(product_id, user_id)
);

-- Indexes
CREATE INDEX idx_productlike_product_id ON products_productlike(product_id);
CREATE INDEX idx_productlike_user_id ON products_productlike(user_id);
```

### Comment Likes Table

```sql
CREATE TABLE products_commentlike (
    id SERIAL PRIMARY KEY,
    comment_id INTEGER NOT NULL REFERENCES products_comment(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users_user(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(comment_id, user_id)
);

-- Indexes
CREATE INDEX idx_commentlike_comment_id ON products_commentlike(comment_id);
CREATE INDEX idx_commentlike_user_id ON products_commentlike(user_id);
```

## Relationships

### Entity Relationship Diagram

```
Users (1) ──────── (1) UserProfiles
  │
  │ (1)
  │
  └─── (N) Products
  │      │
  │      │ (N)
  │      │
  │      └─── (N) ProductImages
  │      │
  │      │ (N)
  │      │
  │      └─── (N) ProductTags
  │      │
  │      │ (N)
  │      │
  │      └─── (N) Comments
  │      │
  │      │ (N)
  │      │
  │      └─── (N) ProductLikes
  │
  │ (N)
  │
  └─── (N) UserFollowing (N) ──── Users
  │
  │ (N)
  │
  └─── (N) CommentLikes

Categories (1) ──── (N) Products
  │
  │ (1)
  │
  └─── (N) Categories (self-reference)
```

## Data Types and Constraints

### Common Data Types

- **SERIAL**: Auto-incrementing integer primary key
- **VARCHAR(n)**: Variable-length string with max length
- **TEXT**: Unlimited length text
- **DECIMAL(p,s)**: Fixed-point decimal number
- **BOOLEAN**: True/false value
- **TIMESTAMP WITH TIME ZONE**: Date and time with timezone
- **DATE**: Date only (no time)

### Constraints

- **PRIMARY KEY**: Unique identifier for each row
- **FOREIGN KEY**: References to other tables
- **UNIQUE**: Ensures uniqueness across rows
- **NOT NULL**: Prevents null values
- **CHECK**: Validates data against conditions
- **DEFAULT**: Sets default values

## Indexing Strategy

### Primary Indexes

- All primary keys are automatically indexed
- Foreign keys are indexed for join performance
- Unique constraints create unique indexes

### Performance Indexes

- **Search indexes**: Full-text search on product names and descriptions
- **Filter indexes**: Common filter fields (is_active, created_at)
- **Sort indexes**: Fields used for ordering (price, views_count, likes_count)
- **Composite indexes**: Multiple columns for complex queries

### Index Maintenance

- Regular index analysis and optimization
- Monitor index usage and performance
- Drop unused indexes to save space
- Rebuild indexes after bulk data changes

## Data Migration Strategy

### Migration Principles

- **Backward compatibility**: Maintain compatibility with existing data
- **Zero downtime**: Minimize service interruption
- **Data integrity**: Ensure data consistency during migration
- **Rollback capability**: Ability to revert changes if needed

### Migration Process

1. **Planning**: Analyze current schema and plan changes
2. **Testing**: Test migrations on staging environment
3. **Backup**: Create full database backup
4. **Execution**: Run migrations in controlled manner
5. **Verification**: Verify data integrity and performance
6. **Cleanup**: Remove temporary tables and indexes

## Performance Optimization

### Query Optimization

- Use appropriate indexes for common queries
- Optimize JOIN operations
- Use EXPLAIN ANALYZE to identify bottlenecks
- Implement query result caching

### Database Configuration

- **Connection pooling**: Manage database connections efficiently
- **Memory settings**: Optimize PostgreSQL memory configuration
- **Disk I/O**: Use SSD storage for better performance
- **Replication**: Set up read replicas for scaling

### Monitoring

- **Query performance**: Monitor slow queries
- **Index usage**: Track index effectiveness
- **Connection usage**: Monitor connection pool
- **Disk usage**: Track database growth

## Security Considerations

### Data Protection

- **Encryption**: Encrypt sensitive data at rest
- **Access control**: Implement proper user permissions
- **Audit logging**: Track database access and changes
- **Backup encryption**: Encrypt database backups

### SQL Injection Prevention

- Use parameterized queries
- Validate and sanitize input data
- Implement proper error handling
- Regular security audits

## Backup and Recovery

### Backup Strategy

- **Full backups**: Daily full database backups
- **Incremental backups**: Hourly incremental backups
- **Point-in-time recovery**: WAL archiving for PITR
- **Cross-region backups**: Geographic redundancy

### Recovery Procedures

- **Disaster recovery**: Complete system restoration
- **Data recovery**: Restore specific data or tables
- **Point-in-time recovery**: Restore to specific timestamp
- **Testing**: Regular recovery testing and validation
