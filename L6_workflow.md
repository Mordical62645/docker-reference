# Docker Complete Workflow

## Development to Production Pipeline

### Phase 1: Development Setup
```bash
# 1. Initialize project structure
mkdir myapp && cd myapp
mkdir docker/{backend,frontend,nginx}
touch docker/docker-compose.yml
touch docker/backend/Dockerfile
touch .dockerignore

# 2. Create development environment
docker-compose -f docker-compose.dev.yml up --build

# 3. Enable hot reload
# Mount source code as volumes for live editing

# 4. Test locally
docker-compose exec backend npm test
docker-compose exec frontend npm run test
```

### Phase 2: Version Control Integration
```bash
# 1. Commit Docker configurations
git add Dockerfile docker-compose.yml .dockerignore
git commit -m "Add Docker configuration"

# 2. Create feature branch workflow
git checkout -b feature/docker-setup
# Make changes
git add . && git commit -m "Update Docker configs"
git push origin feature/docker-setup
```

### Phase 3: Build and Test
```bash
# 1. Build production images
docker build --target production -t myapp:$(git rev-parse --short HEAD) .

# 2. Run integration tests
docker-compose -f docker-compose.test.yml up --abort-on-container-exit

# 3. Security scanning
docker scout quickview myapp:latest
```

### Phase 4: Registry Management
```bash
# 1. Tag for registry
VERSION=$(git describe --tags --abbrev=0)
docker tag myapp:latest username/myapp:$VERSION
docker tag myapp:latest username/myapp:latest

# 2. Push to registry
docker push username/myapp:$VERSION
docker push username/myapp:latest

# 3. Clean up local images
docker image prune -f
```

### Phase 5: Deployment
```bash
# 1. Pull on production server
docker pull username/myapp:$VERSION

# 2. Deploy with zero downtime
docker-compose -f docker-compose.prod.yml up -d

# 3. Verify deployment
docker-compose ps
curl -f http://localhost/health
```

## Daily Development Workflow

### Morning Routine
```bash
# 1. Pull latest changes
git pull origin main

# 2. Rebuild if Dockerfile changed
docker-compose build --pull

# 3. Start development environment
docker-compose up -d

# 4. View logs
docker-compose logs -f backend frontend
```

### Feature Development
```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make code changes
# Edit source files

# 3. Test in container
docker-compose exec backend npm run test:watch
docker-compose exec frontend npm run dev

# 4. Commit and push
git add . && git commit -m "Add new feature"
git push origin feature/new-feature
```

### End of Day
```bash
# 1. Stop containers
docker-compose down

# 2. Clean up unused resources
docker system prune -f

# 3. Push work
git push origin feature/new-feature
```

## CI/CD Pipeline Workflow

### GitHub Actions Example
```yaml
name: Docker Build and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
        
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
          
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: |
            username/myapp:latest
            username/myapp:${{ github.sha }}
```

### GitLab CI Example
```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  script:
    - docker run --rm $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA npm test

deploy:
  stage: deploy
  script:
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker-compose -f docker-compose.prod.yml up -d
  only:
    - main
```

## Environment Management

### Development Environment
```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  backend:
    build:
      target: development
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=*
    ports:
      - "5000:5000"
```

### Staging Environment
```yaml
# docker-compose.staging.yml
version: '3.8'
services:
  backend:
    image: username/myapp:staging
    environment:
      - NODE_ENV=staging
      - DATABASE_URL=${STAGING_DATABASE_URL}
    ports:
      - "5000:5000"
```

### Production Environment
```yaml
# docker-compose.prod.yml
version: '3.8'
services:
  backend:
    image: username/myapp:latest
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    ports:
      - "5000:5000"
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
```

## Release Management

### Semantic Versioning
```bash
# 1. Create release branch
git checkout -b release/v1.2.0

# 2. Update version
npm version 1.2.0
# or manually update package.json

# 3. Build release
docker build -t myapp:1.2.0 .

# 4. Tag multiple versions
docker tag myapp:1.2.0 myapp:1.2
docker tag myapp:1.2.0 myapp:1
docker tag myapp:1.2.0 myapp:latest

# 5. Push all tags
docker push --all-tags username/myapp

# 6. Create git tag
git tag v1.2.0
git push origin v1.2.0

# 7. Merge to main
git checkout main
git merge release/v1.2.0
git push origin main
```

### Hotfix Workflow
```bash
# 1. Create hotfix branch from main
git checkout main
git checkout -b hotfix/security-fix

# 2. Make minimal changes
# Fix the issue

# 3. Build and test
docker build -t myapp:hotfix .
docker run --rm myapp:hotfix npm test

# 4. Version bump (patch)
npm version patch  # 1.2.0 -> 1.2.1

# 5. Deploy hotfix
docker tag myapp:hotfix username/myapp:1.2.1
docker push username/myapp:1.2.1

# 6. Merge back
git checkout main
git merge hotfix/security-fix
git checkout develop
git merge hotfix/security-fix
```

## Monitoring and Maintenance

### Health Checks
```bash
# Container health
docker ps --filter="health=unhealthy"

# Service health
curl -f http://localhost:5000/health

# Resource usage
docker stats --no-stream
```

### Log Management
```bash
# View logs
docker-compose logs -f --tail=100 backend

# Log rotation
docker-compose exec backend logrotate /etc/logrotate.conf

# Centralized logging
docker-compose logs | grep ERROR
```

