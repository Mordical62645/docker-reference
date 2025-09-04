# Docker Image Management: Build, Pull, Push, Tag

## Building Images

### Basic Build
```bash
# Build from current directory
docker build -t myapp .

# Build from specific directory
docker build -t myapp /path/to/project

# Build with specific Dockerfile
docker build -t myapp -f Dockerfile.prod .
```

### Build with Tags
```bash
# Single tag
docker build -t myapp:1.0.0 .

# Multiple tags
docker build -t myapp:1.0.0 -t myapp:latest .

# With registry
docker build -t registry.example.com/myapp:1.0.0 .
```

### Build Arguments
```bash
# Pass build arguments
docker build --build-arg NODE_VERSION=18 -t myapp .

# Multiple arguments
docker build \
  --build-arg NODE_VERSION=18 \
  --build-arg BUILD_ENV=production \
  -t myapp .
```

### Build Options
```bash
# No cache
docker build --no-cache -t myapp .

# Specific target (multi-stage)
docker build --target production -t myapp .

# Progress output
docker build --progress=plain -t myapp .

# Memory limit
docker build --memory=1g -t myapp .
```

## Pulling Images

### Basic Pull
```bash
# Pull latest tag
docker pull nginx

# Pull specific tag
docker pull nginx:alpine

# Pull specific version
docker pull postgres:15.2
```

### Registry Images
```bash
# Pull from Docker Hub
docker pull ubuntu:20.04

# Pull from private registry
docker pull registry.company.com/myapp:latest

# Pull with full path
docker pull docker.io/library/nginx:latest
```

### Pull All Tags
```bash
# Pull all available tags
docker pull --all-tags nginx
```

## Pushing Images

### Login to Registry
```bash
# Docker Hub
docker login

# Private registry
docker login registry.company.com

# With credentials
docker login -u username -p password registry.company.com
```

### Push to Registry
```bash
# Push to Docker Hub
docker push username/myapp:latest

# Push specific tag
docker push username/myapp:1.0.0

# Push to private registry
docker push registry.company.com/myapp:latest
```

### Push Multiple Tags
```bash
# Push all tags for an image
docker push --all-tags username/myapp
```

## Tagging Images

### Basic Tagging
```bash
# Tag existing image
docker tag myapp:latest myapp:1.0.0

# Tag for registry
docker tag myapp:latest username/myapp:latest

# Tag for different registry
docker tag myapp:latest registry.company.com/myapp:latest
```

### Tagging Strategies
```bash
# Semantic versioning
docker tag myapp:latest myapp:1.2.3
docker tag myapp:latest myapp:1.2
docker tag myapp:latest myapp:1

# Environment tags
docker tag myapp:latest myapp:dev
docker tag myapp:latest myapp:staging
docker tag myapp:latest myapp:prod

# Git-based tags
docker tag myapp:latest myapp:commit-abc1234
docker tag myapp:latest myapp:branch-main
```

## Complete Workflow Examples

### Development Workflow
```bash
# 1. Build development image
docker build -t myapp:dev .

# 2. Test locally
docker run myapp:dev

# 3. Tag for staging
docker tag myapp:dev myapp:staging

# 4. Push to registry
docker push username/myapp:staging
```

### Production Release
```bash
# 1. Build production image
docker build --target production -t myapp:1.2.3 .

# 2. Tag with multiple versions
docker tag myapp:1.2.3 myapp:1.2
docker tag myapp:1.2.3 myapp:1
docker tag myapp:1.2.3 myapp:latest

# 3. Tag for registry
docker tag myapp:1.2.3 username/myapp:1.2.3
docker tag myapp:1.2.3 username/myapp:latest

# 4. Push all tags
docker push --all-tags username/myapp
```

