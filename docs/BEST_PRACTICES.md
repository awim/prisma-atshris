# 🎯 Best Practices & Common Patterns

Development guidelines and recommended patterns for the Prisma Strapi project.

## Table of Contents

1. [Content Type Development](#content-type-development)
2. [API Design Patterns](#api-design-patterns)
3. [Database Best Practices](#database-best-practices)
4. [Security Guidelines](#security-guidelines)
5. [Performance Optimization](#performance-optimization)
6. [Error Handling](#error-handling)
7. [Testing Patterns](#testing-patterns)
8. [Deployment Best Practices](#deployment-best-practices)

---

## Content Type Development

### Schema Design Principles

✅ **DO:**
```json
{
  "kind": "collectionType",
  "displayName": "Article",
  "attributes": {
    "title": {
      "type": "string",
      "required": true,
      "maxLength": 255,
      "configurable": false
    },
    "slug": {
      "type": "uid",
      "targetField": "title"
    },
    "publishedAt": {
      "type": "datetime"
    },
    "author": {
      "type": "relation",
      "relation": "manyToOne",
      "target": "api::author.author"
    }
  }
}
```

❌ **DON'T:**
```json
{
  "attributes": {
    "field_name": {
      "type": "string"
    },
    "authorName": {
      "type": "string"
    },
    "authorEmail": {
      "type": "string"
    }
  }
}
```

**Why?** Use relations instead of duplicating author data; use camelCase for field names; add validation.

### Content Type Naming

```
✅ DO:                      ❌ DON'T:
- article                   - Article
- author                    - AUTHOR
- blog_post                 - blog-post
- product_category          - productCategory
```

**Rule:** Lowercase with underscores for multi-word names in schema.

### Field Validation

```json
{
  "attributes": {
    "email": {
      "type": "email",
      "required": true,
      "unique": true
    },
    "age": {
      "type": "integer",
      "min": 0,
      "max": 150
    },
    "description": {
      "type": "text",
      "maxLength": 5000
    },
    "price": {
      "type": "decimal",
      "required": true,
      "min": 0
    }
  }
}
```

**Best Practice:** Always validate at schema level; add server-side validation in controllers.

---

## API Design Patterns

### RESTful Endpoint Design

✅ **Correct Resource-Based Design:**
```
GET    /api/articles              # Collection
GET    /api/articles/1            # Single resource
GET    /api/articles/1/comments   # Sub-resource (if needed)
POST   /api/articles              # Create
PUT    /api/articles/1            # Update
PATCH  /api/articles/1            # Partial update
DELETE /api/articles/1            # Delete
```

❌ **Avoid Action-Based Design:**
```
GET  /api/getArticles
POST /api/createArticle
PUT  /api/updateArticle/1
DELETE /api/deleteArticle/1
```

### Consistent Response Format

```typescript
// ✅ DO: Use standard format
{
  "data": {
    "id": 1,
    "attributes": { ... }
  },
  "meta": {
    "pagination": { ... }
  }
}

// ❌ DON'T: Inconsistent formats
{
  "id": 1,
  "attributes": { ... }
}

// ❌ DON'T: Top-level arrays
[
  { "id": 1, ... }
]
```

### Filtering & Query Parameters

✅ **Good Filter Patterns:**
```bash
GET /api/articles?filters[status][]=published&filters[status][]=draft
GET /api/articles?filters[publishedAt][$gte]=2026-01-01
GET /api/articles?filters[$or][0][title][$contains]=news
GET /api/articles?sort=-publishedAt,title
GET /api/articles?populate=author,category
```

❌ **Avoid:**
```bash
GET /api/articles?status=published&draft    # Ambiguous
GET /api/articles/published                 # Action-based path
GET /api/searchArticles                     # Use filters instead
```

### Error Response Pattern

```typescript
// ✅ Consistent error format
{
  "data": null,
  "error": {
    "status": 400,
    "name": "ValidationError",
    "message": "Validation failed",
    "details": {
      "errors": [
        {
          "path": ["email"],
          "message": "Email is already taken"
        }
      ]
    }
  }
}
```

### Pagination Pattern

```typescript
// ✅ DO: Always include pagination metadata
{
  "data": [ ... 10 items ... ],
  "meta": {
    "pagination": {
      "page": 2,
      "pageSize": 10,
      "pageCount": 5,
      "total": 47
    }
  }
}

// ✅ DO: Default and max limits
- Default pageSize: 25
- Max pageSize: 100
- Client can override within limits
```

---

## Database Best Practices

### Index Strategy

```typescript
// config/database.ts
export default ({ env }) => ({
  connection: {
    // ... connection settings ...
    migrations: {
      auto: true,
      safe: true
    }
  },
  
  // Add indexes for frequently filtered/sorted fields
  // These are created through admin panel or migrations
});
```

**Index Frequently:**
- Fields used in `filters` (author, category, status)
- Fields used in `sort` (publishedAt, createdAt)
- Foreign key relationships
- Unique fields (email, username)

```sql
-- Example indexes
CREATE INDEX idx_article_author_id ON articles(author_id);
CREATE INDEX idx_article_published_at ON articles(published_at);
CREATE INDEX idx_article_status ON articles(status);
```

### Query Optimization

❌ **Avoid N+1 Queries:**
```typescript
// DON'T: This causes multiple queries
const articles = await strapi.entityService.findMany('api::article.article');
articles.forEach(article => {
  // This runs a query for EACH article!
  const author = strapi.entityService.findOne(
    'api::author.author',
    article.author
  );
});
```

✅ **Use Population:**
```typescript
// DO: Load relations in one query
const articles = await strapi.entityService.findMany(
  'api::article.article',
  {
    populate: {
      author: true,
      category: true,
      seo: true
    }
  }
);
```

### Database Connection Pooling

```env
# .env for production
DATABASE_CLIENT=mysql
DATABASE_HOST=db.example.com

# Pool settings automatically configured
# Min connections: 2
# Max connections: 10
# Adjust if needed based on load
```

---

## Security Guidelines

### Authentication in Controllers

✅ **DO: Check authentication:**
```typescript
export default factories.createCoreController(
  'api::article.article',
  {
    async create(ctx) {
      // Authentication is automatic for core controller
      // But explicit check for safety:
      if (!ctx.state.user) {
        return ctx.unauthorized('User not authenticated');
      }
      
      return await super.create(ctx);
    }
  }
);
```

### Authorization (Role-Based Access)

✅ **DO: Implement permission checks:**
```typescript
async update(ctx) {
  const user = ctx.state.user;
  const articleId = ctx.params.id;
  
  // Check if user is Super Admin
  if (user.roles.some(r => r.name === 'Super Admin')) {
    return await super.update(ctx);
  }
  
  // Check if user owns the article
  const article = await strapi.entityService.findOne(
    'api::article.article',
    articleId,
    { populate: { author: true } }
  );
  
  if (article.author.user_id !== user.id) {
    return ctx.forbidden('Cannot edit other authors articles');
  }
  
  return await super.update(ctx);
}
```

### Sensitive Data Handling

✅ **DO:**
```env
# .env - Store in environment, not code
DATABASE_PASSWORD=secure_password
API_SECRET_KEY=xxxxxxxxxxxx
STRIPE_API_KEY=sk_live_xxxxx
```

❌ **DON'T:**
```typescript
// ❌ NEVER hardcode secrets
const dbPassword = 'password123';
const apiKey = 'sk_test_xxxxx';
```

### Input Validation

✅ **DO: Validate all inputs:**
```typescript
async create(ctx) {
  const { title, email, age } = ctx.request.body.data;
  
  // Validate required fields
  if (!title || !email) {
    return ctx.badRequest('Title and email are required');
  }
  
  // Validate format
  if (!email.includes('@')) {
    return ctx.badRequest('Invalid email format');
  }
  
  // Validate range
  if (age && (age < 0 || age > 150)) {
    return ctx.badRequest('Age must be between 0 and 150');
  }
  
  return await super.create(ctx);
}
```

### CORS Configuration

✅ **DO: Specify allowed origins:**
```env
# Production: Only allow your frontend domain
CORS_ORIGIN=https://app.example.com,https://www.example.com

# Development: Allow localhost
NODE_ENV=development  # Auto-allows localhost
```

---

## Performance Optimization

### Query Optimization

❌ **Inefficient:**
```bash
GET /api/articles?populate=*&pagination[pageSize]=1000
```

✅ **Optimized:**
```bash
GET /api/articles?populate=author,category&pagination[pageSize]=20&sort=-publishedAt
```

**Rules:**
1. Use pagination (never fetch all records)
2. Populate only needed relations
3. Sort by indexed fields
4. Add filters to reduce result set

### Caching Patterns

```typescript
// Simple request cache
const cache = new Map();

export default factories.createCoreController('api::article.article', {
  async find(ctx) {
    const cacheKey = `articles_${ctx.querystring}`;
    
    // Check cache first
    if (cache.has(cacheKey)) {
      return cache.get(cacheKey);
    }
    
    // Fetch and cache
    const result = await super.find(ctx);
    cache.set(cacheKey, result);
    
    // Clear cache after 5 minutes
    setTimeout(() => cache.delete(cacheKey), 5 * 60 * 1000);
    
    return result;
  }
});
```

### Response Compression

✅ **Enabled by default in Strapi:**
```typescript
// config/server.ts
export default ({ env }) => ({
  compression: {
    enabled: true,
    gzip: {
      level: 6
    }
  }
});
```

### Field Selection (Partial Responses)

Consider implementing field selection to reduce payload:
```bash
GET /api/articles?fields=id,title,author  # Only return these fields
```

---

## Error Handling

### Standard Error Response

```typescript
// ✅ DO: Use consistent error handling
export default factories.createCoreController('api::article.article', {
  async create(ctx) {
    try {
      // Validation
      if (!ctx.request.body.data?.title) {
        return ctx.badRequest('Title is required');
      }
      
      return await super.create(ctx);
    } catch (error) {
      strapi.log.error(error);
      return ctx.internalServerError('Failed to create article');
    }
  }
});
```

### HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | DELETE success |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid token |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate unique field |
| 422 | Unprocessable | Validation failed |
| 500 | Server Error | Unexpected error |

### Logging

```typescript
// ✅ DO: Log important operations
export default factories.createCoreController('api::article.article', {
  async create(ctx) {
    strapi.log.info(`Creating article: ${ctx.request.body.data.title}`);
    
    try {
      const result = await super.create(ctx);
      strapi.log.info(`Article created with ID: ${result.data.id}`);
      return result;
    } catch (error) {
      strapi.log.error(`Failed to create article: ${error.message}`);
      throw error;
    }
  }
});
```

---

## Testing Patterns

### API Testing with cURL

```bash
# Test create
curl -X POST http://localhost:1337/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "data": {
      "title": "Test Article",
      "slug": "test-article"
    }
  }'

# Test update
curl -X PUT http://localhost:1337/api/articles/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"data":{"title":"Updated Title"}}'

# Test delete
curl -X DELETE http://localhost:1337/api/articles/1 \
  -H "Authorization: Bearer $TOKEN"
```

### Testing Filters

```bash
# Test basic filter
curl "http://localhost:1337/api/articles?filters[title][$contains]=test"

# Test date filter
curl "http://localhost:1337/api/articles?filters[publishedAt][$gte]=2026-01-01"

# Test relationship filter
curl "http://localhost:1337/api/articles?filters[author][id]=1"

# Test complex filter
curl "http://localhost:1337/api/articles?filters[\$or][0][title][\$contains]=news&filters[\$or][1][slug][$contains]=news"
```

---

## Deployment Best Practices

### Pre-Deployment Checklist

```
✅ Environment Preparation
  [ ] Generate new APP_KEYS for production
  [ ] Use strong database passwords
  [ ] Configure HTTPS certificates
  [ ] Set NODE_ENV=production
  
✅ Database
  [ ] Backup production database
  [ ] Test migrations on staging
  [ ] Verify database connection
  [ ] Check connection pool settings
  
✅ Security
  [ ] Review security headers
  [ ] Verify authentication is working
  [ ] Check CORS configuration
  [ ] Test token expiration
  
✅ Performance
  [ ] Build admin panel
  [ ] Test API response times
  [ ] Verify indexes are in place
  [ ] Test pagination with large datasets
  
✅ Monitoring
  [ ] Setup error logging (Sentry)
  [ ] Configure uptime monitoring
  [ ] Setup alerts
  [ ] Plan rollback strategy
```

### Health Check Endpoint

```typescript
// Create simple health check
export default (router) => {
  router.get('/health', (ctx) => {
    ctx.body = {
      status: 'ok',
      timestamp: new Date(),
      uptime: process.uptime()
    };
  });
};
```

**Usage:**
```bash
curl http://localhost:1337/api/health
```

### Environment Variables for Production

```env
NODE_ENV=production
HOST=0.0.0.0
PORT=1337

# Database
DATABASE_CLIENT=mysql
DATABASE_HOST=prod-db.example.com
DATABASE_NAME=strapi_prod
DATABASE_PASSWORD=VERY_SECURE

# Security
APP_KEYS=newkey1,newkey2,newkey3,newkey4
API_TOKEN_SALT=newsalt
ADMIN_JWT_SECRET=newsecret
JWT_SECRET=newsecret

# File uploads
AWS_ACCESS_KEY_ID=xxxx
AWS_SECRET_ACCESS_KEY=xxxx

# Monitoring
SENTRY_DSN=https://xxxxx@sentry.io/xxxxx
```

---

## Code Quality Guidelines

### TypeScript Best Practices

```typescript
// ✅ DO: Use types
async function getArticle(id: number): Promise<Article> {
  return await strapi.entityService.findOne(
    'api::article.article',
    id,
    { populate: { author: true } }
  );
}

// ❌ DON'T: Use any
async function getArticle(id: any): Promise<any> {
  return await strapi.entityService.findOne('api::article.article', id);
}
```

### File Organization

```
src/api/article/
├── content-types/
│   └── article/
│       └── schema.json
├── controllers/
│   ├── article.ts        # Main controller
│   └── index.ts          # Exports
├── services/
│   ├── article.ts        # Main service
│   └── index.ts          # Exports
├── routes/
│   └── article.ts
└── middlewares/          # Optional custom middlewares
    ├── validate.ts
    └── index.ts
```

---

**Related Documentation:**
- [Developer Starter Guide](./DEVELOPER_GUIDE.md)
- [Architecture Guide](./ARCHITECTURE.md)
- [API Reference](./API_REFERENCE.md)
- [Quick Reference](./QUICK_REFERENCE.md)

