# ⚡ Quick Reference Cheat Sheet

Fast lookup guide for common tasks and commands.

## 🚀 Common Commands

```bash
# Installation & Setup
yarn install              # Install dependencies
yarn dev                  # Start development server
yarn build               # Build admin panel
yarn start               # Start production server

# Database
yarn seed:example        # Run database seeding
yarn strapi migrate      # Run migrations

# Admin & Utilities
yarn console             # Open Strapi console
yarn strapi              # Show all Strapi commands
yarn upgrade             # Upgrade Strapi version
```

## 🌐 Quick URLs

| Resource | URL |
|----------|-----|
| **Admin Panel** | http://localhost:1337/admin |
| **API Base** | http://localhost:1337/api |
| **API Docs** | http://localhost:1337/documentation |
| **Articles** | http://localhost:1337/api/articles |
| **Authors** | http://localhost:1337/api/authors |
| **Categories** | http://localhost:1337/api/categories |

## 📁 Key File Locations

| What | Where |
|------|-------|
| Content type schema | `src/api/[type]/content-types/[type]/schema.json` |
| Controller logic | `src/api/[type]/controllers/[type].ts` |
| Service methods | `src/api/[type]/services/[type].ts` |
| API routes | `src/api/[type]/routes/[type].ts` |
| Components | `src/components/shared/` |
| Server config | `config/server.ts` |
| Database config | `config/database.ts` |
| Environment vars | `.env` |
| TypeScript types | `types/generated/` |

## 🔑 Environment Variables (Key Ones)

```env
HOST=0.0.0.0           # Server host
PORT=1337              # Server port
NODE_ENV=development   # Environment
DATABASE_CLIENT=sqlite # Database type
DATABASE_FILENAME=.tmp/data.db  # SQLite path
```

## 📡 API Query Examples

### List & Filter
```bash
# Get all articles
GET /api/articles

# With populated relations
GET /api/articles?populate=*

# Filter by category
GET /api/articles?filters[category]=5

# Search in title
GET /api/articles?filters[title][$contains]=news

# Published articles only
GET /api/articles?filters[publishedAt][$notNull]=true

# Sort descending
GET /api/articles?sort=publishedAt:desc

# Paginate
GET /api/articles?pagination[pageSize]=10&pagination[page]=2
```

### CRUD Operations
```bash
# Create
POST /api/articles
Authorization: Bearer TOKEN
{ "data": { "title": "New" } }

# Update
PUT /api/articles/1
Authorization: Bearer TOKEN
{ "data": { "title": "Updated" } }

# Delete
DELETE /api/articles/1
Authorization: Bearer TOKEN
```

## 🔐 Authentication

```bash
# Get token
POST /api/auth/local
{ "identifier": "admin@example.com", "password": "xxx" }

# Use token in requests
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

## 🐛 Troubleshooting Quick Fixes

| Issue | Fix |
|-------|-----|
| Port already in use | Change `PORT` in `.env` |
| Dependencies missing | `yarn cache clean && yarn install` |
| Database locked | Delete `.tmp/data.db` and restart |
| Admin not loading | Hard refresh browser (Ctrl+Shift+R) |
| Token expired | Get new token from `/api/auth/local` |
| Query returns empty | Add `populate=*` to get relations |

## 📦 Project Structure (Shortened)

```
src/api/
├── article/        # Article content type
├── author/         # Author content type  
├── category/       # Category content type
├── about/          # About page
└── global/         # Global settings

src/components/shared/
├── media.json      # Images/files
├── rich-text.json  # Formatted text
├── seo.json        # Meta tags
├── slider.json     # Image carousels
└── quote.json      # Testimonials

config/
├── server.ts       # Server config
├── database.ts     # Database config
├── middlewares.ts  # Middleware
├── plugins.ts      # Plugins
└── api.ts          # API config
```

## 🎯 Content Type Endpoints

```
GET    /api/articles              # List
GET    /api/articles/1            # Get by ID
POST   /api/articles              # Create
PUT    /api/articles/1            # Update
DELETE /api/articles/1            # Delete

GET    /api/authors               # List authors
GET    /api/categories            # List categories
GET    /api/global                # Get global settings
GET    /api/about                 # Get about page
```

## 🔗 Useful Links

- [Official Docs](https://docs.strapi.io/)
- [REST API Guide](https://docs.strapi.io/dev-docs/api/rest)
- [Community Forum](https://forum.strapi.io/)
- [GitHub Repo](https://github.com/strapi/strapi)

## 🎓 Related Documents

- [Full Developer Guide](./DEVELOPER_GUIDE.md)
- [Architecture Details](./ARCHITECTURE.md)
- [API Reference](./API_REFERENCE.md)
- [Setup & Config](./SETUP_CONFIG.md)
- [Documentation Index](./README.md)

---

**Pro Tips:**
- Use `?populate=*` to get all relations in one request
- Add `?sort=createdAt:desc` to show newest first
- Use `?pagination[pageSize]=5` for smaller pages
- Always include `Authorization: Bearer TOKEN` for write operations
- Check response `meta.pagination` to know total items

