# Docker Development Environment

A collection of Dockerfiles and docker-compose configurations for local development and production.

---

## Table of Contents

- [Node.js](#nodejs)
- [Python](#python)
- [PHP](#php)
- [Databases](#databases) (SQLite, MySQL, PostgreSQL)
- [Quick Start](#quick-start)
- [Notes](#notes)

---

## Node.js

### Without Nginx

**Dockerfile.node**

```dockerfile
FROM node:22-slim

RUN npm install -g npm@latest

WORKDIR /app

USER node
```

**docker-compose.yml**

```yaml
services:
  node:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node-container
    working_dir: /app
    volumes:
      - .:/app
    ports:
      - "3000:3000"
    tty: true
    command: sh -c "cd web && npm run dev"
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

Access: `http://localhost:3000`

---

### With Nginx

**Dockerfile.node**

```dockerfile
FROM node:22-slim

RUN apt-get update && apt-get install -y \
    nginx \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g npm@latest

WORKDIR /app

EXPOSE 80

CMD ["/usr/bin/supervisord", "-n"]
```

**docker-compose.yml**

```yaml
services:
  node:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node-container
    working_dir: /app
    volumes:
      - .:/app
      - ./nginx/node.conf:/etc/nginx/sites-available/default
      - ./supervisor/node.conf:/etc/supervisor/conf.d/node.conf
    ports:
      - "80:80"
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

**nginx/node.conf**

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**supervisor/node.conf**

```ini
[supervisord]
nodaemon=true

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true

[program:node]
command=npm run start
directory=/app
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

Access: `http://localhost`

---

## Python

### Without Nginx

**Dockerfile.python**

```dockerfile
FROM python:3.13-slim

RUN useradd -u 1000 -m python

RUN pip install flask flask-cors

WORKDIR /app

USER python
```

**docker-compose.yml**

```yaml
services:
  python:
    build:
      context: .
      dockerfile: Dockerfile.python
    container_name: python-container
    working_dir: /app
    volumes:
      - .:/app
    ports:
      - "4000:4000"
    tty: true
    command: sh -c "cd api && python3 index.py"
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

Access: `http://localhost:4000`

---

### With Nginx

**Dockerfile.python**

```dockerfile
FROM python:3.13-slim

RUN apt-get update && apt-get install -y \
    nginx \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

RUN pip install flask flask-cors gunicorn

WORKDIR /app

EXPOSE 80

CMD ["/usr/bin/supervisord", "-n"]
```

**docker-compose.yml**

```yaml
services:
  python:
    build:
      context: .
      dockerfile: Dockerfile.python
    container_name: python-container
    working_dir: /app
    volumes:
      - .:/app
      - ./nginx/python.conf:/etc/nginx/sites-available/default
      - ./supervisor/python.conf:/etc/supervisor/conf.d/python.conf
    ports:
      - "80:80"
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

**nginx/python.conf**

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**supervisor/python.conf**

```ini
[supervisord]
nodaemon=true

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true

[program:gunicorn]
command=gunicorn -w 4 -b 127.0.0.1:4000 index:app
directory=/app
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

Access: `http://localhost`

---

## PHP / Laravel

> This setup is optimized for Laravel but works for vanilla PHP too.
> For vanilla PHP, just change `root /app/public` to your document root.

### Without Nginx

**Dockerfile.php**

```dockerfile
FROM php:8.3-cli

RUN apt-get update && apt-get install -y \
    git curl zip unzip \
    libpng-dev libonig-dev libxml2-dev libzip-dev \
    libcurl4-openssl-dev libssl-dev libsqlite3-dev \
    && rm -rf /var/lib/apt/lists/*

# PHP extensions for Laravel
RUN docker-php-ext-install \
    pdo pdo_mysql pdo_sqlite \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    zip \
    xml \
    curl

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install Node.js (for Laravel Vite/Mix)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g npm@latest

RUN useradd -u 1000 -m php

WORKDIR /app

USER php
```

**docker-compose.yml**

```yaml
services:
  php:
    build:
      context: .
      dockerfile: Dockerfile.php
    container_name: php-container
    working_dir: /app
    volumes:
      - .:/app
    ports:
      - "8080:8080"
    tty: true
    # Laravel: php artisan serve
    command: sh -c "php artisan serve --host=0.0.0.0 --port=8080"
    # Vanilla PHP: php built-in server
    # command: sh -c "php -S 0.0.0.0:8080 -t public"
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

Access: `http://localhost:8080`

---

### With Nginx

**Dockerfile.php**

```dockerfile
FROM php:8.3-fpm

RUN apt-get update && apt-get install -y \
    nginx supervisor \
    git curl zip unzip \
    libpng-dev libonig-dev libxml2-dev libzip-dev \
    libcurl4-openssl-dev libssl-dev libsqlite3-dev \
    && rm -rf /var/lib/apt/lists/*

# PHP extensions for Laravel
RUN docker-php-ext-install \
    pdo pdo_mysql pdo_sqlite \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    zip \
    xml \
    curl

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install Node.js (for Laravel Vite/Mix)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g npm@latest

WORKDIR /app

EXPOSE 80

CMD ["/usr/bin/supervisord", "-n"]
```

**docker-compose.yml**

```yaml
services:
  php:
    build:
      context: .
      dockerfile: Dockerfile.php
    container_name: php-container
    working_dir: /app
    volumes:
      - .:/app
      - ./nginx/php.conf:/etc/nginx/sites-available/default
      - ./supervisor/php.conf:/etc/supervisor/conf.d/php.conf
    ports:
      - "80:80"
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

**nginx/php.conf**

```nginx
server {
    listen 80;
    server_name localhost;

    # Laravel: /app/public
    # Vanilla PHP: change to your document root (e.g., /app or /app/public)
    root /app/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

**supervisor/php.conf**

```ini
[supervisord]
nodaemon=true

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true

[program:php-fpm]
command=/usr/local/sbin/php-fpm -F
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

Access: `http://localhost`

---

## Databases

### SQLite

> SQLite is a file-based database - no Docker container needed.
> The database is just a file in your project directory.

**Setup:**

```bash
# Create database file
touch database/database.sqlite
```

**Connection examples:**

```php
// PHP / Laravel (.env)
DB_CONNECTION=sqlite
DB_DATABASE=/app/database/database.sqlite
```

```javascript
// Node.js (better-sqlite3)
const Database = require('better-sqlite3');
const db = new Database('./database/database.sqlite');
```

```python
# Python
import sqlite3
conn = sqlite3.connect('./database/database.sqlite')
```

> SQLite support (`pdo_sqlite`) is already included in the PHP Dockerfiles above.

| Pros | Cons |
|------|------|
| No container needed | Not for high concurrency |
| Zero configuration | Single file = harder to scale |
| Great for development | No network access (local only) |

---

### MySQL

**docker-compose.yml**

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql-container
    environment:
      MYSQL_ROOT_PASSWORD: secret
    ports:
      - "3306:3306"
    volumes:
      - ./mysql-data:/var/lib/mysql
    restart: unless-stopped
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

| Field | From Container | From Host |
|-------|----------------|-----------|
| Host | `mysql` | `localhost` |
| Port | `3306` | `3306` |
| User | `root` | `root` |
| Password | `secret` | `secret` |

---

### PostgreSQL

**docker-compose.yml**

```yaml
services:
  postgres:
    image: postgres:16
    container_name: postgres-container
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

| Field | From Container | From Host |
|-------|----------------|-----------|
| Host | `postgres` | `localhost` |
| Port | `5432` | `5432` |
| User | `postgres` | `postgres` |
| Password | `secret` | `secret` |

---

## Quick Start

```bash
# docker compose (v2)
# Build and start
docker compose up -d --build

# View logs
docker compose logs -f

# List services
docker compose ps

# Access container (use service name, not container name)
docker compose exec node sh
docker compose exec php bash
docker compose exec python bash

# Run one-off command
docker compose exec php composer install
docker compose exec node npm install

# Stop
docker compose down

# Stop and reset database
docker compose down -v
rm -rf mysql-data postgres-data
```

---

## Notes

- Database data is persisted in `mysql-data` and `postgres-data` directories
- Add these directories to `.gitignore`
- Modify `command` field to match your project structure
- For production, consider using separate Nginx container for better scalability
