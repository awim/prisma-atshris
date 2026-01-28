# 📖 API Reference Guide

Complete API endpoint documentation for the Prisma Strapi system.

## Base URL

```
http://localhost:1337/api
```

## Authentication

Most endpoints require authentication using Bearer tokens:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:1337/api/articles
```

### Getting a Token

**Via Admin Panel:**
1. Login to http://localhost:1337/admin
2. Go to Settings → API Tokens
3. Create new token with desired permissions
4. Copy and use in Authorization header

**Via Login Endpoint:**
```bash
curl -X POST http://localhost:1337/api/auth/local \
  -H "Content-Type: application/json" \
  -d '{
    "identifier": "admin@example.com",
    "password": "your-password"
  }'
```

Response:
```json
{
  "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "email": "admin@example.com",
    "username": "admin"
  }
}
```

## Query Parameters

### Population (Relations)

Populate related data:

```bash
# Populate all relations
GET /api/articles?populate=*

# Populate specific relations
GET /api/articles?populate=author,category

# Nested population
GET /api/articles?populate[author][populate]=*

# Component population
GET /api/articles?populate=seo,media
```

### Filtering

Filter results by field values:

```bash
# Exact match
GET /api/articles?filters[title]=My Article

# Comparison operators
GET /api/articles?filters[publishedAt][$gte]=2026-01-01
GET /api/articles?filters[id][$gt]=5
GET /api/articles?filters[id][$lt]=100

# String operators
GET /api/articles?filters[title][$contains]=article
GET /api/articles?filters[title][$startsWith]=Breaking
GET /api/articles?filters[title][$endsWith]=News

# Logical operators (AND)
GET /api/articles?filters[$and][0][title][$contains]=news&filters[$and][1][publishedAt][$gte]=2026-01-01

# Logical operators (OR)
GET /api/articles?filters[$or][0][title][$contains]=news&filters[$or][1][title][$contains]=update

# Null checks
GET /api/articles?filters[author][$null]=true
GET /api/articles?filters[deletedAt][$notNull]=true
```

**Available Operators:**
- `$eq` - Equals
- `$ne` - Not equals
- `$gt` - Greater than
- `$gte` - Greater than or equal
- `$lt` - Less than
- `$lte` - Less than or equal
- `$in` - In array
- `$nin` - Not in array
- `$contains` - String contains
- `$notContains` - String doesn't contain
- `$startsWith` - Starts with
- `$endsWith` - Ends with
- `$null` - Is null
- `$notNull` - Is not null

### Sorting

Sort results by field:

```bash
# Ascending order (default)
GET /api/articles?sort=title

# Descending order
GET /api/articles?sort=publishedAt:desc

# Multiple sorts
GET /api/articles?sort=category:asc,publishedAt:desc
```

### Pagination

Control result pagination:

```bash
# Set page size
GET /api/articles?pagination[pageSize]=20

# Get specific page
GET /api/articles?pagination[page]=2&pagination[pageSize]=10

# Start offset (alternative to page)
GET /api/articles?pagination[start]=20&pagination[limit]=10
```

Default pagination: 25 items per page

Response includes:
```json
{
  "data": [...],
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 25,
      "pageCount": 3,
      "total": 67
    }
  }
}
```

## Content Type Endpoints

### Articles API

#### List Articles

```bash
GET /api/articles
GET /api/articles?populate=*&pagination[pageSize]=10&sort=publishedAt:desc
```

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "documentId": "uuid-123",
      "title": "Breaking News",
      "slug": "breaking-news",
      "content": "<p>Article content...</p>",
      "publishedAt": "2026-01-28T10:30:00Z",
      "author": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com"
      },
      "category": {
        "id": 5,
        "name": "Technology",
        "slug": "technology"
      },
      "seo": {
        "metaTitle": "Breaking News | Your Site",
        "metaDescription": "Latest news article",
        "keywords": "news, breaking, technology"
      },
      "createdAt": "2026-01-28T10:30:00Z",
      "updatedAt": "2026-01-28T10:30:00Z"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 10,
      "pageCount": 7,
      "total": 67
    }
  }
}
```

#### Get Single Article

```bash
GET /api/articles/1
GET /api/articles/uuid-123  # By documentId
GET /api/articles?filters[slug]=breaking-news
```

**Response:**
```json
{
  "data": {
    "id": 1,
    "documentId": "uuid-123",
    "title": "Breaking News",
    ...
  }
}
```

#### Create Article

```bash
POST /api/articles
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "data": {
    "title": "New Article",
    "slug": "new-article",
    "content": "<p>Content here</p>",
    "author": 1,
    "category": 5,
    "publishedAt": "2026-01-28T10:30:00Z",
    "seo": {
      "metaTitle": "New Article",
      "metaDescription": "Article description"
    }
  }
}
```

