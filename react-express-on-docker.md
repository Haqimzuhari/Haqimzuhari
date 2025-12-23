# 🐳 Node.js Development Setup with Docker

A production-ready development environment for React and Express.js with Nginx for serving built assets.

---

## 📋 Prerequisites

- 🐋 **Docker** (version 20.10 or higher)
- 🔧 **Docker Compose** (version 2.0 or higher)

---

## 🏗️ Project Structure

### React Project
```
react-project/
├── 📄 Dockerfile
├── 📄 Dockerfile.dev
├── 📄 docker-compose.yml
├── 📄 docker-compose.dev.yml
├── 📄 nginx.conf
├── 📁 public/
├── 📁 src/
├── 📄 package.json
└── 📄 .env
```

### Express Project
```
express-project/
├── 📄 Dockerfile
├── 📄 Dockerfile.dev
├── 📄 docker-compose.yml
├── 📄 docker-compose.dev.yml
├── 📁 src/
│   └── index.js
├── 📄 package.json
└── 📄 .env
```

---

## 📝 Configuration Files

## For React Projects

### 1️⃣ Dockerfile (Production)

```dockerfile
# Build stage
FROM node:24-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build the app
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built assets from builder
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 2️⃣ Dockerfile.dev (Development)

```dockerfile
FROM node:24-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### 3️⃣ docker-compose.yml (Production)

```yaml
services:
  react:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: react_prod
    ports:
      - "80:80"
    restart: always
```

### 4️⃣ docker-compose.dev.yml (Development)

```yaml
services:
  react:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: react_dev
    ports:
      - "3000:3000"
    volumes:
      - ./src:/app/src
      - ./public:/app/public
    environment:
      - REACT_APP_API_URL=http://localhost:8080/api
      - CHOKIDAR_USEPOLLING=true
    stdin_open: true
    tty: true
```

### 5️⃣ nginx.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # React Router support
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

---

## For Express.js Projects

### 1️⃣ Dockerfile (Production)

```dockerfile
FROM node:24-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production

# Copy source code
COPY . .

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -u 1000 -G appuser -s /bin/sh -D appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 3000

CMD ["node", "src/index.js"]
```

### 2️⃣ Dockerfile.dev (Development)

```dockerfile
FROM node:24-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev)
RUN npm install

# Copy source code
COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

### 3️⃣ docker-compose.yml (Production)

```yaml
services:
  express:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: express_prod
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - DB_HOST=${DB_HOST}
      - DB_DATABASE=${DB_DATABASE}
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
    restart: always
```

### 4️⃣ docker-compose.dev.yml (Development)

```yaml
services:
  express:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: express_dev
    ports:
      - "3000:3000"
    volumes:
      - ./src:/app/src
      - ./package.json:/app/package.json
    environment:
      - NODE_ENV=development
      - PORT=3000
    restart: unless-stopped
```

### 5️⃣ Example Express App (src/index.js)

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Express!' });
});

app.get('/api/health', (req, res) => {
  res.json({ status: 'OK', env: process.env.NODE_ENV });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

### 6️⃣ Example package.json for Express

```json
{
  "name": "express-app",
  "version": "1.0.0",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

---

## 🚀 Getting Started

### React Projects

**Development:**
```bash
# Build and start
docker-compose -f docker-compose.dev.yml build
docker-compose -f docker-compose.dev.yml up -d

# Visit http://localhost:3000
```

**Production:**
```bash
docker-compose build
docker-compose up -d

# Visit http://localhost
```

### Express.js Projects

**Development:**
```bash
# Build and start
docker-compose -f docker-compose.dev.yml build
docker-compose -f docker-compose.dev.yml up -d

# Visit http://localhost:3000
# API endpoint: http://localhost:3000/api/health
```

**Production:**
```bash
docker-compose build
docker-compose up -d

# Visit http://localhost:3000
```

---

## 💻 Common Commands

### Development

```bash
# Start
docker-compose -f docker-compose.dev.yml up -d

# Stop
docker-compose -f docker-compose.dev.yml down

# View logs
docker-compose -f docker-compose.dev.yml logs -f

# Install package (React)
docker-compose -f docker-compose.dev.yml exec react npm install axios

# Install package (Express)
docker-compose -f docker-compose.dev.yml exec express npm install mongoose

# Access container
docker-compose -f docker-compose.dev.yml exec react sh
docker-compose -f docker-compose.dev.yml exec express sh
```

### Production

```bash
# Build
docker-compose build

# Start
docker-compose up -d

# Stop
docker-compose down

# View logs
docker-compose logs -f
```

---

## 🔗 Connecting React to Express Backend

### React .env (Development)
```env
REACT_APP_API_URL=http://localhost:3000/api
```

### React .env (Production)
```env
REACT_APP_API_URL=https://api.yourdomain.com
```

### Express CORS Setup

```javascript
const express = require('express');
const cors = require('cors');
const app = express();

// CORS configuration
app.use(cors({
  origin: process.env.NODE_ENV === 'production' 
    ? 'https://yourdomain.com' 
    : 'http://localhost:3000'
}));

app.use(express.json());

// Your routes here
```

---

## 📊 Quick Comparison

| Feature | React | Express |
|---------|-------|---------|
| **Type** | Frontend | Backend API |
| **Dev Port** | 3000 | 3000 |
| **Prod Server** | Nginx | Node.js |
| **Hot Reload** | ✅ Yes | ✅ Yes (nodemon) |
| **Build Step** | ✅ Yes | ❌ No |
| **Static Files** | ✅ Yes | ❌ No |

---

## 📄 License

Feel free to modify and use this setup for your projects!

---

**Happy Coding! 🚀**