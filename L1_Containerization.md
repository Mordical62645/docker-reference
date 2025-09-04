# Docker Containerization Guide

## Understanding Containerization

Think of a project like a toy set that your friend built. You don't know how they made it, you just know:

1. **What pieces they used** (their files, dependencies)
2. **How it's supposed to work** when you play with it (commands to run it)

Containerizing is like putting their toy into a clear plastic box so that no matter where you take it—your house, your cousin's, or school—it will look and work exactly the same.

So even if you didn't build the toy, you can still box it up.

## Universal Containerization Checklist

### Step 1: Get the Recipe
Ask for the recipe that tells you what ingredients (dependencies) the project needs.

Look for these dependency files:

**Documentation & Instructions:**
- [ ] `README.md` or `README.txt` (main instructions)
- [ ] `INSTALL.md` or `INSTALLATION.md` (installation guide)
- [ ] `CONTRIBUTING.md` (contribution guidelines)
- [ ] `CHANGELOG.md` (version history)
- [ ] `docs/` folder (detailed documentation)

**Language-Specific Dependencies:**
- [ ] **Python:** `requirements.txt`, `Pipfile`, `pyproject.toml`, `setup.py`, `environment.yml` (conda), `poetry.lock`
- [ ] **Node.js:** `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `.nvmrc`
- [ ] **PHP:** `composer.json`, `composer.lock`
- [ ] **Ruby:** `Gemfile`, `Gemfile.lock`, `.ruby-version`
- [ ] **Go:** `go.mod`, `go.sum`, `vendor/` folder
- [ ] **Java:** `pom.xml` (Maven), `build.gradle` (Gradle), `ivy.xml` (Ivy)
- [ ] **C#/.NET:** `*.csproj`, `*.sln`, `packages.config`, `Directory.Build.props`
- [ ] **Rust:** `Cargo.toml`, `Cargo.lock`
- [ ] **Swift:** `Package.swift`, `Package.resolved`
- [ ] **Kotlin:** `build.gradle.kts`, `pom.xml`
- [ ] **Scala:** `build.sbt`, `project/` folder
- [ ] **R:** `DESCRIPTION`, `renv.lock`, `packrat/`
- [ ] **Julia:** `Project.toml`, `Manifest.toml`
- [ ] **Perl:** `cpanfile`, `META.json`
- [ ] **Elixir:** `mix.exs`, `mix.lock`

**Frontend & Web Dependencies:**
- [ ] `bower.json` (Bower)
- [ ] `webpack.config.js` (Webpack)
- [ ] `vite.config.js` (Vite)
- [ ] `rollup.config.js` (Rollup)
- [ ] `gulpfile.js` (Gulp)
- [ ] `grunt.js` (Grunt)
- [ ] `angular.json` (Angular)
- [ ] `vue.config.js` (Vue)
- [ ] `next.config.js` (Next.js)
- [ ] `nuxt.config.js` (Nuxt.js)
- [ ] `svelte.config.js` (Svelte)

**Configuration Files:**
- [ ] `.env` files (environment variables)
- [ ] `config/` folder (application configs)
- [ ] `settings.json`, `appsettings.json` (app settings)
- [ ] `.ini`, `.yaml`, `.yml`, `.toml` files (configs)
- [ ] `Makefile` (build instructions)
- [ ] `CMakeLists.txt` (CMake)
- [ ] `configure.ac`, `Makefile.in` (Autotools)
- [ ] `meson.build` (Meson)

**Database & Data Files:**
- [ ] Database migration files (`migrations/`, `db/migrate/`)
- [ ] Database seed files (`seeds/`, `fixtures/`)
- [ ] SQL schema files (`schema.sql`, `*.sql`)
- [ ] Database config files (`database.yml`, `db.json`)
- [ ] CSV, JSON, XML data files that the app requires

**Static Assets & Media (YES, these are dependencies!):**
- [ ] **Images:** `*.png`, `*.jpg`, `*.jpeg`, `*.gif`, `*.svg`, `*.ico`, `*.webp`
- [ ] **Videos:** `*.mp4`, `*.avi`, `*.mov`, `*.webm`, `*.mkv`
- [ ] **Audio:** `*.mp3`, `*.wav`, `*.ogg`, `*.m4a`
- [ ] **Fonts:** `*.ttf`, `*.otf`, `*.woff`, `*.woff2`
- [ ] **CSS/Styles:** `*.css`, `*.scss`, `*.sass`, `*.less`, `*.styl`
- [ ] **Static files:** `public/`, `static/`, `assets/`, `media/` folders

**Container & Deployment Files:**
- [ ] `Dockerfile`, `Dockerfile.*` (existing containers)
- [ ] `docker-compose.yml`, `docker-compose.*.yml`
- [ ] `.dockerignore` (files to exclude)
- [ ] `k8s/`, `kubernetes/` folders (Kubernetes manifests)
- [ ] `helm/` charts
- [ ] `ansible/` playbooks
- [ ] `terraform/` infrastructure code

**CI/CD & Tooling:**
- [ ] `.github/workflows/` (GitHub Actions)
- [ ] `.gitlab-ci.yml` (GitLab CI)
- [ ] `Jenkinsfile` (Jenkins)
- [ ] `.circleci/config.yml` (CircleCI)
- [ ] `.travis.yml` (Travis CI)
- [ ] `azure-pipelines.yml` (Azure DevOps)

**Version Control & Ignore Files:**
- [ ] `.gitignore`, `.gitattributes`
- [ ] `.gitmodules` (git submodules)
- [ ] `.hgignore` (Mercurial)

**Development Tools:**
- [ ] `.editorconfig` (editor settings)
- [ ] `tslint.json`, `eslint.json` (linting)
- [ ] `.prettierrc` (code formatting)
- [ ] `jest.config.js`, `vitest.config.js` (testing)
- [ ] `.babelrc` (Babel transpiler)
- [ ] `tsconfig.json` (TypeScript)
- [ ] `jsconfig.json` (JavaScript)

**Virtual Environments & Containers:**
- [ ] `Vagrantfile` (Vagrant)
- [ ] `.devcontainer/` (VS Code dev containers)
- [ ] `flake.nix` (Nix)
- [ ] `shell.nix` (Nix shell)

**Mobile Development:**
- [ ] **Android:** `build.gradle`, `gradle.properties`, `local.properties`
- [ ] **iOS:** `Podfile`, `Podfile.lock` (CocoaPods), `Cartfile` (Carthage)
- [ ] **React Native:** `metro.config.js`, `react-native.config.js`
- [ ] **Flutter:** `pubspec.yaml`, `pubspec.lock`
- [ ] **Xamarin:** `*.csproj`, `packages.config`

### Step 2: Find the "Start Button"
Figure out how to start the app. This will be the `CMD` or `ENTRYPOINT` in your Dockerfile.

Common start commands:
- [ ] **Python** → `python main.py` or `python app.py`
- [ ] **Node.js** → `npm start` or `node index.js`
- [ ] **PHP** → `php artisan serve` or `php index.php`
- [ ] **Ruby** → `ruby app.rb` or `rails server`
- [ ] **Go** → `go run main.go` or `./app`
- [ ] **Java** → `java -jar app.jar`

### Step 3: Write the Box Instructions (Dockerfile)
Create a `Dockerfile` with these steps:
- [ ] Pick a base image (Python, Node, etc.)
- [ ] Copy the files in
- [ ] Install the dependencies
- [ ] Tell it what command to run

### Step 4: Write docker-compose.yml (if needed)
If the app needs multiple services (like app + database), create a `docker-compose.yml`:
- [ ] Define the main application service
- [ ] Add database service (if needed)
- [ ] Configure networking between services
- [ ] Set up volumes for data persistence

### Step 5: Test the Box
Make sure everything works as expected:
- [ ] Run `docker build -t <image-name> .`
- [ ] Run `docker run <image-name>` (or with appropriate flags)
- [ ] Or run `docker-compose up --build`
- [ ] Verify the application works exactly like the original

## Quick Reference

### Essential Docker Commands
```bash
# Build an image
docker build -t my-app .