**Response:** 201 Created
```json
{
  "data": {
    "id": 68,
    "documentId": "uuid-new",
    "title": "New Article",
    ...
  }
}
```

#### Update Article

```bash
PUT /api/articles/1
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "data": {
    "title": "Updated Title",
    "publishedAt": "2026-01-28T15:00:00Z"
  }
}
```

#### Delete Article

```bash
DELETE /api/articles/1
Authorization: Bearer YOUR_TOKEN
```

**Response:** 200 OK (empty data)

---

### Authors API

#### List Authors

```bash
GET /api/authors?populate=articles
```

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "documentId": "uuid-author-1",
      "name": "John Doe",
      "email": "john@example.com",
      "bio": "Experienced journalist",
      "avatar": {
        "id": 10,
        "name": "john-avatar.jpg",
        "url": "/uploads/john-avatar.jpg"
      },
      "articles": [
        { "id": 1, "title": "Article 1" },
        { "id": 5, "title": "Article 2" }
      ],
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ]
}
```

#### Create Author

```bash
POST /api/authors
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "data": {
    "name": "Jane Smith",
    "email": "jane@example.com",
    "bio": "Contributing writer"
  }
}
```

---

### Categories API

#### List Categories

```bash
GET /api/categories
```

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "documentId": "uuid-cat-1",
      "name": "Technology",
      "slug": "technology",
      "description": "Technology news and updates"
    },
    {
      "id": 2,
      "documentId": "uuid-cat-2",
      "name": "Business",
      "slug": "business"
    }
  ]
}
```

#### Create Category

```bash
POST /api/categories
Authorization: Bearer YOUR_TOKEN

{
  "data": {
    "name": "Sports",
    "slug": "sports",
    "description": "Sports related content"
  }
}
```

---

### Global Settings API

Global settings is a **singleton type** (single instance):

#### Get Global Settings

```bash
GET /api/global?populate=*
```

**Response:**
```json
{
  "data": {
    "id": 1,
    "documentId": "uuid-global",
    "siteName": "My Website",
    "siteDescription": "A great website",
    "logo": {
      "url": "/uploads/logo.png"
    },
    "socialLinks": [
      { "platform": "twitter", "url": "https://twitter.com/..." },
      { "platform": "facebook", "url": "https://facebook.com/..." }
    ],
    "contactEmail": "contact@example.com"
  }
}
```

#### Update Global Settings

```bash
PUT /api/global/1
Authorization: Bearer YOUR_TOKEN

{
  "data": {
    "siteName": "Updated Site Name",
    "contactEmail": "newemail@example.com"
  }
}
```

---

### About Page API

#### Get About Content

```bash
GET /api/about?populate=*
```

**Response:**
```json
{
  "data": {
    "id": 1,
    "documentId": "uuid-about",
    "title": "About Us",
    "content": "<p>Company description...</p>",
    "images": [
      {
        "id": 1,
        "name": "team.jpg",
        "url": "/uploads/team.jpg"
      }
    ]
  }
}
```

#### Update About Page

```bash
PUT /api/about/1
Authorization: Bearer YOUR_TOKEN

{
  "data": {
    "title": "About Our Company",
    "content": "<p>Updated content...</p>"
  }
}
```

---

## Component Endpoints

Components are nested within content types and accessed through population.

### Media Component

Accessed via `populate=media` on articles, authors, etc.

```json
{
  "media": {
    "id": 1,
    "url": "/uploads/image.jpg",
    "name": "image.jpg",
    "alternativeText": "Alt text",
    "caption": "Image caption",
    "width": 800,
    "height": 600,
    "mimetype": "image/jpeg"
  }
}
```

### SEO Component

Accessed via `populate=seo` on articles, about page, etc.

```json
{
  "seo": {
    "metaTitle": "Page Title | Site",
    "metaDescription": "Page description for search engines",
    "keywords": "keyword1, keyword2",
    "canonicalUrl": "https://example.com/page"
  }
}
```

### Rich Text Component

```json
{
  "richText": "<p>Formatted content</p><ul><li>Item 1</li></ul>"
}
```

### Slider Component

```json
{
  "slider": {
    "slides": [
      {
        "id": 1,
        "image": { "url": "/uploads/slide1.jpg" },
        "title": "Slide 1",
        "description": "First slide"
      },
      {
        "id": 2,
        "image": { "url": "/uploads/slide2.jpg" },
        "title": "Slide 2"
      }
    ]
  }
}
```

---

## User & Permission Endpoints

### Create User

```bash
POST /api/auth/local/register
Content-Type: application/json

{
  "username": "newuser",
  "email": "user@example.com",
  "password": "securepassword123"
}
```

