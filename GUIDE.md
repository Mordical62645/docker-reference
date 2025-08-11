# Docker Containerization Guide

## Understanding Containerization

Think of a project like a toy set that your friend built. You don't know how they made it, you just know:

1. **What pieces they used** (their files, dependencies)
2. **How it's supposed to work** when you play with it (commands to run it)

Containerizing is like putting their toy into a clear plastic box so that no matter where you take it—your house, your cousin's, or school—it will look and work exactly the same.

So even if you didn't build the toy, you can still box it up!

## Universal Containerization Checklist

### Step 1: Get the Recipe
Ask for the recipe that tells you what ingredients (dependencies) the project needs.

Look for these files:
- [ ] `README.md` (instructions)
- [ ] `requirements.txt` (Python)
- [ ] `package.json` (Node.js)
- [ ] `composer.json` (PHP)
- [ ] `Gemfile` (Ruby)
- [ ] `go.mod` (Go)
- [ ] `pom.xml` or `build.gradle` (Java)

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

---

*This guide helps you containerize any project, regardless of the technology stack.*