# Run a container
docker run -p 8080:8080 my-app

# Run with docker-compose
docker-compose up --build

# Stop and remove containers
docker-compose down
```

### Common Dockerfile Template
```dockerfile
FROM <base-image>
WORKDIR /app
COPY . .
RUN <install-dependencies-command>
EXPOSE <port>
CMD ["<start-command>"]
```

## Remember
If you follow this process for every project, you don't need to be part of the development team. You just need to:
1. Get the recipe
2. Find the start button  
3. Make a box that runs it anywhere

## Infrastructure Requirements Checklist

### System Requirements
- [ ] **Operating System:** Linux (production), Windows/macOS (development)
- [ ] **CPU Architecture:** x86_64 (AMD64), ARM64 (for M1 Macs, ARM servers)
- [ ] **Memory:** Minimum 2GB RAM for Docker, 4GB+ recommended
- [ ] **Storage:** 20GB+ free space for images and containers
- [ ] **Docker Version:** 20.10+ (check with `docker --version`)
- [ ] **Docker Compose Version:** 2.0+ (check with `docker-compose --version`)

### Network Requirements
- [ ] **Ports:** Identify all ports your application uses
- [ ] **External Services:** APIs, databases, caches the app connects to
- [ ] **Load Balancing:** If multiple instances needed
- [ ] **SSL/TLS:** Certificate requirements for HTTPS
- [ ] **Domain Names:** Custom domains and DNS configuration

### Data Requirements
- [ ] **Database:** Type (PostgreSQL, MySQL, MongoDB, SQLite)
- [ ] **Data Persistence:** What data needs to survive container restarts
- [ ] **Backup Strategy:** How and when to backup data
- [ ] **Migration Scripts:** Database schema updates
- [ ] **Seed Data:** Initial data the application needs

### Security Requirements
- [ ] **Secrets:** API keys, passwords, certificates
- [ ] **User Permissions:** Non-root user setup
- [ ] **Network Security:** Firewall rules, VPN access
- [ ] **Image Security:** Base image vulnerabilities
- [ ] **Registry Access:** Private registry authentication

### Environment Requirements
- [ ] **Environment Variables:** All required env vars documented
- [ ] **Configuration Files:** Config that varies by environment
- [ ] **Timezone:** Application timezone requirements (like Asia/Manila)
- [ ] **Locale:** Language and region settings
- [ ] **Feature Flags:** Environment-specific features

### Performance Requirements
- [ ] **Resource Limits:** CPU and memory constraints
- [ ] **Scaling:** Horizontal scaling requirements
- [ ] **Caching:** Redis, Memcached, or other cache needs
- [ ] **Monitoring:** Health checks and metrics collection
- [ ] **Logging:** Log aggregation and retention

### Deployment Requirements
- [ ] **CI/CD Pipeline:** Automated build and deployment
- [ ] **Registry:** Where to store Docker images
- [ ] **Orchestration:** Docker Compose, Kubernetes, or Docker Swarm
- [ ] **Blue/Green Deployment:** Zero-downtime deployment strategy
- [ ] **Rollback Plan:** How to revert to previous version

### External Dependencies
- [ ] **Third-party Services:** Payment processors, email services, etc.
- [ ] **CDN:** Content delivery network for static assets
- [ ] **Message Queues:** RabbitMQ, Apache Kafka, AWS SQS
- [ ] **Search Engines:** Elasticsearch, Solr, or similar
- [ ] **Analytics:** Google Analytics, monitoring services

### Development vs Production Differences
- [ ] **Development:** Hot reload, debug mode, test databases
- [ ] **Staging:** Production-like environment for testing
- [ ] **Production:** Optimized builds, secure configurations, monitoring
- [ ] **Resource Allocation:** Different CPU/memory per environment
- [ ] **Data Volume:** Development uses sample data, production uses real data

---

*This guide helps you containerize any project, regardless of the technology stack.*
