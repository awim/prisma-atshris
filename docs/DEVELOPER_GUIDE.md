# 👨‍💻 Developer Starter Guide - Prisma Strapi

Welcome to the **Prisma Strapi** project! This guide will help you set up and run the project locally.

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Environment Setup](#environment-setup)
5. [Running the Project](#running-the-project)
6. [Project Structure](#project-structure)
7. [Available Scripts](#available-scripts)
8. [Common Tasks](#common-tasks)
9. [Troubleshooting](#troubleshooting)
10. [Additional Resources](#additional-resources)

---

## 🎯 Project Overview

**Prisma Strapi** is a system integration middleware built on top of Strapi 5.33.4. It serves as a headless CMS with a robust API for managing content types including articles, authors, categories, global settings, and more.

**Key Technologies:**
- **Strapi 5.33.4** - Headless CMS and Content Management
- **Node.js 24** - JavaScript runtime (supports 20.0.0 to 24.x.x)
- **Yarn** - Package manager
- **SQLite** - Default database (configurable to MySQL)
- **TypeScript** - Type-safe development
- **React 18** - Admin panel UI

**Main Features:**
- RESTful API for content management
- Multi-content type support (Articles, Authors, Categories, About, Global)
- Component-based architecture (Media, Rich Text, SEO, Slider, Quote)
- Media upload management
- User permissions system
- Cloud deployment capabilities

---

## 📦 Prerequisites

Before you begin, ensure you have the following installed on your system:

### Required
- **Node.js**: Version 20.0.0 to 24.x.x
  - Check version: `node --version`
  - Download: https://nodejs.org/
  
- **Yarn**: Package manager (recommended over npm)
  - Install: `npm install -g yarn`
  - Check version: `yarn --version`

### Recommended
- **Git**: Version control
  - Download: https://git-scm.com/
  
- **VS Code**: Code editor with TypeScript support
  - Download: https://code.visualstudio.com/

- **Postman or Insomnia**: API testing tools
  - Postman: https://www.postman.com/
  - Insomnia: https://insomnia.rest/

---

## 🚀 Installation

### 1. Clone the Repository

```bash
cd your-workspace-path
git clone <repository-url>
cd prisma-strapi
```

### 2. Install Dependencies

```bash
yarn install
```

This will install all required dependencies listed in `package.json`.

**Verification**: The command should complete without errors and create a `node_modules` folder.

### 3. Verify Installation

Check that key packages are installed:

```bash
yarn list @strapi/strapi
node --version  # Should be 20.0.0 - 24.x.x
yarn --version
```

---

## ⚙️ Environment Setup

### 1. Environment Variables File

The project uses a `.env` file for configuration. A template is provided:

**Location**: `d:\wProject\prisma-strapi\.env`

### 2. Default Configuration

The `.env` file includes:

```env
# Server Configuration
HOST=0.0.0.0          # Listen on all interfaces
PORT=1337             # Default Strapi port

# Security Keys (Pre-configured)
APP_KEYS=...
API_TOKEN_SALT=...
ADMIN_JWT_SECRET=...
TRANSFER_TOKEN_SALT=...
ENCRYPTION_KEY=...
JWT_SECRET=...

# Database Configuration
DATABASE_CLIENT=sqlite       # Current: SQLite (lightweight, no setup needed)
DATABASE_FILENAME=.tmp/data.db
# For MySQL, uncomment and configure:
# DATABASE_HOST=localhost
# DATABASE_PORT=3306
# DATABASE_NAME=strapi
# DATABASE_USERNAME=strapi
# DATABASE_PASSWORD=your-password
```

### 3. Configuration Options

**For Local Development (SQLite - Default)**
- No additional setup needed
- Database file stored at `.tmp/data.db`
- Best for quick local testing

**For MySQL Database**
Edit `.env` to use MySQL:
```env
DATABASE_CLIENT=mysql
DATABASE_HOST=localhost
DATABASE_PORT=3306
DATABASE_NAME=your_db_name
DATABASE_USERNAME=your_user
DATABASE_PASSWORD=your_password
DATABASE_SSL=false
```

### 4. Security Note

⚠️ **IMPORTANT**: The security keys in `.env` are for development only. For production:
- Generate new keys using Strapi
- Store sensitive values in secure vaults (AWS Secrets Manager, HashiCorp Vault)
- Never commit `.env` to version control (already in `.gitignore`)

---

## ▶️ Running the Project

### Start Development Server

```bash
yarn dev
```

or

```bash
yarn develop
```

> default admin credentials:
>
> | username | password |
> | - | - |
> | `support+strap1@innovatech-s.com` | `SUPPORT+strap1@innovatech-s.com` |

**Expected Output:**
```
[2026-01-28 10:30:00.000] info: Strapi is starting...
[2026-01-28 10:30:05.123] info: Server is running on http://0.0.0.0:1337
```

### Access the Application

- **Admin Panel**: http://localhost:1337/admin
- **API Endpoint**: http://localhost:1337/api
- **API Documentation**: http://localhost:1337/api/documentation (if enabled)

### Initial Admin Setup

On first run, Strapi will prompt you to create an admin account. Follow the prompts to:
1. Enter admin email
2. Set password
3. Create initial account

**Default Access**:
- Email: `admin@example.com`
- Password: (as you set during setup)

---

## 📁 Project Structure

```
prisma-strapi/
├── config/                          # Strapi Configuration
│   ├── admin.ts                    # Admin panel settings
│   ├── api.ts                      # API configuration
│   ├── database.ts                 # Database config (SQLite/MySQL)
│   ├── middlewares.ts              # Express middlewares
│   ├── plugins.ts                  # Plugin configuration
│   └── server.ts                   # Server settings
│
├── src/
│   ├── index.ts                    # Application entry point
│   ├── api/                        # Content Types & Controllers
│   │   ├── about/                  # About page content
│   │   ├── article/                # Article content type
│   │   ├── author/                 # Author content type
│   │   ├── category/               # Category content type
│   │   └── global/                 # Global settings
│   │
│   ├── components/                 # Reusable Components
│   │   └── shared/
│   │       ├── media.json          # Media component
│   │       ├── quote.json          # Quote component
│   │       ├── rich-text.json      # Rich text editor
│   │       ├── seo.json            # SEO meta component
│   │       └── slider.json         # Image slider
│   │
│   ├── admin/                      # Custom Admin Panel
│   │   └── app.example.tsx         # React admin component example
│   │
│   └── extensions/                 # Plugin extensions

├── types/
│   └── generated/                  # Auto-generated TypeScript types
│       ├── components.d.ts         # Component types
│       └── contentTypes.d.ts       # Content type definitions

├── database/
│   └── migrations/                 # Database migrations

├── scripts/
│   └── seed.js                     # Database seeding script

├── data/
│   ├── data.json                   # Sample/fixture data
│   └── uploads/                    # Uploaded media files

├── public/
│   ├── robots.txt                  # SEO robots file
│   └── uploads/                    # Public media uploads

├── .env                            # Environment variables
├── package.json                    # Dependencies & scripts
├── tsconfig.json                   # TypeScript configuration
└── README.md                       # Quick start guide
```

### Key Directories Explained

| Directory | Purpose |
|-----------|---------|
| `src/api/` | Define content types, routes, controllers, and services |
| `src/components/` | Reusable component schemas (fields, relations) |
| `src/admin/` | Customize the admin panel UI |
| `config/` | Application-wide configuration |
| `.tmp/` | Temporary files, SQLite database |
| `public/` | Static files served publicly |

---

## 📝 Available Scripts

### Development

```bash
# Start development server with auto-reload
yarn dev
yarn develop

# Start production server
yarn start

# Open Strapi console (interactive terminal)
yarn console
```

### Building & Deployment

```bash
# Build the admin panel
yarn build

# Deploy to Strapi Cloud
yarn deploy

# Upgrade Strapi to latest version
yarn upgrade

# Dry run upgrade (preview changes)
yarn upgrade:dry
```

### Database

```bash
# Run seeding script
yarn seed:example
```

---

## 🛠️ Common Tasks

### 1. Add a New Content Type

1. Navigate to **Admin Panel** → **Content-Type Builder**
2. Click **Create new content type**
3. Define fields (title, description, etc.)
4. Click **Save**

Auto-generated files will be created in `src/api/[content-type]/`

### 2. Create a New API Endpoint

**File Structure for new content type `product`:**
```
src/api/product/
├── content-types/
│   └── product/
│       └── schema.json
├── controllers/
│   └── product.ts
├── routes/
│   └── product.ts
└── services/
    └── product.ts
```

**Example controller** (`src/api/product/controllers/product.ts`):
```typescript
import { factories } from '@strapi/strapi';

export default factories.createCoreController('api::product.product', {
  async find(ctx) {
    const { data, meta } = await super.find(ctx);
    return { data, meta };
  }
});
```

### 3. Seed Database with Example Data

```bash
yarn seed:example
```

Modify `scripts/seed.js` to add custom seeding logic.

### 4. Query API

**Get all articles:**
```bash
curl http://localhost:1337/api/articles
```

**Get single article with relations:**
```bash
curl "http://localhost:1337/api/articles/1?populate=*"
```

**Create article (authenticated):**
```bash
curl -X POST http://localhost:1337/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"data":{"title":"New Article"}}'
```

### 5. Generate Types

TypeScript types are auto-generated in `types/generated/`:
```bash
yarn build  # Regenerates types
```

Use them in your code:
```typescript
import type { ContentTypeComponents } from '#types/generated/contentTypes';
```

---

## 🐛 Troubleshooting

### Issue: Port 1337 Already in Use

**Solution**: Change port in `.env`
```env
PORT=3000  # or any available port
```

Then restart:
```bash
yarn dev
```

### Issue: Database Lock Error (SQLite)

**Cause**: Multiple processes accessing the database simultaneously.

**Solutions**:
1. Ensure only one instance is running
2. Delete `.tmp/data.db` and restart (data will be lost)
3. Switch to MySQL for concurrent access

```bash
# Stop all Strapi processes
# Delete database
rm .tmp/data.db

# Restart
yarn dev
```

### Issue: Node Version Mismatch

**Error**: `The engine "node" is incompatible with this package`

**Solution**:
```bash
# Install correct Node version (20-24)
nvm install 24.0.0
nvm use 24.0.0

# Verify
node --version
```

### Issue: Dependencies Not Installing

```bash
# Clear cache and reinstall
yarn cache clean
rm -rf node_modules
yarn install
```

### Issue: Admin Panel Not Loading

**Solutions**:
1. Clear browser cache (Ctrl+Shift+Delete)
2. Hard reload (Ctrl+Shift+R)
3. Check browser console for errors (F12)
4. Ensure server is running: `yarn dev`

### Issue: Permission Denied on Windows

If you get permission errors:
```powershell
# Run PowerShell as Administrator
# Then retry: yarn install
```

---

## 📚 Additional Resources

### Official Documentation
- [Strapi Documentation](https://docs.strapi.io/)
- [Strapi API Reference](https://docs.strapi.io/dev-docs/api/rest)
- [TypeScript Setup](https://docs.strapi.io/dev-docs/typescript)

### Community & Support
- [Strapi Forum](https://forum.strapi.io/)
- [GitHub Discussions](https://github.com/strapi/strapi/discussions)
- [Strapi Community Slack](https://slack.strapi.io/)

### Useful Tools
- [Strapi Cloud Dashboard](https://cloud.strapi.io/)
- [RESTful API Tester](https://www.postman.com/)
- [Database GUI](https://www.dbeavertools.com/) - For SQLite/MySQL

### Learning Resources
- [Strapi Tutorials](https://strapi.io/tutorials)
- [Strapi Blog](https://strapi.io/blog)
- [REST API Best Practices](https://restfulapi.net/)

---

## ✅ Quick Start Checklist

- [ ] Install Node.js (version 20-24)
- [ ] Install Yarn globally
- [ ] Clone the repository
- [ ] Run `yarn install`
- [ ] Review and confirm `.env` configuration
- [ ] Run `yarn dev`
- [ ] Access http://localhost:1337/admin
- [ ] Create admin account
- [ ] Explore content types and API endpoints
- [ ] Read through `/docs` for architecture details

---

## 💡 Tips for Success

1. **Start Simple**: Explore existing content types before creating new ones
2. **Use Admin Panel**: Learn through the UI before diving into code
3. **Check Logs**: Monitor terminal output for helpful error messages
4. **Test APIs**: Use Postman/Insomnia to test endpoints
5. **Read Schema Files**: `schema.json` files define your data structure
6. **Use TypeScript**: Leverage generated types for safer development
7. **Keep Security**: Never commit sensitive data to version control
8. **Document Changes**: Update schema files when modifying content types

---

## 🤝 Getting Help

If you encounter issues:

1. Check **Troubleshooting** section above
2. Review [Strapi Docs](https://docs.strapi.io/)
3. Search [GitHub Issues](https://github.com/strapi/strapi/issues)
4. Ask in [Strapi Forum](https://forum.strapi.io/)
5. Contact your team lead

---

**Last Updated**: January 2026  
**Strapi Version**: 5.33.4  
**Node Version**: 20.0.0 - 24.x.x
