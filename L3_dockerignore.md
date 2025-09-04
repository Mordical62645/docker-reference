# .dockerignore File Guide

## Purpose
.dockerignore prevents unnecessary files from being sent to Docker build context, reducing build time and image size.

## Basic .dockerignore Template
```gitignore
# Version control
.git
.gitignore
.gitattributes
.gitmodules

# Documentation
README.md
CHANGELOG.md
CONTRIBUTING.md
docs/
*.md

# Development tools
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
.DS_Store?
Thumbs.db
ehthumbs.db

# Logs
logs/
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Testing
coverage/
.nyc_output/
test/
tests/
__tests__/
*.test.js
*.spec.js

# Build artifacts
build/
dist/
out/
.next/
.nuxt/
target/
tmp/
temp/

# Docker files
Dockerfile*
docker-compose*.yml
.dockerignore

# CI/CD
.github/
.gitlab-ci.yml
Jenkinsfile
.travis.yml
```

## Language-Specific Patterns

### Python Projects
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual environments
venv/
ENV/
env/
.venv/
.ENV/
.env/
pyvenv.cfg

# Jupyter Notebook
.ipynb_checkpoints

# PyCharm
.idea/

# pytest
.pytest_cache/
.coverage

# mypy
.mypy_cache/
.dmypy.json
dmypy.json
```

### Node.js Projects
```gitignore
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.npm
.yarn-integrity

# Build outputs
build/
dist/
.next/
.nuxt/
static/

# Environment files
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Package manager files
package-lock.json
yarn.lock
pnpm-lock.yaml

# IDE
.vscode/
.idea/

# Testing
coverage/
.nyc_output/

# Storybook
storybook-static/
```

### PHP Projects
```gitignore
# Composer
vendor/
composer.phar
composer.lock

# Laravel
storage/app/*
storage/framework/cache/*
storage/framework/sessions/*
storage/framework/testing/*
storage/framework/views/*
storage/logs/*
bootstrap/cache/*
.env
.env.backup
.phpunit.result.cache

# Symfony
var/cache/*
var/logs/*
var/sessions/*
```

### Java Projects
```gitignore
# Compiled class file
*.class

# Package Files
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties

# Gradle
.gradle
build/

# IDE
.idea/
*.iws
*.iml
*.ipr
.classpath
.project
.settings/
bin/
```

### Go Projects
```gitignore
# Binaries
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binary
*.test

# Output of go coverage tool
*.out

# Go workspace file
go.work

# Vendor directory
vendor/

# Dependency directories
Godeps/
```

## Security Patterns
```gitignore
# Secrets and credentials
.env
.env.*
*.key
*.pem
*.crt
*.p12
*.pfx
secrets/
credentials/
keys/
certs/

# Database files
*.db
*.sqlite
*.sqlite3
*.db-journal

# Configuration with sensitive data
config.json
settings.json
appsettings.json
connection-strings.json

# SSH keys
.ssh/
id_rsa
id_rsa.pub
authorized_keys
known_hosts
```

## Development Files
```gitignore
# Editor files
*.swp
*.swo
*~
.vscode/
.idea/
*.sublime-*

# Temporary files
tmp/
temp/
*.tmp
*.temp

# Cache directories
.cache/
cache/

# Local development
local/
dev/
development/

# Backup files
*.bak
*.backup
*.orig
```

## Large Files and Media
```gitignore
# Large media files
*.mp4
*.avi
*.mov
*.mkv
*.wmv
*.flv
*.webm
*.m4v

# Large images (if not needed in container)
*.psd
*.ai
*.sketch
*.fig
*.xcf

# Archives
*.zip
*.tar
*.tar.gz
*.rar
*.7z

# Database dumps
*.sql
*.dump
*.backup
```

## Multi-Service Project Template
```gitignore
# Version control
.git/
.gitignore

# Documentation
*.md
docs/

# Development
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Dependencies (per service)
backend/node_modules/
frontend/node_modules/
api/vendor/

# Build outputs
backend/dist/
frontend/build/
api/target/

# Environment files
.env*
!.env.example

# Testing
coverage/
test-results/

# CI/CD
.github/
.gitlab-ci.yml

# Docker development files
docker-compose.override.yml
docker-compose.dev.yml
```

## Best Practices

### Include Examples
```gitignore
# Exclude all .env files except example
.env*
!.env.example

# Exclude build artifacts but keep structure
build/
!build/.gitkeep
```

### Use Wildcards Efficiently
```gitignore
# All log files
**/*.log

# All node_modules directories
**/node_modules/

# Temporary files anywhere
**/*.tmp
```

### Optimize for Build Context
```gitignore
# Large directories that are never needed
node_modules/
.git/
coverage/
dist/
build/

# Files that change frequently but aren't needed
*.log
*.tmp
.env
```

## Testing Your .dockerignore

Check what gets sent to build context:
```bash
# See build context contents
docker build --no-cache --progress=plain .

# Or use this to see the context
tar -czh . | tar -tv
```

Remember: .dockerignore affects the build context sent to Docker daemon, not what gets copied into the image.