### User Login

```bash
POST /api/auth/local
Content-Type: application/json

{
  "identifier": "user@example.com",
  "password": "securepassword123"
}
```

**Response:**
```json
{
  "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 2,
    "username": "newuser",
    "email": "user@example.com",
    "confirmed": true
  }
}
```

### Get Current User

```bash
GET /api/users/me
Authorization: Bearer YOUR_TOKEN
```

**Response:**
```json
{
  "id": 1,
  "username": "admin",
  "email": "admin@example.com",
  "confirmed": true,
  "blocked": false
}
```

---

## Error Responses

### 400 Bad Request

```json
{
  "data": null,
  "error": {
    "status": 400,
    "name": "BadRequestError",
    "message": "Invalid request",
    "details": {
      "errors": [
        {
          "path": ["title"],
          "message": "This field is required"
        }
      ]
    }
  }
}
```

### 401 Unauthorized

```json
{
  "data": null,
  "error": {
    "status": 401,
    "name": "UnauthorizedError",
    "message": "Invalid token"
  }
}
```

**Fix:** Ensure token is valid and not expired
```bash
Authorization: Bearer YOUR_VALID_TOKEN
```

### 403 Forbidden

```json
{
  "data": null,
  "error": {
    "status": 403,
    "name": "ForbiddenError",
    "message": "You do not have permission to access this resource"
  }
}
```

### 404 Not Found

```json
{
  "data": null,
  "error": {
    "status": 404,
    "name": "NotFoundError",
    "message": "Article not found"
  }
}
```

### 500 Internal Server Error

```json
{
  "data": null,
  "error": {
    "status": 500,
    "name": "InternalServerError",
    "message": "An internal error occurred"
  }
}
```

---

## Common Use Cases

### Get Published Articles with Author Info

```bash
GET /api/articles?filters[publishedAt][$notNull]=true&populate[author][populate]=*&sort=publishedAt:desc&pagination[pageSize]=10
```

### Search Articles by Title or Category

```bash
GET /api/articles?filters[$or][0][title][$contains]=search&filters[$or][1][category][slug]=technology
```

### Get Articles by Specific Author

```bash
GET /api/articles?filters[author][id]=1&populate=*
```

### Paginate Through All Articles

```bash
# Page 1
GET /api/articles?pagination[page]=1&pagination[pageSize]=20

# Page 2
GET /api/articles?pagination[page]=2&pagination[pageSize]=20

# Page 3
GET /api/articles?pagination[page]=3&pagination[pageSize]=20
```

### Get Latest 5 Articles

```bash
GET /api/articles?sort=publishedAt:desc&pagination[pageSize]=5
```

### Upload Media File

```bash
POST /api/upload
Authorization: Bearer YOUR_TOKEN
Content-Type: multipart/form-data

[Binary file data]
```

**Response:**
```json
[
  {
    "id": 1,
    "name": "image.jpg",
    "url": "/uploads/image.jpg",
    "size": 102400,
    "width": 800,
    "height": 600,
    "mimetype": "image/jpeg"
  }
]
```

---

## Rate Limiting

Default rate limiting (if enabled):
- **Global**: 100 requests per minute per IP
- **Auth**: 5 failed attempts per 15 minutes

To check rate limit headers in response:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1643280000
```

---

## Pagination Response Examples

### Single Page Response

```json
{
  "data": [{ "id": 1, ... }, { "id": 2, ... }],
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 25,
      "pageCount": 1,
      "total": 15
    }
  }
}
```

### Multiple Pages Available

```json
{
  "data": [{ ... 25 items ... }],
  "meta": {
    "pagination": {
      "page": 2,
      "pageSize": 25,
      "pageCount": 5,
      "total": 120
    }
  }
}
```

---

## Testing API Endpoints

### Using cURL

```bash
# List all articles
curl http://localhost:1337/api/articles

# Create article (with token)
curl -X POST http://localhost:1337/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"data":{"title":"Test"}}'

# Filter by category
curl "http://localhost:1337/api/articles?filters[category]=5"
```

### Using Postman

1. Import Collection from Swagger docs
2. Set environment variable: `{{baseUrl}}` = `http://localhost:1337/api`
3. Authenticate: Copy JWT to `Authorization` header
4. Make requests with populated fields

### Using JavaScript Fetch

```javascript
// List articles
const response = await fetch('/api/articles?populate=*');
const { data, meta } = await response.json();

// Create article
const created = await fetch('/api/articles', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    data: { title: 'New Article' }
  })
});
```

---

**Last Updated:** January 2026  
**API Version:** Strapi 5.33.4  
**Endpoint Base:** `http://localhost:1337/api`
