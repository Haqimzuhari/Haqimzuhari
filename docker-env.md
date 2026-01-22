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

RUN npm install -g npm@latest

WORKDIR /app

USER node
```

### Python

**Filename:** `Dockerfile.python`

```dockerfile
FROM python:3.13-slim

RUN useradd -u 1000 -m python

RUN pip install flask flask-cors

WORKDIR /app

USER python
```

### PHP

**Filename:** `Dockerfile.php`

```dockerfile
FROM php:8.3-cli

# Update and upgrade
RUN apt-get update && apt-get upgrade -y

# Install necessary packages
RUN apt-get install -y \
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

# Install Composer (official method)
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install Node.js (official method via NodeSource)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Upgrade npm to latest
RUN npm install -g npm@latest

# Create user
RUN useradd -u 1000 -m php

WORKDIR /app

COPY ./php/php.ini /usr/local/etc/php/php.ini

USER php
```

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

## Docker Compose

**Filename:** `docker-compose.yml`

### Application Services

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
      - mysql_shared_network

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
      - postgres_shared_network

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
    command: sh -c "cd php && php -S 0.0.0.0:8080"
    networks:
      - mysql_shared_network
```

### Database Services

```yaml
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
