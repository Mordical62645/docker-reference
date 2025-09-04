# docker-compose.yml Setup Guide

## Basic Structure
```yaml
version: '3.8'

services:
  service-name:
    # Service configuration

volumes:
  # Named volumes

networks:
  # Custom networks
```

## Complete Template
```yaml
version: '3.8'

services:
  backend:
    build:
      context: ..
      dockerfile: docker/backend/Dockerfile
    container_name: app-backend
    restart: unless-stopped
    ports:
      - "5000:5000"
    volumes:
      - ../backend:/app
      - backend_data:/app/data
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - TZ=Asia/Manila
    env_file:
      - ../backend/.env
    depends_on:
      - db
      - redis
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build:
      context: ..
      dockerfile: docker/frontend/Dockerfile
    container_name: app-frontend
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - ../frontend/src:/app/src
      - ../frontend/public:/app/public
    environment:
      - NODE_ENV=production
      - REACT_APP_API_URL=http://localhost:5000
    depends_on:
      - backend
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    container_name: app-db
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    container_name: app-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

  nginx:
    build:
      context: ..
      dockerfile: docker/nginx/Dockerfile
    container_name: app-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

volumes:
  db_data:
  redis_data:
  backend_data:

networks:
  app-network:
    driver: bridge
```

## Environment-Specific Configurations

### Development (docker-compose.dev.yml)
```yaml
version: '3.8'

services:
  backend:
    build:
      context: ..
      dockerfile: docker/backend/Dockerfile
      target: development
    volumes:
      - ../backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=*
    command: npm run dev

  frontend:
    build:
      context: ..
      dockerfile: docker/frontend/Dockerfile
      target: development
    volumes:
      - ../frontend/src:/app/src
      - ../frontend/public:/app/public
    environment:
      - NODE_ENV=development
      - FAST_REFRESH=true
    command: npm run start
```

### Production (docker-compose.prod.yml)
```yaml
version: '3.8'

services:
  backend:
    build:
      context: ..
      dockerfile: docker/backend/Dockerfile
      target: production
    restart: always
    environment:
      - NODE_ENV=production
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
    logging:
      driver: json-file
      options:
        max-size: 10m
        max-file: "3"

  frontend:
    build:
      context: ..
      dockerfile: docker/frontend/Dockerfile
      target: production
    restart: always
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.25'
```

## Service Configuration Options

### Build Configuration
```yaml
services:
  app:
    build:
      context: ..
      dockerfile: docker/app/Dockerfile
      target: production
      args:
        - BUILD_VERSION=1.0.0
        - NODE_ENV=production
      cache_from:
        - myapp:latest
```

### Volume Configurations
```yaml
services:
  app:
    volumes:
      # Bind mount (development)
      - ../app:/code
      
      # Named volume (production)
      - app_data:/data
      
      # Read-only mount
      - ../config:/config:ro
      
      # Exclude subdirectory
      - /code/node_modules
```

### Network Configuration
```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
  
services:
  web:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend  # Only backend network
```

### Environment Variables
```yaml
services:
  app:
    environment:
      # Direct assignment
      - NODE_ENV=production
      - PORT=3000
      
      # From host environment
      - DATABASE_URL=${DATABASE_URL}
      
    env_file:
      # Multiple env files
      - .env
      - .env.local
      - environments/${NODE_ENV}.env
```

## Database Services

### PostgreSQL
```yaml
services:
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### MySQL
```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - MYSQL_DATABASE=myapp
      - MYSQL_USER=myuser
      - MYSQL_PASSWORD=mypassword
    volumes:
      - mysql_data:/var/lib/mysql
      - ./my.cnf:/etc/mysql/conf.d/my.cnf
    ports:
      - "3306:3306"

volumes:
  mysql_data:
```

### MongoDB
```yaml
services:
  mongodb:
    image: mongo:6.0
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
      - MONGO_INITDB_DATABASE=myapp
    volumes:
      - mongodb_data:/data/db
      - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js
    ports:
      - "27017:27017"

volumes:
  mongodb_data:
```

## Common Service Patterns

### Redis Cache
```yaml
services:
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass mypassword
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"

volumes:
  redis_data:
```

### Nginx Reverse Proxy
```yaml
services:
  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - backend
      - frontend
```

### Background Worker
```yaml
services:
  worker:
    build:
      context: ..
      dockerfile: docker/backend/Dockerfile
    command: python worker.py
    environment:
      - WORKER_QUEUES=default,emails
    depends_on:
      - redis
      - db
    deploy:
      replicas: 3
```

## Override Files

### docker-compose.override.yml (auto-loaded)
```yaml
version: '3.8'

services:
  backend:
    volumes:
      - ../backend:/app
    environment:
      - DEBUG=true
    command: npm run dev
```

### Usage with override files
```bash
# Default + override
docker-compose up

# Specific files
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

# Multiple overrides
docker-compose -f docker-compose.yml -f docker-compose.dev.yml -f docker-compose.local.yml up
```

## Security Configurations

### User and Security
```yaml
services:
  app:
    user: "1000:1000"  # Non-root user
    read_only: true     # Read-only container
    tmpfs:
      - /tmp
      - /var/run
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
```

### Secrets Management
```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true

services:
  app:
    secrets:
      - db_password
      - api_key
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
```

## Resource Limits
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
```

## Health Checks
```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Logging Configuration
```yaml
services:
  app:
    logging:
      driver: json-file
      options:
        max-size: 10m
        max-file: "3"
        labels: "production_status"
        env: "os,customer"
```

## Common Commands
```bash
# Build and start
docker-compose up --build

# Start in background
docker-compose up -d

# Stop and remove
docker-compose down

# Stop and remove with volumes
docker-compose down -v

# View logs
docker-compose logs -f service-name

# Scale services
docker-compose up --scale worker=3

# Run one-off command
docker-compose run app python manage.py migrate
```
