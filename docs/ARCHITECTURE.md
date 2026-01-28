# 🏗️ Architecture Guide

This document provides an in-depth overview of the Prisma Strapi system architecture and design patterns.

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                   Client Applications                    │
│  (Web Apps, Mobile Apps, Third-party Integrations)      │
└────────────────────┬────────────────────────────────────┘
                     │ HTTP/REST
┌─────────────────────▼────────────────────────────────────┐
│                  Strapi API Gateway                       │
│              (Express.js + REST Routes)                   │
└────────┬─────────────┬──────────────┬────────────────────┘
         │             │              │
    ┌────▼──┐    ┌─────▼───┐   ┌─────▼─────┐
    │Content│    │ Auth &  │   │ Plugins & │
    │Routes │    │ Perms   │   │Extensions │
    └────┬──┘    └─────┬───┘   └─────┬─────┘
         │             │              │
┌────────▼─────────────▼──────────────▼──────────────────┐
│           Application Layer                             │
│  - Controllers (Business Logic)                         │
│  - Services (Data Operations)                          │
│  - Middlewares (Request Processing)                    │
└──────────────────┬──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│          Database Layer                                  │
│  - SQLite (Dev) / MySQL (Prod)                         │
│  - Knex.js Query Builder                               │
│  - Migrations & Seeders                                │
└──────────────────────────────────────────────────────────┘
```

## Content Type Architecture

### Standard Content Type Structure

Each content type follows this pattern:

```
src/api/[content-type]/
├── content-types/
│   └── [content-type]/
│       └── schema.json           # Defines data structure & fields
├── controllers/
│   └── [content-type].ts         # Handles HTTP requests
├── routes/
│   └── [content-type].ts         # Defines API endpoints
└── services/
    └── [content-type].ts         # Business logic & data operations
```

### File Responsibilities

| File | Purpose |
|------|---------|
| `schema.json` | Data model definition (fields, relations, attributes) |
| `controllers.ts` | HTTP request handlers, response formatting |
| `routes.ts` | Endpoint mapping, middleware application |
| `services.ts` | Database queries, business logic, data transformation |

### Example: Article Content Type

**Schema Definition** (`schema.json`):
```json
{
  "kind": "collectionType",
  "collectionName": "articles",
  "info": {
    "singularName": "article",
    "pluralName": "articles",
    "displayName": "Article"
  },
  "options": { "increments": true },
  "attributes": {
    "title": { "type": "string", "required": true },
    "slug": { "type": "uid", "targetField": "title" },
    "content": { "type": "richtext" },
    "category": { "type": "relation", "relation": "manyToOne", "target": "api::category.category" },
    "author": { "type": "relation", "relation": "manyToOne", "target": "api::author.author" },
    "publishedAt": { "type": "datetime" }
  }
}
```

**Controller** (`controllers/article.ts`):
```typescript
import { factories } from '@strapi/strapi';

export default factories.createCoreController('api::article.article', {
  async find(ctx) {
    // Pre-hook: modify query
    return await super.find(ctx);
  },
  
  async create(ctx) {
    // Validate & create
    return await super.create(ctx);
  }
});
```

**Service** (`services/article.ts`):
```typescript
import { factories } from '@strapi/strapi';

export default factories.createCoreService('api::article.article', {
  async findBySlug(slug) {
    return await strapi.entityService.findMany(
      'api::article.article',
      { filters: { slug }, populate: ['*'] }
    );
  }
});
```

## Content Type Relationships

### Defined Relations in Project

```
Article
├── has many Authors (many-to-one)
├── has many Categories (many-to-one)
└── uses Components (media, rich-text, seo)

Author
├── can write Articles (one-to-many)
└── can have Profile Info

Category
├── groups Articles (one-to-many)
└── can be nested (self-relation)

Global
└── stores Site-wide Settings (singleton)

About
└── Static Page Content
```

### Relation Types

| Type | Example | Usage |
|------|---------|-------|
| One-to-Many | Author has many Articles | Parent → Multiple children |
| Many-to-One | Article belongs to Author | Child → Single parent |
| Many-to-Many | Articles ↔ Categories | Bidirectional multiple |
| One-to-One | User ↔ Profile | Single pairing |
| Polymorphic | Media used in Article & Author | Multiple content types |

## Component Architecture

Components are reusable field sets across content types.

**Location**: `src/components/shared/`

### Available Components

1. **Media Component** (`media.json`)
   - Image/file uploads
   - Used in: Articles, About, Global
   - Fields: url, alt text, caption

2. **Rich Text Component** (`rich-text.json`)
   - WYSIWYG text editor
   - Used in: Articles, Global
   - Supports markdown, HTML

3. **SEO Component** (`seo.json`)
   - Meta tags management
   - Used in: Articles, About
   - Fields: meta_title, meta_description, keywords

4. **Slider Component** (`slider.json`)
   - Image carousel
   - Used in: Global, Category
   - Fields: slides (array of media)

5. **Quote Component** (`quote.json`)
   - Testimonials/quotes
   - Used in: Articles, About
   - Fields: text, author, attribution

## API Endpoint Patterns

All endpoints follow REST conventions:

```
GET    /api/articles              → List all articles
GET    /api/articles?populate=*   → List with relations
GET    /api/articles/{id}         → Get single article
POST   /api/articles              → Create article
PUT    /api/articles/{id}         → Update article
DELETE /api/articles/{id}         → Delete article