### CI/CD Pipeline
```bash
#!/bin/bash

# Get version from git
VERSION=$(git describe --tags --abbrev=0)
COMMIT=$(git rev-parse --short HEAD)

# Build image
docker build -t myapp:$VERSION .

# Tag with commit and latest
docker tag myapp:$VERSION myapp:commit-$COMMIT
docker tag myapp:$VERSION myapp:latest

# Tag for registry
docker tag myapp:$VERSION registry.company.com/myapp:$VERSION
docker tag myapp:$VERSION registry.company.com/myapp:latest

# Push to registry
docker push registry.company.com/myapp:$VERSION
docker push registry.company.com/myapp:latest
```

## Multi-Architecture Builds

### Setup Buildx
```bash
# Create and use new builder
docker buildx create --name multiarch --driver docker-container
docker buildx use multiarch

# Inspect builder
docker buildx inspect --bootstrap
```

### Build Multi-Architecture
```bash
# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t username/myapp:latest \
  --push .

# Build and load locally (single platform)
docker buildx build \
  --platform linux/amd64 \
  -t myapp:latest \
  --load .
```

## Registry Management

### List Images
```bash
# List local images
docker images

# List with specific repository
docker images myapp

# List with filters
docker images --filter="dangling=true"
docker images --filter="before=myapp:1.0.0"
```

### Remove Images
```bash
# Remove single image
docker rmi myapp:1.0.0

# Remove multiple images
docker rmi myapp:1.0.0 myapp:1.1.0

# Remove all unused images
docker image prune

# Remove all images
docker image prune -a
```

### Image Information
```bash
# Inspect image
docker inspect myapp:latest

# View image history
docker history myapp:latest

# Get image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

## Docker Hub Specific

### Repository Setup
```bash
# Create repository naming
username/repository-name:tag

# Examples
johndoe/webapp:latest
company/api-server:v1.2.3
myteam/frontend:production
```

### Automated Builds
```bash
# Link GitHub repository to Docker Hub
# 1. Connect GitHub account
# 2. Create automated build
# 3. Configure build rules:
#    - Source: main branch
#    - Docker tag: latest
#    - Dockerfile location: /
```

### Docker Hub Tags
```bash
# Common tagging patterns
docker tag myapp:latest username/myapp:latest
docker tag myapp:latest username/myapp:v1.0.0
docker tag myapp:latest username/myapp:stable
```

## Private Registry

### Setup Local Registry
```bash
# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for local registry
docker tag myapp:latest localhost:5000/myapp:latest

# Push to local registry
docker push localhost:5000/myapp:latest

# Pull from local registry
docker pull localhost:5000/myapp:latest
```

### Enterprise Registry
```bash
# Login to enterprise registry
docker login registry.company.com

# Tag with registry URL
docker tag myapp:latest registry.company.com/team/myapp:latest

# Push to enterprise registry
docker push registry.company.com/team/myapp:latest
```

## Best Practices

### Tagging Strategy
```bash
# Use semantic versioning
myapp:1.2.3    # Specific version
myapp:1.2      # Minor version
myapp:1        # Major version
myapp:latest   # Latest stable

# Environment-based tags
myapp:dev      # Development
myapp:test     # Testing
myapp:stage    # Staging
myapp:prod     # Production

# Feature-based tags
myapp:feature-auth
myapp:hotfix-security
myapp:release-candidate
```

### Image Naming
```bash
# Good naming conventions
company/service-name:version
team/application-component:tag
project/microservice-name:environment

# Examples
shop/api-gateway:v2.1.0
finance/payment-service:staging
ecommerce/frontend:latest
```

### Security
```bash
# Scan images for vulnerabilities
docker scout quickview myapp:latest

# Sign images (Docker Content Trust)
export DOCKER_CONTENT_TRUST=1
docker push username/myapp:latest

# Use minimal base images
FROM alpine:latest
FROM node:18-alpine
FROM python:3.11-slim
```

## Common Commands Summary
```bash
# Build
docker build -t name:tag .

# Tag
docker tag source:tag target:tag

# Push
docker push name:tag

# Pull
docker pull name:tag

# List
docker images

# Remove
docker rmi name:tag

# Login
docker login [registry]

# Logout
docker logout [registry]
```
