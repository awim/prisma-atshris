# ⚙️ Setup & Configuration Guide

Detailed configuration instructions for different environments and scenarios.

## Table of Contents

1. [Environment Variables](#environment-variables)
2. [Database Configuration](#database-configuration)
3. [Server Configuration](#server-configuration)
4. [Security Configuration](#security-configuration)
5. [Admin Panel Setup](#admin-panel-setup)
6. [Plugin Configuration](#plugin-configuration)
7. [Development Tools Setup](#development-tools-setup)
8. [Production Deployment](#production-deployment)

---

## Environment Variables

### Location
`.env` file in project root (automatically loaded by Strapi)

### Variable Reference

#### Server Configuration

```env
# Public-facing host
HOST=0.0.0.0

# Server port
PORT=1337

# URL prefix for API (optional)
URL_PREFIX=/api

# Admin panel path
ADMIN_PATH=/admin
```

#### Security Keys (Auto-generated, don't change)

```env
# Encryption keys for app functionality
APP_KEYS=key1,key2,key3,key4

# Salt for API token hashing
API_TOKEN_SALT=xxxx

# JWT secret for admin authentication
ADMIN_JWT_SECRET=xxxx

# Salt for transfer tokens
TRANSFER_TOKEN_SALT=xxxx

# General encryption key
ENCRYPTION_KEY=xxxx

# JWT secret for user authentication
JWT_SECRET=xxxx
```

#### Database Configuration

```env
# Database type: sqlite, mysql, postgres
DATABASE_CLIENT=sqlite

# SQLite specific
DATABASE_FILENAME=.tmp/data.db

# MySQL/Postgres connection details
DATABASE_HOST=localhost
DATABASE_PORT=3306
DATABASE_NAME=strapi
DATABASE_USERNAME=strapi
DATABASE_PASSWORD=password

# SSL configuration
DATABASE_SSL=false
DATABASE_SSL_KEY=
DATABASE_SSL_CERT=
DATABASE_SSL_CA=
DATABASE_SSL_REJECT_UNAUTHORIZED=true
```

#### Optional Features

```env
# Email provider
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USERNAME=noreply@example.com
SMTP_PASSWORD=password

# S3/Object storage (for production file uploads)
AWS_ACCESS_KEY_ID=xxxx
AWS_SECRET_ACCESS_KEY=xxxx
AWS_REGION=us-east-1
AWS_BUCKET=my-strapi-bucket

# Analytics/Monitoring
SENTRY_DSN=https://xxxxx@sentry.io/xxxxxx

# Node environment
NODE_ENV=development  # or production
```

---

## Database Configuration

### SQLite (Development - Default)

**Configuration:** `.env`
```env
DATABASE_CLIENT=sqlite
DATABASE_FILENAME=.tmp/data.db
```

**Pros:**
- Zero setup required
- Perfect for local development
- File-based, easy to backup
- No separate server needed

**Cons:**
- Single user only
- Poor concurrency support
- Limited for production
- Not suitable for team development

**Backup:**
```bash
# Copy database file
cp .tmp/data.db ./backups/data-backup-$(date +%Y%m%d).db
```

**Reset:**
```bash
# Delete database to start fresh
rm .tmp/data.db

# Restart server
yarn dev
```

### MySQL (Recommended for Production)

**Prerequisites:**
```bash
# Install MySQL Server
# Windows: https://dev.mysql.com/downloads/mysql/
# macOS: brew install mysql
# Linux: sudo apt-get install mysql-server
```

**Create Database:**
```sql
-- Connect to MySQL
mysql -u root -p

-- Create database
CREATE DATABASE strapi_prod CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user
CREATE USER 'strapi_user'@'localhost' IDENTIFIED BY 'strong_password';

-- Grant permissions
GRANT ALL PRIVILEGES ON strapi_prod.* TO 'strapi_user'@'localhost';
FLUSH PRIVILEGES;

-- Exit
EXIT;
```

**Configuration:** `.env`
```env
DATABASE_CLIENT=mysql
DATABASE_HOST=localhost
DATABASE_PORT=3306
DATABASE_NAME=strapi_prod
DATABASE_USERNAME=strapi_user
DATABASE_PASSWORD=strong_password
DATABASE_SSL=false
```

**Connection Pool Settings** (`config/database.ts`):
```typescript
export default ({ env }) => ({
  connection: {
    // ... connection details
    acquireConnectionTimeout: 30000,  // 30 seconds
    pool: {
      min: 2,
      max: 10,  // Maximum 10 concurrent connections
      acquireTimeoutMillis: 30000,
      idleTimeoutMillis: 30000,
      reapIntervalMillis: 1000
    }
  }
});
```

**Backup:**
```bash
# Export database
mysqldump -u strapi_user -p strapi_prod > backup.sql

# Restore from backup
mysql -u strapi_user -p strapi_prod < backup.sql
```

### PostgreSQL (Alternative)

**Configuration:** `.env`
```env
DATABASE_CLIENT=postgres
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=strapi
DATABASE_USERNAME=strapi
DATABASE_PASSWORD=password
DATABASE_SSL=false
```

---

## Server Configuration

### File: `config/server.ts`

```typescript
export default ({ env }) => ({
  // Server binding
  host: env('HOST', '0.0.0.0'),  // 0.0.0.0 = all interfaces
  port: env.int('PORT', 1337),   // 1337 = default Strapi port

  // Application encryption keys
  app: {
    keys: env.array('APP_KEYS'),  // Must have 4 keys
  },

  // HTTP compression
  compression: {
    enabled: true,
    gzip: {
      level: 6
    }
  },

  // HTTPS/SSL (for production)
  https: env.bool('HTTPS_ENABLED', false),
  ssl: {
    key: env('SSL_KEY_PATH', ''),
    cert: env('SSL_CERT_PATH', '')
  },

  // Admin panel customization
  admin: {
    autoOpen: false,  // Don't auto-open browser
    watchIgnoreFiles: ['**/node_modules/**'],  // Don't watch these
  },

  // Webhook configuration
  webhooks: {
    populateRelations: env.bool('WEBHOOKS_POPULATE_RELATIONS', false),
  }
});
```

### Changing Server Port

```env
PORT=3000
```

Then access at: `http://localhost:3000/admin`

### HTTPS Configuration

For production HTTPS:

```typescript
// config/server.ts
export default ({ env }) => ({
  https: true,
  ssl: {
    key: env('SSL_KEY_PATH'),     // Path to private key
    cert: env('SSL_CERT_PATH')    // Path to certificate
  }
});
```

---

## Security Configuration

### File: `config/admin.ts`

```typescript
export default ({ env }) => ({
  auth: {
    secret: env('ADMIN_JWT_SECRET'),
    options: {
      expiresIn: '7d',  // Admin token expiration
    }
  },

  apiToken: {
    salt: env('API_TOKEN_SALT'),
  },

  // CORS settings
  cors: {
    enabled: true,
    origin: env.array('ADMIN_CORS_ORIGIN', [
      'http://localhost:3000',
      'http://localhost:8000'
    ]),
    credentials: true,
  },

  // Rate limiting
  rateLimit: {
    enabled: true,
    max: 100,              // Max requests
    windowMs: 60 * 1000,   // Per 1 minute
    skipSuccessfulRequests: false
  }
});
```

### File: `config/middlewares.ts`

```typescript
export default [
  'strapi::errors',
  'strapi::security',  // Helmet security headers
  'strapi::cors',
  'strapi::poweredBy',
  'strapi::logger',
  'strapi::query',
  'strapi::body',
  'strapi::session',
  'strapi::favicon',
  'strapi::public',
  
  // Custom middleware
  {
    name: 'strapi::cors',
    config: {
      enabled: true,
      origin: process.env.CORS_ORIGIN?.split(',') || ['*'],
      credentials: true,
      methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'HEAD', 'OPTIONS'],
      headers: ['Content-Type', 'Authorization'],
      maxAge: 31536000
    }
  }
];
```

### JWT Configuration

```typescript
// config/api.ts
export default {
  rest: {
    defaultLimit: 25,
    maxLimit: 100
  },
  
  auth: {
    strategies: {
      'jwt': {
        source: 'header',
        identifier: 'Authorization',
        scheme: 'bearer'
      }
    }
  }
};
```

---

## Admin Panel Setup

### First Time Admin Creation

**On first run**, Strapi prompts for admin account:

1. Open http://localhost:1337/admin
2. Fill in form:
   - Email: `admin@example.com`
   - Password: Create secure password
   - First name: Your name
3. Click **Let's start**

### Subsequent Admin Logins

```
Email: admin@example.com
Password: (as set during first setup)
```

### Create Additional Admin Users

1. Login as existing admin
2. Go: **Settings** → **Users & Permissions** → **Users**
3. Click **Invite new user**
4. Enter email
5. Select role: **Super Admin** or custom role
6. Send invitation

### Role-Based Access Control

**Predefined Roles:**
- **Super Admin**: Full access to everything
- **Editor**: Can manage content types, create/edit content
- **Author**: Can create and edit own content only
- **Contributor**: Limited permissions, can draft content

**Create Custom Role:**
1. Settings → **Roles**
2. Click **Create**
3. Define permissions per content type
4. Assign to users

---

## Plugin Configuration

### File: `config/plugins.ts`

```typescript
export default ({ env }) => ({
  // User permissions plugin
  'users-permissions': {
    enabled: true,
    config: {
      jwt: {
        expiresIn: '7d',
      },
      jwtSecret: env('JWT_SECRET'),
      defaultRole: 'authenticated',
      enableEachApi: true,
      routes: {
        rolesFindAll: '/roles',
        rolesFindOne: '/roles/:id',
        createRole: '/roles',
        updateRole: '/roles/:id',
        deleteRole: '/roles/:id'
      }
    }
  },

  // Cloud plugin
  'cloud': {
    enabled: true
  },

  // Custom upload provider (optional)
  upload: {
    config: {
      provider: 'local',  // or 's3', 'cloudinary', etc.
      providerOptions: {
        sizeLimit: 100000000  // 100MB
      }
    }
  },

  // Email provider
  email: {
    config: {
      provider: 'sendgrid',
      providerOptions: {
        apiKey: env('SENDGRID_API_KEY')
      },
      settings: {
        defaultFrom: 'noreply@example.com',
        defaultReplyTo: 'support@example.com'
      }
    }
  },

  // GraphQL (optional)
  graphql: {
    enabled: true,
    config: {
      endpoint: '/graphql',
      shadowCRUD: true,
      playgroundAlways: env('NODE_ENV') === 'development'
    }
  }
});
```

---

## Development Tools Setup

### ESLint & TypeScript

**Initialize ESLint:**
```bash
yarn add -D eslint eslint-config-strapi
echo "extends: 'eslapi'" > .eslintrc.js
```

**Run linting:**
```bash
yarn lint
yarn lint --fix  # Auto-fix issues
```

### Debugging

**VS Code Launch Configuration** (`.vscode/launch.json`):
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Strapi Debug",
      "program": "${workspaceFolder}/node_modules/@strapi/strapi/lib/commands/develop.js",
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "restart": true,
      "protocol": "inspector"
    }
  ]
}
```

**Debugging in Terminal:**
```bash
# Run with debugging enabled
NODE_OPTIONS=--inspect yarn dev

# Then attach debugger to chrome://inspect
```

### API Testing Tools

**Postman Setup:**
1. Download [Postman](https://www.postman.com)
2. Create workspace
3. Import Swagger docs: `http://localhost:1337/documentation`
4. Set variables:
   ```
   baseUrl = http://localhost:1337/api
   token = your_api_token
   ```

**Insomnia Setup:**
1. Download [Insomnia](https://insomnia.rest)
2. Create HTTP Request
3. Set URL: `http://localhost:1337/api/articles`
4. Set Headers: `Authorization: Bearer YOUR_TOKEN`

---

## Production Deployment

### Pre-Deployment Checklist

```
[ ] Update NODE_ENV to "production"
[ ] Generate new APP_KEYS for production
[ ] Setup MySQL or PostgreSQL database
[ ] Configure S3/CDN for file uploads
[ ] Enable HTTPS with valid certificates
[ ] Setup email provider (SendGrid, etc.)
[ ] Configure security headers
[ ] Enable rate limiting
[ ] Setup monitoring (Sentry, NewRelic)
[ ] Test all API endpoints
[ ] Backup database
[ ] Document deployment procedure
```

### Environment Setup

```env
NODE_ENV=production
HOST=0.0.0.0
PORT=1337

# Database - use cloud-hosted MySQL
DATABASE_CLIENT=mysql
DATABASE_HOST=prod-db.example.com
DATABASE_NAME=strapi_prod
DATABASE_USERNAME=strapi_prod_user
DATABASE_PASSWORD=VERY_SECURE_PASSWORD

# HTTPS
HTTPS_ENABLED=true
SSL_KEY_PATH=/etc/ssl/private/key.pem
SSL_CERT_PATH=/etc/ssl/certs/cert.pem

# New security keys (generate via Strapi)
APP_KEYS=newkey1,newkey2,newkey3,newkey4
API_TOKEN_SALT=newsalt1
ADMIN_JWT_SECRET=newsecret1
JWT_SECRET=newsecret2

# File uploads - use S3
AWS_ACCESS_KEY_ID=xxxx
AWS_SECRET_ACCESS_KEY=xxxx
AWS_REGION=us-east-1
AWS_BUCKET=prod-strapi-uploads

# Email
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USERNAME=apikey
SMTP_PASSWORD=SG.xxxxx
```

### Build for Production

```bash
# Install production dependencies
yarn install --production

# Build admin panel
yarn build

# Start server
yarn start
```

### Using PM2 for Process Management

```bash
# Install PM2
npm install -g pm2

# Create ecosystem config (ecosystem.config.js)
module.exports = {
  apps: [{
    name: 'strapi',
    script: './node_modules/@strapi/strapi/lib/commands/start.js',
    instances: 'max',
    exec_mode: 'cluster',
    env_production: {
      NODE_ENV: 'production'
    }
  }]
};

# Start with PM2
pm2 start ecosystem.config.js --env production

# Monitor
pm2 monit

# View logs
pm2 logs strapi

# Restart on reboot
pm2 startup
pm2 save
```

### Docker Deployment

**Dockerfile:**
```dockerfile
FROM node:24-alpine

WORKDIR /app

COPY package.json yarn.lock ./
RUN yarn install --production

COPY . .

RUN yarn build

EXPOSE 1337

CMD ["yarn", "start"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  strapi:
    build: .
    ports:
      - "1337:1337"
    environment:
      NODE_ENV: production
      DATABASE_CLIENT: mysql
      DATABASE_HOST: mysql
      DATABASE_NAME: strapi
    depends_on:
      - mysql

  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: strapi
      MYSQL_USER: strapi
      MYSQL_PASSWORD: strapi
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

**Deploy:**
```bash
docker-compose up -d
```

---

**Related Documentation:**
- [Developer Starter Guide](./DEVELOPER_GUIDE.md)
- [Architecture Guide](./ARCHITECTURE.md)
- [API Reference](./API_REFERENCE.md)