GET    /api/articles?filters[category]=5   → Filter
GET    /api/articles?pagination[pageSize]=10&pagination[page]=2  → Paginate
GET    /api/articles?sort=publishedAt:desc → Sort
```

## Request/Response Flow

### Incoming Request Flow

```
HTTP Request
    ↓
Strapi Router (routes/[type].ts)
    ↓
Middlewares (auth, cors, body-parser)
    ↓
Controller (controllers/[type].ts)
    ├─ Validate request
    ├─ Call service
    └─ Format response
    ↓
Service (services/[type].ts)
    ├─ Query database
    ├─ Transform data
    └─ Return result
    ↓
HTTP Response (JSON)
```

### Response Structure

**Success Response** (200 OK):
```json
{
  "data": {
    "id": 1,
    "documentId": "uuid-string",
    "title": "Article Title",
    "content": "...",
    "author": { "id": 1, "name": "John Doe" },
    "createdAt": "2026-01-28T10:30:00Z",
    "updatedAt": "2026-01-28T10:30:00Z"
  },
  "meta": {}
}
```

**List Response** (200 OK):
```json
{
  "data": [
    { "id": 1, "title": "Article 1", ... },
    { "id": 2, "title": "Article 2", ... }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 10,
      "pageCount": 5,
      "total": 47
    }
  }
}
```

**Error Response** (4xx/5xx):
```json
{
  "data": null,
  "error": {
    "status": 400,
    "name": "ValidationError",
    "message": "Invalid input",
    "details": {
      "errors": [
        { "path": ["title"], "message": "This field is required" }
      ]
    }
  }
}
```

## Database Layer

### SQLite Configuration (Development)

- **File**: `.tmp/data.db`
- **Connection**: File-based, zero-setup
- **Best for**: Local development, testing
- **Limitations**: Single-user, limited concurrency

### MySQL Configuration (Production)

```env
DATABASE_CLIENT=mysql
DATABASE_HOST=db.example.com
DATABASE_PORT=3306
DATABASE_NAME=strapi_prod
DATABASE_USERNAME=strapi_user
DATABASE_PASSWORD=secure_password
```

### Migration System

Strapi auto-generates migrations when schema changes:

```bash
# Migrations stored in: database/migrations/

# Manual migration (if needed)
yarn strapi migrate
```

## Middleware Stack

**Order of Execution**:

1. **CORS Middleware** - Cross-origin requests
2. **Body Parser** - JSON parsing
3. **Authentication** - JWT token validation
4. **Authorization** - Permission checks
5. **Rate Limiting** - (if configured)
6. **Route Handler** - Your controller logic
7. **Error Handler** - Exception formatting

## Security Architecture

### Authentication Flow

```
Client Request
    ↓
Extract JWT from Header: "Authorization: Bearer <token>"
    ↓
Verify Token Signature (using ADMIN_JWT_SECRET)
    ↓
Decode Claims (user ID, role, permissions)
    ↓
Attach User to Request Context (ctx.state.user)
    ↓
Check Authorization (role-based access control)
    ↓
Allow/Deny Route Handler
```

### Key Security Features

- **JWT Tokens**: Short-lived access tokens
- **API Keys**: Long-lived integration tokens
- **RBAC**: Role-Based Access Control
- **CORS**: Restricted cross-origin access
- **SSL/TLS**: HTTPS encryption
- **CSRF Protection**: Anti-forgery tokens

## Plugin Architecture

### Built-in Plugins

```
@strapi/plugin-cloud          → Cloud deployment features
@strapi/plugin-users-permissions → User management & roles
```

### Plugin Structure

```
plugins/
└── my-plugin/
    ├── server/
    │   ├── index.ts           # Plugin entry point
    │   ├── routes/
    │   └── controllers/
    └── strapi-server.js       # Server configuration
```

## Configuration Files

### config/server.ts
- HOST and PORT settings
- Application keys for encryption
- CORS configuration

### config/database.ts
- Database client selection
- Connection pooling
- SSL options
- Migration settings

### config/middlewares.ts
- Express middleware registration
- CORS, rate limiting, logging

### config/plugins.ts
- Plugin configuration
- Feature flags
- Custom settings

### config/api.ts
- API-wide settings
- Response formatting
- Default pagination

## Deployment Architecture

### Development Stack
```
Node.js Process
↓
Express Server (Port 1337)
↓
SQLite Database (.tmp/data.db)
↓
In-memory file uploads
```

### Production Stack
```
Load Balancer
↓
Multiple Node.js Instances (Auto-scaling)
↓
Connection Pool
↓
MySQL Database (Replicated)
↓
Object Storage (S3/Cloud Storage)
↓
CDN (Image optimization)
```

## Performance Considerations

### Query Optimization
- Use `populate` selectively (avoid N+1 queries)
- Implement pagination (default: 25 items per page)
- Add database indexes on frequently filtered fields
- Cache static content (Global settings, Categories)

### Caching Strategy
```
Client Browser Cache (Static assets)
        ↓
CDN Cache (Images, compiled code)
        ↓
Server Cache (Query results, ORM cache)
        ↓
Database Cache (Connection pool, query cache)
```

### File Upload Handling
- Uploads stored in: `/public/uploads/`
- Served through: http://localhost:1337/uploads/[filename]
- Size limits: Configure in plugin settings
- Formats: Image optimization on upload

---

**Related Documentation:**
- [Developer Starter Guide](./DEVELOPER_GUIDE.md)
- [API Reference](./API_REFERENCE.md)
- [Troubleshooting Guide](../DEVELOPER_GUIDE.md#troubleshooting)
