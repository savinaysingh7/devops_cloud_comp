# Docker For Absolute Beginners

This guide is written for someone starting from zero. No prior Docker knowledge is assumed.

## 1. Docker in Very Simple Words

Think of Docker like this:
- You have a lunchbox (container).
- Inside it, you pack food + spoon + napkin (app + dependencies + runtime).
- Anyone can open the same lunchbox anywhere and get the same meal.

In software terms:
- Image: a ready-made package (blueprint/template).
- Container: a running copy of that package.
- Dockerfile: instructions to create the package.

Why this matters:
- "Works on my machine" problems become much smaller.
- Team setup becomes faster.
- Deployment becomes more predictable.

## 2. One-Minute Vocabulary

- Docker Engine: the service that runs containers.
- Docker CLI: command line tool (`docker ...`).
- Image: packaged app template.
- Container: running app instance from an image.
- Registry: image store on the internet (Docker Hub, GHCR, ECR).
- Volume: persistent data location managed by Docker.
- Bind mount: your local folder attached inside a container.
- Docker Compose: run multiple containers using one file.

## 3. Confirm Your Setup (Windows PowerShell)

You already ran `docker run hello-world`, which is perfect.

You can recheck any time:

```powershell
docker --version
docker info
docker run hello-world
```

If these commands work, you are ready.

## 3.1 Very Important Rule (Avoid This Exact Error)

Do not type Dockerfile instructions directly in PowerShell.

Example of Dockerfile instruction:

```dockerfile
FROM nginx:alpine
```

This line must be written inside a file named `Dockerfile`, not executed as a terminal command.

Terminal commands start with `docker ...` (for example `docker build ...`, `docker run ...`).

## 4. Your First Real Container (Nginx)

This is your "Hello, Docker" project.

1. Start a container:

```powershell
docker run -d --name myweb -p 8080:80 nginx:latest
```

2. Open browser:
- Visit `http://localhost:8080`
- You should see the Nginx welcome page.

3. Check running containers:

```powershell
docker ps
```

4. View logs:

```powershell
docker logs myweb
```

5. Stop and remove it:

```powershell
docker stop myweb
docker rm myweb
```

What you just learned:
- `docker run` starts a container.
- `-d` runs in background.
- `--name` gives a friendly name.
- `-p 8080:80` maps host port to container port.

## 5. Important Mental Model (Image vs Container)

New learners often confuse these two. Keep this rule:
- Image is fixed (like a class template).
- Container is temporary runtime (like an object instance).

You can run many containers from one image.

Useful commands:

```powershell
docker images
docker ps
docker ps -a
```

## 6. Build Your Own Image (Mini App)

Below is a tiny static website example.

### Step A: Create files

If you want no mistakes, use this exact PowerShell flow.

1. Go to your project folder:

```powershell
cd C:\Users\savin\Documents\01_Projects\Active\devops_cloud_comp
```

2. Create `index.html` from terminal:

```powershell
@"
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>My First Docker App</title>
</head>
<body>
  <h1>Hello from Docker</h1>
  <p>If you can read this, your custom image works.</p>
</body>
</html>
"@ | Set-Content -Path .\index.html
```

3. Create `Dockerfile` from terminal:

```powershell
@"
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
"@ | Set-Content -Path .\Dockerfile
```

4. Confirm both files exist:

```powershell
Get-ChildItem .\index.html, .\Dockerfile
```

Create `index.html`:

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>My First Docker App</title>
</head>
<body>
  <h1>Hello from Docker</h1>
  <p>If you can read this, your custom image works.</p>
</body>
</html>
```

Create `Dockerfile` in same folder:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

Important:
- Filename must be exactly `Dockerfile` (no extension like `.txt`).
- The `Dockerfile` and `index.html` must be in same folder while building.

### Step B: Build image

```powershell
docker build -t my-static-site:1.0 .
```

If build fails, run this first to verify current folder:

```powershell
Get-Location
Get-ChildItem
```

### Step C: Run container

```powershell
docker run -d --name site1 -p 8090:80 my-static-site:1.0
```

Open `http://localhost:8090`.

If you get "container name already in use", run:

```powershell
docker rm -f site1
docker run -d --name site1 -p 8090:80 my-static-site:1.0
```

### Step D: Clean up

```powershell
docker stop site1
docker rm site1
```

### Step E: One Copy-Paste Block (No-Error Path)