### Backup and Recovery
```bash
# Database backup
docker-compose exec db pg_dump -U user dbname > backup.sql

# Volume backup
docker run --rm -v myapp_data:/data -v $(pwd):/backup alpine tar czf /backup/data.tar.gz /data

# Restore
docker run --rm -v myapp_data:/data -v $(pwd):/backup alpine tar xzf /backup/data.tar.gz -C /
```

## Troubleshooting Workflow

### Container Issues
```bash
# 1. Check container status
docker ps -a

# 2. Inspect container
docker inspect container_name

# 3. Check logs
docker logs --tail=50 container_name

# 4. Debug inside container
docker exec -it container_name /bin/bash

# 5. Check resource usage
docker stats container_name
```

### Network Issues
```bash
# 1. List networks
docker network ls

# 2. Inspect network
docker network inspect network_name

# 3. Test connectivity
docker exec container1 ping container2

# 4. Check port mapping
docker port container_name
```

### Volume Issues
```bash
# 1. List volumes
docker volume ls

# 2. Inspect volume
docker volume inspect volume_name

# 3. Check mount points
docker exec container mount | grep volume_name

# 4. Permissions check
docker exec container ls -la /path/to/volume
```

## Performance Optimization

### Image Optimization
```bash
# 1. Use multi-stage builds
# See L2_More-about-dockerfile.md

# 2. Minimize layers
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*

# 3. Use .dockerignore
# See L3_dockerignore.md

# 4. Choose minimal base images
FROM alpine:latest
FROM node:18-alpine
FROM python:3.11-slim
```

### Runtime Optimization
```bash
# 1. Resource limits
docker run --memory=512m --cpus=0.5 myapp

# 2. Health checks
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost/health

# 3. Restart policies
docker run --restart=unless-stopped myapp

# 4. Log limits
docker run --log-opt max-size=10m --log-opt max-file=3 myapp
```

## Security Workflow

### Image Security
```bash
# 1. Scan for vulnerabilities
docker scout quickview myapp:latest

# 2. Use official base images
FROM node:18-alpine
FROM python:3.11-slim

# 3. Run as non-root user
RUN adduser -D appuser
USER appuser

# 4. Keep images updated
docker pull node:18-alpine
docker build --pull -t myapp .
```

### Runtime Security
```bash
# 1. Read-only containers
docker run --read-only --tmpfs /tmp myapp

# 2. Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# 3. Use secrets
docker secret create db_password /path/to/password
docker service create --secret db_password myapp

# 4. Network segmentation
docker network create --internal backend-net
```

## Disaster Recovery

### Backup Strategy
```bash
# 1. Automated backups
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
docker-compose exec -T db pg_dump -U user dbname > backup_$DATE.sql
aws s3 cp backup_$DATE.sql s3://backups/

# 2. Image backup
docker save myapp:latest | gzip > myapp_latest.tar.gz

# 3. Volume backup
docker run --rm -v myapp_data:/source -v $(pwd):/dest alpine tar czf /dest/data_$DATE.tar.gz /source
```

### Recovery Process
```bash
# 1. Restore database
cat backup.sql | docker-compose exec -T db psql -U user dbname

# 2. Restore images
docker load < myapp_latest.tar.gz

# 3. Restore volumes
docker run --rm -v myapp_data:/dest -v $(pwd):/source alpine tar xzf /source/data.tar.gz -C /dest --strip-components=1

# 4. Restart services
docker-compose up -d
```

## Team Collaboration

### Code Reviews
```bash
# 1. Review Dockerfile changes
git diff HEAD~1 Dockerfile
git diff HEAD~1 docker-compose.yml

# 2. Test proposed changes
git checkout feature-branch
docker-compose build
docker-compose up -d
# Test functionality

# 3. Security review
docker scout quickview myapp:latest
```

### Documentation
```bash
# 1. Update README.md
## Docker Setup
docker-compose up -d

## Development
docker-compose -f docker-compose.dev.yml up

# 2. Environment variables
cp .env.example .env
# Edit .env with your values

# 3. Troubleshooting section
### Common Issues
- Port already in use: `docker-compose down`
- Permission denied: Check file ownership
```

## Deployment Strategies

### Blue-Green Deployment
```bash
# 1. Deploy to staging (green)
docker-compose -f docker-compose.green.yml up -d

# 2. Test green environment
curl -f http://green.example.com/health

# 3. Switch traffic
# Update load balancer configuration

# 4. Keep blue as backup
# In case rollback is needed
```

### Rolling Updates
```bash
# 1. Update one instance at a time
docker service update --image myapp:v2 myapp_backend

# 2. Monitor health
docker service ps myapp_backend

# 3. Continue if healthy
# Docker Swarm handles the rolling update
```

### Canary Deployment
```bash
# 1. Deploy canary version
docker service create --name myapp_canary --replicas 1 myapp:v2

# 2. Route small percentage of traffic
# Configure load balancer for 5% traffic to canary

# 3. Monitor metrics
# Check error rates, response times

# 4. Scale up if successful
docker service scale myapp_canary=3
docker service scale myapp_main=2
```

## Final Checklist

### Before Production Deploy
- [ ] All tests passing
- [ ] Security scan completed
- [ ] Resource limits configured
- [ ] Monitoring setup
- [ ] Backup strategy in place
- [ ] Rollback plan documented
- [ ] Health checks working
- [ ] Load testing completed
- [ ] Documentation updated
- [ ] Team notified

---

*This workflow covers the complete Docker lifecycle from development to production maintenance.*
