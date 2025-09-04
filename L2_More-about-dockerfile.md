# Dockerfile Setup and Templates

## Docker Directory Structure

Organize Docker files in your project:

```
project/
├── docker/
│   ├── backend/
│   │   └── Dockerfile
│   ├── frontend/
│   │   └── Dockerfile
│   ├── nginx/
│   │   ├── Dockerfile
│   │   └── nginx.conf
│   └── docker-compose.yml
├── backend/
│   ├── requirements.txt
│   └── app.py
├── frontend/
│   ├── package.json
│   └── src/
└── .dockerignore
```

## Dockerfile Templates

### Python/Flask/Django
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

### Node.js/React/Vue
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Build if needed (for React/Vue)
# RUN npm run build

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

EXPOSE 3000

CMD ["npm", "start"]
```

### PHP/Laravel
```dockerfile
FROM php:8.2-fpm-alpine

WORKDIR /var/www/html

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Copy composer files
COPY composer.json composer.lock ./

# Install PHP dependencies
RUN composer install --no-dev --optimize-autoloader

# Copy application code
COPY . .

# Set permissions
RUN chown -R www-data:www-data /var/www/html

EXPOSE 9000

CMD ["php-fpm"]
```

### Go
```dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN go build -o main .

# Final stage
FROM alpine:latest

WORKDIR /app

# Install ca-certificates for HTTPS
RUN apk --no-cache add ca-certificates

# Copy binary from builder
COPY --from=builder /app/main .

EXPOSE 8080

CMD ["./main"]
```

### Java/Spring Boot
```dockerfile
FROM openjdk:17-jdk-slim AS builder

WORKDIRL /app

# Copy Maven/Gradle files
COPY pom.xml ./
COPY mvnw ./
COPY .mvn .mvn

# Download dependencies
RUN ./mvnw dependency:go-offline

# Copy source code
COPY src ./src

# Build application
RUN ./mvnw package -DskipTests

# Final stage
FROM openjdk:17-jre-slim

WORKDIR /app

# Copy jar from builder
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

### Nginx (Reverse Proxy/Static)
```dockerfile
FROM nginx:alpine

# Remove default config
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom config
COPY nginx.conf /etc/nginx/conf.d/

# Copy static files (if serving static content)
# COPY dist/ /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Multi-stage Build Example
```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Best Practices

### Layer Optimization
```dockerfile
# Bad - creates multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# Good - single layer
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*
```

### Cache Optimization
```dockerfile
# Copy dependency files first
COPY package*.json ./
RUN npm install

# Copy source code after
COPY . .
```

### Security
```dockerfile
# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
USER appuser

# Use specific image tags
FROM node:18.17.0-alpine

# Remove package managers
RUN rm -rf /var/lib/apt/lists/*
```

## Environment Variables
```dockerfile
# Set default environment
ENV NODE_ENV=production
ENV PORT=3000

# Use ARG for build-time variables
ARG BUILD_VERSION
ENV VERSION=$BUILD_VERSION
```

## Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1
```