If you prefer one block, run this exactly:

```powershell
cd C:\Users\savin\Documents\01_Projects\Active\devops_cloud_comp
@"
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>My First Docker App</title>
</head>
<body>
  <h1>Hello from Docker</h1>
</body>
</html>
"@ | Set-Content -Path .\index.html
@"
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
"@ | Set-Content -Path .\Dockerfile
docker build -t my-static-site:1.0 .
docker rm -f site1 2>$null
docker run -d --name site1 -p 8090:80 my-static-site:1.0
docker ps --filter name=site1
```

## 7. Most Useful Commands (Daily Use)

```powershell
# Images
docker images
docker pull redis:latest
docker build -t app:1.0 .
docker rmi app:1.0

# Containers
docker run -d --name app1 -p 8080:80 nginx
docker ps
docker ps -a
docker stop app1
docker start app1
docker restart app1
docker rm app1

# Logs and shell
docker logs -f app1
docker exec -it app1 sh

# Cleanup
docker system df
docker image prune -f
docker container prune -f
docker volume prune -f
docker system prune -f
```

Tip:
- `prune` removes unused items. Run carefully.

## 8. Data: Why Volumes Matter

Problem:
- If a container is deleted, its internal data is usually lost.

Solution:
- Use volumes for persistent data.

Example (PostgreSQL data persistence):

```powershell
docker volume create pgdata
docker run -d --name mydb -e POSTGRES_PASSWORD=secret -v pgdata:/var/lib/postgresql/data postgres:16
```

Now database files stay safe even if container is recreated.

## 9. Bind Mounts for Development

Bind mounts connect your local folder with container folder.

PowerShell example:

```powershell
docker run -d --name dev-nginx -p 8081:80 -v ${PWD}\html:/usr/share/nginx/html nginx:latest
```

When you edit local files, container sees updates immediately.

Use bind mounts mainly for local development.

## 10. Docker Networking in Plain English

By default, containers are isolated.

If two containers need to talk (API -> DB), place both on same network.

```powershell
docker network create appnet
docker run -d --name db --network appnet postgres:16
docker run -d --name api --network appnet my-api:1.0
```

Inside this network, `api` can reach database using hostname `db`.

## 11. Docker Compose (Run Full Stack Easily)

Compose lets you start multiple services with one command.

Create `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  redis:
    image: redis:latest
```

Run:

```powershell
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```

This is the easiest way to run multi-container apps.

## 12. Beginner Debugging Checklist

When something fails, do this in order:

1. Is container running?

```powershell
docker ps -a
```

2. What do logs say?

```powershell
docker logs <container_name>
```

3. Can you inspect settings?

```powershell
docker inspect <container_name>
```

4. Can you enter shell?

```powershell
docker exec -it <container_name> sh
```

Top beginner mistakes:
- Wrong port mapping.
- App listens on localhost instead of 0.0.0.0 inside container.
- Files not copied due to wrong Dockerfile path.
- Container exits because startup command fails.
- Dockerfile lines typed in terminal (for example `FROM nginx:alpine`).

## 13. Security Basics (Start Early)

- Use official/trusted base images.
- Avoid using `latest` in production.
- Keep images updated.
- Never hardcode passwords/secrets in Dockerfile.
- Scan images for vulnerabilities.

Quick check:

```powershell
docker history my-static-site:1.0
```

## 14. Beginner-Friendly Dockerfile Tips

Good habits from day one:
- Keep Dockerfiles small and readable.
- Copy dependency files before source code (improves cache).
- Use `.dockerignore` so unnecessary files are not sent to build.
- Pin versions for predictable builds.

Example `.dockerignore`:

```text
node_modules
.git
.env
dist
coverage
```

## 15. Day 1 Practice Plan (Hands-On)

Follow this exact flow once:

1. Run Nginx container.
2. Open browser and verify page.
3. Check logs.
4. Stop and remove container.
5. Create your own `index.html` + `Dockerfile`.
6. Build custom image.
7. Run custom container.
8. Remove resources.

If you can complete these 8 steps, your Docker foundation is strong.

## 16. Day 2 Practice Plan

1. Add a volume to a database container.
2. Delete and recreate the DB container.
3. Verify data still exists.
4. Run two services with Compose.
5. Bring stack down and up again.

## 17. Day 3 Practice Plan

