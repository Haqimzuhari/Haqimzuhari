# 🐳 PHP Development Setup with Docker

A production-ready PHP 8.2 environment with separate dev and prod configurations.

---

## 📋 Prerequisites

- 🐋 **Docker** (version 20.10 or higher)
- 🔧 **Docker Compose** (version 2.0 or higher)

---

## 🏗️ Project Structure

```
php-project/
├── 📄 Dockerfile
├── 📄 Dockerfile.dev
├── 📄 docker-compose.yml
├── 📄 docker-compose.dev.yml
├── 📄 nginx.conf
├── 📄 nginx.dev.conf
├── 📄 php.ini
├── 📄 php.dev.ini
├── 📁 public/
│   └── index.php
└── 📁 src/
```

---

## 📝 Configuration Files

### 1️⃣ Dockerfile.dev (Development)

Create a `Dockerfile.dev` in your project root:

```dockerfile
FROM php:8.2-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl zip unzip libzip-dev libpng-dev \
    libonig-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql zip mbstring xml bcmath gd fileinfo opcache

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer

# Create non-root user
RUN groupadd -g 1000 www && \
    useradd -u 1000 -ms /bin/bash -g www www

# Set working directory
WORKDIR /var/www

# Switch to non-root user
USER www

# Install NVM as www user
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash && \
    bash -c "source ~/.nvm/nvm.sh && nvm install 24"

EXPOSE 9000
```

### 2️⃣ Dockerfile (Production)

Create a `Dockerfile` in your project root:

```dockerfile
FROM php:8.2-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl zip unzip libzip-dev libpng-dev \
    libonig-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql zip mbstring xml bcmath gd fileinfo opcache

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer

# Create non-root user
RUN groupadd -g 1000 www && \
    useradd -u 1000 -ms /bin/bash -g www www

# Set working directory
WORKDIR /var/www

# Copy application files
COPY --chown=www:www . /var/www

# Install dependencies (production only)
RUN composer install --no-dev --optimize-autoloader --no-interaction

# Install Node.js for building assets
RUN curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Build frontend assets (if needed)
RUN if [ -f "package.json" ]; then npm ci && npm run build && rm -rf node_modules; fi

# Switch to non-root user
USER www

EXPOSE 9000
```

### 3️⃣ docker-compose.dev.yml (Development)

Create a `docker-compose.dev.yml` in your project root:

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: nginx_dev
    ports:
      - "8080:80"
    volumes:
      - ./nginx.dev.conf:/etc/nginx/conf.d/default.conf:ro
      - ./public:/var/www/public:ro
      - ./src:/var/www/src:ro
    depends_on:
      - php
    restart: unless-stopped

  php:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: php_dev
    volumes:
      - ./:/var/www
      - ./php.dev.ini:/usr/local/etc/php/php.ini:ro
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
    restart: unless-stopped
```

### 4️⃣ docker-compose.yml (Production)

Create a `docker-compose.yml` in your project root:

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: nginx_prod
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - php
    restart: always

  php:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: php_prod
    volumes:
      - ./php.ini:/usr/local/etc/php/php.ini:ro
    environment:
      - APP_ENV=production
      - APP_DEBUG=false
      - DB_HOST=${DB_HOST}
      - DB_DATABASE=${DB_DATABASE}
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
    restart: always
```

### 5️⃣ nginx.dev.conf (Development)

Create an `nginx.dev.conf` in your project root:

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/public;
    index index.php index.html;

    # Logs for debugging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Main location
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP-FPM configuration
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    # Deny access to hidden files
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### 6️⃣ nginx.conf (Production)

Create an `nginx.conf` in your project root:

```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    root /var/www/public;
    index index.php;

    # SSL certificates
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # CORS headers (if serving API for React)
    add_header 'Access-Control-Allow-Origin' 'https://yourdomain.com' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization' always;

    # Logs
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # Handle preflight requests
    if ($request_method = 'OPTIONS') {
        return 204;
    }

    # Main location
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP-FPM configuration
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    # Deny access to hidden files
    location ~ /\.(?!well-known).* {
        deny all;
    }

    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 7️⃣ php.dev.ini (Development)

Create a `php.dev.ini` in your project root:

```ini
[PHP]
date.timezone = Asia/Kuala_Lumpur

display_errors = On
display_startup_errors = On
error_reporting = E_ALL

max_execution_time = 300
max_input_time = 300
memory_limit = 512M

upload_max_filesize = 64M
post_max_size = 64M

opcache.enable = 1
opcache.validate_timestamps = 1
opcache.revalidate_freq = 0
opcache.memory_consumption = 256
opcache.max_accelerated_files = 10000
```

### 8️⃣ php.ini (Production)

Create a `php.ini` in your project root:

```ini
[PHP]
date.timezone = Asia/Kuala_Lumpur

display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/php_errors.log

max_execution_time = 60
max_input_time = 60
memory_limit = 256M

upload_max_filesize = 10M
post_max_size = 10M

expose_php = Off

opcache.enable = 1
opcache.validate_timestamps = 0
opcache.revalidate_freq = 0
opcache.memory_consumption = 256
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 20000
```

### 9️⃣ public/index.php

Create a test file `public/index.php`:

```php
<?php

phpinfo();
```

---

## 🚀 Getting Started

### Development

```bash
# Build and start development server
docker-compose -f docker-compose.dev.yml build
docker-compose -f docker-compose.dev.yml up -d

# Visit http://localhost:8080
```

### Production

```bash
# Build production image
docker-compose build

# Start production server
docker-compose up -d

# Visit http://localhost (or your domain)
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

# Access PHP container
docker-compose -f docker-compose.dev.yml exec php bash

# Install composer package
docker-compose -f docker-compose.dev.yml exec php composer require vendor/package

# Run Laravel commands
docker-compose -f docker-compose.dev.yml exec php php artisan migrate
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

# Access container
docker-compose exec php bash
```

---

## 📊 Key Differences

| Aspect | Development | Production |
|--------|-------------|------------|
| **Dockerfile** | `Dockerfile.dev` | `Dockerfile` |
| **Compose File** | `docker-compose.dev.yml` | `docker-compose.yml` |
| **Nginx Config** | `nginx.dev.conf` | `nginx.conf` |
| **PHP Config** | `php.dev.ini` | `php.ini` |
| **Volumes** | Mounted from host | Only php.ini mounted |
| **Files** | Live editing | Baked into image |
| **Dependencies** | All (dev + prod) | Production only |
| **Node.js** | NVM in container | Build-time only |
| **HTTPS** | ❌ HTTP only | ✅ HTTPS enabled |
| **Debug** | Enabled | Disabled |
| **Port** | 8080 | 80/443 |

---

## 📄 License

Feel free to modify and use this setup for your projects!

---

**Happy Coding! 🚀**