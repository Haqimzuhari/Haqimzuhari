# Docker Development Environment

A collection of Dockerfiles and docker-compose configurations for local development. Run Python, Node.js, and PHP applications without installing them on your machine.

---

## Table of Contents

- [Docker Development Environment](#docker-development-environment)
  - [Table of Contents](#table-of-contents)
  - [Dockerfiles](#dockerfiles)
    - [Node.js](#nodejs)
    - [Python](#python)
    - [PHP](#php)
  - [Databases](#databases)
    - [MySQL](#mysql)
    - [PostgreSQL](#postgresql)
  - [Nginx (Production Reverse Proxy)](#nginx-production-reverse-proxy)
    - [Nginx Configuration](#nginx-configuration)
  - [Docker Compose](#docker-compose)
    - [Application Services](#application-services)
    - [Database Services](#database-services)
    - [Networks](#networks)
  - [Connecting to Databases](#connecting-to-databases)
    - [From Application Containers](#from-application-containers)
    - [From Host Machine](#from-host-machine)
  - [Quick Start](#quick-start)
  - [Notes](#notes)

---

## Dockerfiles

### Node.js

**Filename:** `Dockerfile.node`

```dockerfile
FROM node:22-slim

# ===========================================
# With Nginx (uncomment below)
# ===========================================
# Install Nginx and Supervisor (to run multiple processes)
# RUN apt-get update && apt-get install -y \
#     nginx \
#     supervisor \
#     && rm -rf /var/lib/apt/lists/*

RUN npm install -g npm@latest

WORKDIR /app

# ===========================================
# With Nginx (uncomment below)
# ===========================================
# COPY ./nginx/node.conf /etc/nginx/sites-available/default
# COPY ./supervisor/node.conf /etc/supervisor/conf.d/node.conf
# EXPOSE 80
# CMD ["/usr/bin/supervisord", "-n"]

# ===========================================
# Without Nginx (comment below if using Nginx)
# ===========================================
USER node
```

**With Nginx - additional files needed:**

<details>
<summary>supervisor/node.conf</summary>

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

</details>

<details>
<summary>nginx/node.conf</summary>

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

</details>

### Python

**Filename:** `Dockerfile.python`

```dockerfile
FROM python:3.13-slim

# ===========================================
# With Nginx (uncomment below)
# ===========================================
# RUN apt-get update && apt-get install -y \
#     nginx \
#     supervisor \
#     && rm -rf /var/lib/apt/lists/*

# Without Nginx
RUN useradd -u 1000 -m python

# Without Nginx (use flask only)
RUN pip install flask flask-cors

# With Nginx (use gunicorn for production)
# RUN pip install flask flask-cors gunicorn

WORKDIR /app

# ===========================================
# With Nginx (uncomment below)
# ===========================================
# COPY ./nginx/python.conf /etc/nginx/sites-available/default
# COPY ./supervisor/python.conf /etc/supervisor/conf.d/python.conf
# EXPOSE 80
# CMD ["/usr/bin/supervisord", "-n"]

# ===========================================
# Without Nginx (comment below if using Nginx)
# ===========================================
USER python
```

**With Nginx - additional files needed:**

<details>
<summary>supervisor/python.conf</summary>

```ini
[supervisord]
nodaemon=true

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true

[program:gunicorn]
# Run Flask app with Gunicorn (4 workers)
command=gunicorn -w 4 -b 127.0.0.1:4000 index:app
directory=/app
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

</details>

<details>
<summary>nginx/python.conf</summary>

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

    # Serve static files directly (optional)
    # location /static/ {
    #     alias /app/static/;
    # }
}
```

</details>

### PHP

**Filename:** `Dockerfile.php`

```dockerfile
# ===========================================
# Without Nginx: use php:8.3-cli
# With Nginx: use php:8.3-fpm
# ===========================================
FROM php:8.3-cli
# FROM php:8.3-fpm

RUN apt-get update && apt-get upgrade -y

# ===========================================
# With Nginx (add nginx and supervisor)
# ===========================================
RUN apt-get install -y \
    # nginx \
    # supervisor \
    git \
    curl \
    zip \
    unzip \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions for Laravel
RUN docker-php-ext-install \
    pdo \
    pdo_mysql \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    zip \
    xml \
    curl

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install Node.js (for Laravel Mix/Vite)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g npm@latest

WORKDIR /app

COPY ./php/php.ini /usr/local/etc/php/php.ini

# ===========================================
# With Nginx (uncomment below)
# ===========================================
# COPY ./nginx/php.conf /etc/nginx/sites-available/default
# COPY ./supervisor/php.conf /etc/supervisor/conf.d/php.conf
# EXPOSE 80
# CMD ["/usr/bin/supervisord", "-n"]

# ===========================================
# Without Nginx (comment below if using Nginx)
# ===========================================
RUN useradd -u 1000 -m php
USER php
```

**With Nginx - additional files needed:**

<details>
<summary>supervisor/php.conf</summary>

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

</details>

<details>
<summary>nginx/php.conf (Laravel example)</summary>

```nginx
server {
    listen 80;
    server_name localhost;

    # Laravel public directory
    root /app/public;
    index index.php index.html;

    # Handle Laravel routes
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP-FPM configuration
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Deny access to hidden files
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

</details>

---

## Databases

### MySQL

Using the official MySQL 8.0 image. Data is persisted in `./mysql-data` directory.

**Default credentials:**
| Field | Value |
|-------|-------|
| Host (from container) | `mysql` |
| Host (from host) | `localhost` |
| Port | `3306` |
| Root Password | `secret` |

### PostgreSQL

Using the official PostgreSQL 16 image. Data is persisted in `./postgres-data` directory.

**Default credentials:**
| Field | Value |
|-------|-------|
| Host (from container) | `postgres` |
| Host (from host) | `localhost` |
| Port | `5432` |
| User | `postgres` |
| Password | `secret` |

---

## Nginx (Production Reverse Proxy)

In production, Nginx sits in front of your application to handle:
- Static file serving
- SSL/TLS termination
- Load balancing
- Request buffering

### Nginx Configuration

**Filename:** `nginx/default.conf`

```nginx
# ===========================================
# Upstream Definitions
# ===========================================
# Define backend servers. Use the Docker service name as hostname.
# Uncomment the upstream block(s) you need.

# Node.js upstream (Express, Fastify, Next.js, etc.)
upstream nodejs {
    server node:3000;
    # For load balancing multiple instances:
    # server node1:3000;
    # server node2:3000;
}

# Python upstream (Flask, FastAPI, Django, etc.)
# upstream python {
#     server python:4000;
# }

# PHP/Laravel upstream (using php-fpm)
# Note: Laravel typically uses php-fpm on port 9000
# upstream php {
#     server php:9000;
# }

# ===========================================
# Server Block
# ===========================================
server {
    listen 80;
    server_name localhost;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    # -------------------------------------------
    # Option 1: Node.js (Reverse Proxy)
    # Proxy all requests to Node.js application
    # -------------------------------------------
    location / {
        proxy_pass http://nodejs;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # -------------------------------------------
    # Option 2: Python/Flask/FastAPI (Reverse Proxy)
    # Uncomment to proxy to Python application
    # -------------------------------------------
    # location / {
    #     proxy_pass http://python;
    #     proxy_set_header Host $host;
    #     proxy_set_header X-Real-IP $remote_addr;
    #     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #     proxy_set_header X-Forwarded-Proto $scheme;
    # }

    # -------------------------------------------
    # Option 3: PHP/Laravel (FastCGI)
    # Uncomment for PHP applications
    # -------------------------------------------
    # root /app/public;
    # index index.php index.html;
    #
    # location / {
    #     try_files $uri $uri/ /index.php?$query_string;
    # }
    #
    # location ~ \.php$ {
    #     fastcgi_pass php;
    #     fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
    #     include fastcgi_params;
    # }

    # -------------------------------------------
    # Static files (Optional)
    # Serve static files directly without proxying
    # -------------------------------------------
    # location /static/ {
    #     alias /app/static/;
    #     expires 30d;
    #     add_header Cache-Control "public, immutable";
    # }
}
```

---

## Docker Compose

**Filename:** `docker-compose.yml`

### Application Services

```yaml
services:
  # ===========================================
  # NGINX (Separate Container)
  # ===========================================
  # Uncomment for: Production with Nginx as separate container
  # Access via: http://localhost (port 80)
  # nginx:
  #   image: nginx:alpine
  #   container_name: nginx-container
  #   ports:
  #     - "80:80"
  #     - "443:443"
  #   volumes:
  #     - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
  #     # For SSL certificates (optional):
  #     # - ./nginx/ssl:/etc/nginx/ssl
  #   depends_on:
  #     - node   # Change to python or php depending on your app
  #   restart: unless-stopped
  #   networks:
  #     - mysql_shared_network

  # ===========================================
  # NODE.JS
  # ===========================================
  node:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node-container
    working_dir: /app
    volumes:
      - .:/app
      # -------------------------------------------
      # With Nginx in same container (uncomment below)
      # Requires: Dockerfile.node with Nginx sections uncommented
      # -------------------------------------------
      # - ./nginx/node.conf:/etc/nginx/sites-available/default
      # - ./supervisor/node.conf:/etc/supervisor/conf.d/node.conf
    # -------------------------------------------
    # Without Nginx (development) - Access: http://localhost:3000
    # -------------------------------------------
    ports:
      - "3000:3000"
    command: sh -c "cd web && npm run dev"
    # -------------------------------------------
    # With Nginx (separate container) - Internal only
    # -------------------------------------------
    # expose:
    #   - "3000"
    # command: sh -c "cd web && npm run start"
    # -------------------------------------------
    # With Nginx (same container) - Access: http://localhost
    # -------------------------------------------
    # ports:
    #   - "80:80"
    # (no command needed, supervisor handles it)
    tty: true
    networks:
      - mysql_shared_network

  # ===========================================
  # PYTHON
  # ===========================================
  # python:
  #   build:
  #     context: .
  #     dockerfile: Dockerfile.python
  #   container_name: python-container
  #   working_dir: /app
  #   volumes:
  #     - .:/app
  #     # With Nginx in same container (uncomment below)
  #     # - ./nginx/python.conf:/etc/nginx/sites-available/default
  #     # - ./supervisor/python.conf:/etc/supervisor/conf.d/python.conf
  #   # Without Nginx (development) - Access: http://localhost:4000
  #   ports:
  #     - "4000:4000"
  #   command: sh -c "cd api && python3 index.py"
  #   # With Nginx (separate container) - Internal only
  #   # expose:
  #   #   - "4000"
  #   # command: sh -c "cd api && gunicorn -w 4 -b 0.0.0.0:4000 index:app"
  #   # With Nginx (same container) - Access: http://localhost
  #   # ports:
  #   #   - "80:80"
  #   tty: true
  #   networks:
  #     - postgres_shared_network

  # ===========================================
  # PHP
  # ===========================================
  # php:
  #   build:
  #     context: .
  #     dockerfile: Dockerfile.php
  #   container_name: php-container
  #   working_dir: /app
  #   volumes:
  #     - .:/app
  #     # With Nginx in same container (uncomment below)
  #     # - ./nginx/php.conf:/etc/nginx/sites-available/default
  #     # - ./supervisor/php.conf:/etc/supervisor/conf.d/php.conf
  #   # Without Nginx (development) - Access: http://localhost:8080
  #   ports:
  #     - "8080:8080"
  #   command: sh -c "cd php && php -S 0.0.0.0:8080"
  #   # With Nginx (separate container) - Internal only, use php-fpm
  #   # expose:
  #   #   - "9000"
  #   # With Nginx (same container) - Access: http://localhost
  #   # ports:
  #   #   - "80:80"
  #   tty: true
  #   networks:
  #     - mysql_shared_network
```

### Database Services

```yaml
  # ===========================================
  # MYSQL
  # ===========================================
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
      - mysql_shared_network

  # ===========================================
  # POSTGRESQL
  # ===========================================
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
      - postgres_shared_network
```

### Networks

```yaml
networks:
  mysql_shared_network:
    name: mysql_external_network
    driver: bridge

  postgres_shared_network:
    name: postgres_external_network
    driver: bridge
```

---

## Connecting to Databases

### From Application Containers

When connecting from within Docker containers on the same network, use the **service name** as the hostname.

**MySQL connection example (PHP):**

```php
$pdo = new PDO(
    'mysql:host=mysql;port=3306;dbname=mydb',
    'root',
    'secret'
);
```

**MySQL connection example (Node.js):**

```javascript
const mysql = require('mysql2');

const connection = mysql.createConnection({
  host: 'mysql',
  port: 3306,
  user: 'root',
  password: 'secret',
  database: 'mydb'
});
```

**PostgreSQL connection example (Python):**

```python
import psycopg2

conn = psycopg2.connect(
    host="postgres",
    port=5432,
    user="postgres",
    password="secret",
    database="mydb"
)
```

### From Host Machine

When connecting from your local machine (e.g., using a database client), use `localhost` as the hostname.

**MySQL:**

```
Host: localhost
Port: 3306
User: root
Password: secret
```

**PostgreSQL:**

```
Host: localhost
Port: 5432
User: postgres
Password: secret
```

---

## Quick Start

1. **Build and start all services:**

   ```bash
   docker-compose up -d --build
   ```

2. **Start only specific services:**

   ```bash
   docker-compose up -d mysql postgres
   docker-compose up -d node
   ```

3. **Access a running container:**

   ```bash
   docker exec -it node-container sh
   docker exec -it php-container bash
   docker exec -it python-container bash
   ```

4. **View logs:**

   ```bash
   docker-compose logs -f node
   ```

5. **Stop all services:**

   ```bash
   docker-compose down
   ```

6. **Stop and remove volumes (reset database):**

   ```bash
   docker-compose down -v
   rm -rf mysql-data postgres-data
   ```

---

## Notes

- All application containers run as non-root users for security
- Database data is persisted locally in `mysql-data` and `postgres-data` directories
- Add these directories to `.gitignore` to avoid committing database files
- Modify the `command` field in docker-compose to match your project structure