1. Build image twice and observe caching speed.
2. Intentionally break a container command.
3. Diagnose using logs and inspect.
4. Fix and rerun.

## 18. Practical Cheat Sheet

```powershell
# Build
docker build -t app:1.0 .

# Run
docker run -d --name app -p 8080:8080 app:1.0

# Logs
docker logs -f app

# Enter shell
docker exec -it app sh

# Stop/remove
docker stop app
docker rm app

# Compose
docker compose up -d
docker compose down
```

## 19. Interview-Style Quick Q and A

Q: What is Docker image?
A: Packaged template containing app + runtime + dependencies.

Q: What is a container?
A: Running instance of an image.

Q: Image vs container?
A: Image is static template; container is live runtime process.

Q: Why use Docker Compose?
A: To run multi-service stacks with a single YAML file and command.

Q: What problem do volumes solve?
A: Persistent data beyond container lifetime.

## 20. Final Guidance for a New Learner

Do not try to memorize everything. Learn in this order:

1. Run containers.
2. Read logs.
3. Build your own image.
4. Use volumes.
5. Use Compose.

Repeat this loop every day:

1. Build.
2. Run.
3. Test.
4. Read logs.
5. Fix.

Consistency beats complexity. After 7 to 10 days of practice, Docker will feel natural.

## 21. How Docker Works Internally (Simple but Accurate)

When you run a Docker command, this happens:

1. You type a command in terminal (Docker CLI).
2. CLI sends request to Docker Engine.
3. Engine checks if required image is present locally.
4. If not present, image is pulled from a registry.
5. Engine creates container using image layers.
6. Engine starts container process.

Technical terms you may hear later:
- Namespaces: isolate process, network, filesystem views.
- Cgroups: limit resource usage (CPU, memory).
- Union filesystem: image layers stacked efficiently.

You do not need to master these now, but this is why containers are fast.

## 22. Full Container Lifecycle (Start to Finish)

Container states:
- Created: container defined but not running.
- Running: process active.
- Exited: process stopped.
- Removed: container deleted.

Useful lifecycle commands:

```powershell
docker create --name demo nginx:latest
docker start demo
docker stop demo
docker rm demo
```

One-line shortcut (create + start):

```powershell
docker run -d --name demo nginx:latest
```

Inspect lifecycle quickly:

```powershell
docker ps
docker ps -a
```

## 23. Command Patterns You Will Use Every Day

Pattern A: Pull + Run

```powershell
docker pull redis:latest
docker run -d --name redis1 -p 6379:6379 redis:latest
```

Pattern B: Build + Run

```powershell
docker build -t web:1.0 .
docker run -d --name web1 -p 8080:80 web:1.0
```

Pattern C: Diagnose + Fix

```powershell
docker ps -a
docker logs web1
docker inspect web1
docker rm -f web1
docker run -d --name web1 -p 8080:80 web:1.0
```

## 24. Dockerfile Masterclass for Beginners

### 24.1 Order matters for speed

Bad order (slow rebuilds):

```dockerfile
COPY . .
RUN npm ci
```

Better order (faster rebuilds):

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

### 24.2 Use explicit versions

Prefer:

```dockerfile
FROM node:20-alpine
```

Avoid only latest for production:

```dockerfile
FROM node:latest
```

### 24.3 Separate build and runtime (multi-stage)

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=build /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

Result:
- Smaller runtime image.
- Better security and faster deployments.

## 25. Compose Deep Dive (Beginner to Intermediate)

Use this template when you need app + database.

```yaml
services:
  app:
    build: .
    container_name: app_service
    ports:
      - "3000:3000"
    environment:
      APP_ENV: development
      DATABASE_URL: postgres://postgres:postgres@db:5432/appdb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    container_name: db_service
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: appdb
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
```

Commands:

```powershell
docker compose up -d --build
docker compose ps
docker compose logs -f app
docker compose down
```

If you want to remove data volume too:

```powershell
docker compose down -v
```

## 26. Resource Limits (Important for Stable Systems)

Without limits, one container can consume too many resources.

Run with memory and CPU limits:

```powershell
docker run -d --name limited-app --memory="512m" --cpus="1.0" my-app:1.0
```

Why this matters:
- Prevents system slowdown.
- Improves predictability in shared environments.

## 27. Backup and Restore for Volumes

### Backup volume example

```powershell
docker run --rm -v pgdata:/data -v ${PWD}:/backup alpine tar czf /backup/pgdata-backup.tar.gz -C /data .
```

### Restore volume example

```powershell
docker run --rm -v pgdata:/data -v ${PWD}:/backup alpine sh -c "cd /data && tar xzf /backup/pgdata-backup.tar.gz"
```

This is useful before risky cleanup commands.

## 28. Troubleshooting Matrix (Symptom -> Cause -> Fix)

Symptom: Browser shows cannot connect
- Cause: wrong host port or container not running
- Fix: check docker ps and port mapping

Symptom: Build is very slow
- Cause: large build context or poor Dockerfile order
- Fix: improve .dockerignore and instruction order

Symptom: Changes in local code not reflected
- Cause: image not rebuilt or no bind mount used
- Fix: rebuild image or mount local folder

Symptom: Database data disappeared
- Cause: data stored only in container layer
- Fix: use Docker volume

Symptom: Permission denied in container
- Cause: user/ownership mismatch
- Fix: adjust file permissions and run with proper user

Symptom: Command works manually but fails in container
- Cause: missing runtime dependencies inside image
- Fix: ensure Dockerfile installs required packages

## 29. Production Readiness Checklist

Before shipping any containerized app, verify:

1. Image tags are explicit, not only latest.
2. Health checks are defined.
3. Logs go to stdout and stderr.
4. Secrets are not hardcoded.
5. Data services use volumes.
6. CPU and memory limits are set.
7. Vulnerability scan is done.
8. Rollback image tag is available.

## 30. Interview Preparation Add-On

Common interview questions:

Q1: Difference between virtual machine and container?
- VM virtualizes full OS, heavier startup and footprint.
- Container shares host kernel, lighter and faster.

Q2: Difference between CMD and ENTRYPOINT?
- CMD provides default arguments/command.
- ENTRYPOINT defines primary executable.

Q3: Why multi-stage builds?
- To keep runtime image smaller and cleaner.

Q4: Why use Compose?
- To define and run multi-service applications reproducibly.

Q5: Why avoid latest in production?
- Because behavior may change unexpectedly between pulls.

## 31. Advanced 30-Day Practice Plan

Days 1-5:
- Run public images and practice lifecycle commands.
- Build two simple images from Dockerfile.

Days 6-10:
- Use bind mounts for live development.
- Use volumes for durable data.

Days 11-15:
- Build a two-container app with Compose.
- Add health checks and restart policy.

Days 16-20:
- Practice troubleshooting from logs and inspect output.
- Intentionally break and fix 5 scenarios.

Days 21-25:
- Apply image optimization using multi-stage builds.
- Add .dockerignore improvements.

Days 26-30:
- Add tagging strategy and push to registry.
- Prepare production checklist and interview revision.

## 32. Final Note

Learning Docker is a hands-on skill.
If one command fails, that is normal and valuable.

Use this workflow for every problem:
1. Reproduce.
2. Read logs.
3. Inspect settings.
4. Fix one thing.
5. Retry.

If you keep this loop, your Docker confidence will grow very fast.

## 33. Publish Your GitHub Repository to Docker Hub (Complete Guide)

This section shows two methods:
1. Manual publish from your machine.
2. Automatic publish using GitHub Actions.

Use method 1 first. Then move to method 2 for automation.

### 33.1 Prerequisites

Before publishing, ensure:
1. Your project has a valid `Dockerfile` in repository root.
2. Docker is running locally.
3. You have a Docker Hub account.
4. Your GitHub repository code is pushed.

Optional but recommended:
1. Add `.dockerignore` to reduce build size.
2. Use version tags (for example `1.0.0`) instead of only `latest`.

### 33.2 Example Repository Structure

```text
my-repo/
  Dockerfile
  .dockerignore
  src/
  package.json
  README.md
```

### 33.3 Step 1: Create Docker Hub Repository

In Docker Hub website:
1. Click Create Repository.
2. Name it (example: `my-app`).
3. Choose visibility: public or private.
4. Create repository.

Your image path pattern becomes:

`dockerhub_username/repository_name:tag`

Example:

`savin/my-app:1.0.0`

### 33.4 Step 2: Build Image Locally

Run in repository folder:

```powershell
cd C:\path\to\your\repo
docker build -t my-app:1.0.0 .
```

Test image before pushing:

```powershell
docker run --rm -p 8080:8080 my-app:1.0.0
```

If app starts and works in browser/API test, continue.

### 33.5 Step 3: Tag Image for Docker Hub

Syntax:

```powershell
docker tag local_image:tag dockerhub_username/repository_name:tag
```

Example:

```powershell
docker tag my-app:1.0.0 savin/my-app:1.0.0
docker tag my-app:1.0.0 savin/my-app:latest
```

### 33.6 Step 4: Login to Docker Hub

```powershell
docker login
```

Enter your Docker Hub username and password or access token.

### 33.7 Step 5: Push Image

```powershell
docker push savin/my-app:1.0.0
docker push savin/my-app:latest
```

Verify in Docker Hub web UI that new tags appear.

### 33.8 Step 6: Test Pull from Anywhere

```powershell
docker pull savin/my-app:1.0.0
docker run --rm -p 8080:8080 savin/my-app:1.0.0
```

If this works on another machine, your publish flow is correct.

### 33.9 Recommended Tagging Strategy

Use at least two tags on release:
1. Immutable release tag, example `1.0.0`.
2. Mutable convenience tag, example `latest`.

Recommended for teams:
1. `1.0.0`
2. `1.0`
3. `latest`
4. Commit tag, example `sha-a1b2c3d`

Avoid using only `latest` in production deployments.

### 33.10 Connect GitHub to Docker Hub with GitHub Actions (Auto Push)

This is best for CI/CD.

#### A) Create Docker Hub access token

In Docker Hub:
1. Account Settings.
2. Security.
3. New Access Token.
4. Copy token once and store safely.

#### B) Add GitHub secrets

In GitHub repository:
1. Settings.
2. Secrets and variables.
3. Actions.
4. Add secret `DOCKERHUB_USERNAME`.
5. Add secret `DOCKERHUB_TOKEN`.

#### C) Add workflow file

Create `.github/workflows/docker-publish.yml`:

```yaml
name: Docker Publish

on:
  push:
    branches: ["main"]
    tags: ["v*"]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKERHUB_USERNAME }}/my-app
          tags: |
            type=raw,value=latest,enable={{is_default_branch}}
            type=ref,event=tag
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

Now each push to `main` or version tag can publish image automatically.

### 33.11 Release Workflow (Practical)

Manual release example:
1. Update code.
2. Build and test locally.
3. Tag image as `1.1.0` and `latest`.
4. Push both tags.
5. Deploy using immutable version tag.

Automated release example:
1. Push code to `main` for `latest` CI image.
2. Create git tag `v1.1.0`.
3. GitHub Actions publishes versioned image.
4. Deploy using `1.1.0` tag.

### 33.12 Common Publishing Errors and Fixes

Error: `requested access to the resource is denied`
1. Cause: Wrong repository name or no permission.
2. Fix: Check tag format and Docker Hub repository visibility/ownership.

Error: `denied: requested access to the resource is denied`
1. Cause: Not logged in or token invalid.
2. Fix: Run `docker login` again or rotate token.

Error: `manifest unknown`
1. Cause: Pulling a tag that does not exist.
2. Fix: Verify tag in Docker Hub UI and pull exact tag.

Error: Build works locally but fails in GitHub Actions
1. Cause: Missing files, wrong paths, secret not set, or architecture mismatch.
2. Fix: Check workflow logs, verify context path, verify secrets, add Buildx config.

Error: Image starts then exits immediately
1. Cause: Wrong `CMD` or app crash.
2. Fix: Check `docker logs`, then fix startup command.

### 33.13 Security and Quality Best Practices for Docker Hub Images

1. Never put secrets in image layers.
2. Use non-root user where possible.
3. Scan images regularly.
4. Pin base image versions.
5. Keep images small using multi-stage builds.
6. Add README with run instructions and exposed ports.

### 33.14 Fast Copy-Paste Manual Publish Flow

Replace placeholders and run:

```powershell
cd C:\path\to\your\repo
docker build -t my-app:1.0.0 .
docker tag my-app:1.0.0 DOCKERHUB_USERNAME/my-app:1.0.0
docker tag my-app:1.0.0 DOCKERHUB_USERNAME/my-app:latest
docker login
docker push DOCKERHUB_USERNAME/my-app:1.0.0
docker push DOCKERHUB_USERNAME/my-app:latest
docker pull DOCKERHUB_USERNAME/my-app:1.0.0
```

If all commands pass, your GitHub project is now packaged and published on Docker Hub.
