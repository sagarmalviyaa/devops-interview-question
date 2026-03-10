# Docker — Theory Questions

---

## Q1: What is Docker?

**Answer:**
**Docker** is a platform for building, shipping, and running applications in containers. It packages your application with all its dependencies into a standardized unit (container) that runs the same everywhere.

**Key components:**
- **Docker Engine** — The runtime that creates and manages containers
- **Docker CLI** — Command-line tool to interact with Docker
- **Dockerfile** — A script that defines how to build an image
- **Docker Hub** — Public registry for sharing images
- **Docker Compose** — Tool for defining multi-container applications

---

## Q2: What is a Dockerfile? Explain the key instructions.

**Answer:**
A **Dockerfile** is a text file with instructions for building a Docker image.

```dockerfile
# Base image — starting point
FROM node:20-alpine

# Set working directory inside the container
WORKDIR /app

# Copy dependency files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Set environment variable
ENV NODE_ENV=production

# Expose a port (documentation only)
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# Default command to run
CMD ["node", "server.js"]
```

**Key instructions:**

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image to build upon |
| `WORKDIR` | Set the working directory |
| `COPY` | Copy files from host to image |
| `ADD` | Like COPY but can extract archives and fetch URLs |
| `RUN` | Execute a command during build (creates a layer) |
| `ENV` | Set environment variables |
| `EXPOSE` | Document which port the app uses |
| `CMD` | Default command when container starts |
| `ENTRYPOINT` | Main executable (CMD becomes arguments to it) |
| `ARG` | Build-time variables |
| `VOLUME` | Create a mount point for persistent data |
| `USER` | Set the user to run commands as |
| `HEALTHCHECK` | Define how to check if the container is healthy |

---

## Q3: What is the difference between `CMD` and `ENTRYPOINT`?

**Answer:**

- **`CMD`** — The default command. Can be overridden when running the container.
- **`ENTRYPOINT`** — The main executable. Not easily overridden. `CMD` becomes its arguments.

```dockerfile
# CMD only — easily overridden
CMD ["python", "app.py"]
# docker run myimage                → runs: python app.py
# docker run myimage bash           → runs: bash (CMD replaced)

# ENTRYPOINT only — always runs
ENTRYPOINT ["python", "app.py"]
# docker run myimage                → runs: python app.py
# docker run myimage --debug        → runs: python app.py --debug

# ENTRYPOINT + CMD — best practice for CLI tools
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "3000"]
# docker run myimage                → runs: python app.py --port 3000
# docker run myimage --port 8080    → runs: python app.py --port 8080
```

---

## Q4: What are Docker layers? How does caching work?

**Answer:**
Each instruction in a Dockerfile creates a **layer**. Layers are stacked on top of each other to form the final image. Docker caches layers to speed up builds.

```dockerfile
FROM node:20-alpine          # Layer 1 (cached from base image)
WORKDIR /app                 # Layer 2
COPY package*.json ./        # Layer 3
RUN npm install              # Layer 4 (cached if package.json didn't change)
COPY . .                     # Layer 5 (invalidated if any source file changed)
```

**How caching works:**
- Docker checks if each layer has changed since the last build
- If a layer hasn't changed, Docker reuses the cached version
- If a layer changes, all subsequent layers are rebuilt

**Optimization tip:** Put things that change rarely (dependencies) before things that change often (source code). This maximizes cache hits.

---

## Q5: What is a multi-stage build?

**Answer:**
A **multi-stage build** uses multiple `FROM` statements in one Dockerfile. Each stage can use a different base image. Only the final stage becomes the output image.

```dockerfile
# Stage 1: Build (large image with build tools)
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production (small image with only what's needed)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Benefits:**
- **Smaller images** — Final image doesn't include build tools, source code, or dev dependencies
- **Security** — Fewer packages = smaller attack surface
- **Speed** — Smaller images push/pull faster

**Typical size reduction:** 1GB+ → under 100MB

---

## Q6: What is Docker Compose?

**Answer:**
**Docker Compose** is a tool for defining and running multi-container applications. You describe all your services in a `docker-compose.yml` file and start them with one command.

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://db:5432/myapp
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  db-data:
```

```bash
docker compose up -d        # Start all services in background
docker compose down          # Stop and remove all services
docker compose logs -f app   # Follow logs for the app service
docker compose ps            # List running services
```

---

## Q7: What are Docker volumes? What's the difference between volumes and bind mounts?

**Answer:**
Containers are ephemeral — when they're deleted, their data is lost. **Volumes** provide persistent storage.

**Docker Volumes (managed by Docker):**
```bash
docker volume create mydata
docker run -v mydata:/app/data myimage
```

**Bind Mounts (map a host directory):**
```bash
docker run -v /host/path:/container/path myimage
```

| Feature | Volume | Bind Mount |
|---------|--------|-----------|
| Managed by | Docker | You |
| Location | Docker's storage area | Anywhere on host |
| Portability | Easy to backup/migrate | Tied to host path |
| Performance | Optimized | Depends on host filesystem |
| Use case | Production data persistence | Development (live code reload) |

---

## Q8: What is Docker networking? What are the network types?

**Answer:**

| Network Type | Description | Use Case |
|-------------|-------------|----------|
| **bridge** (default) | Isolated network on the host | Containers on same host communicating |
| **host** | Container uses host's network directly | Maximum network performance |
| **none** | No networking | Security-sensitive containers |
| **overlay** | Network spanning multiple hosts | Docker Swarm / multi-host |

```bash
# Create a custom network
docker network create mynetwork

# Run containers on the same network (they can reach each other by name)
docker run --network mynetwork --name web mywebapp
docker run --network mynetwork --name db postgres

# From the web container, you can reach the database as "db:5432"
```

---

## Q9: What are Docker best practices for production?

**Answer:**

1. **Use specific image tags** (not `latest`):
   ```dockerfile
   FROM node:20.11-alpine    # Good: specific version
   FROM node:latest          # Bad: unpredictable
   ```

2. **Run as non-root user:**
   ```dockerfile
   RUN addgroup -S appgroup && adduser -S appuser -G appgroup
   USER appuser
   ```

3. **Use multi-stage builds** to minimize image size

4. **Use `.dockerignore`** to exclude unnecessary files

5. **One process per container** — don't run multiple services in one container

6. **Use health checks:**
   ```dockerfile
   HEALTHCHECK --interval=30s CMD curl -f http://localhost:3000/health || exit 1
   ```

7. **Don't store secrets in images** — use environment variables or secret managers

8. **Scan images for vulnerabilities:**
   ```bash
   trivy image myapp:latest
   ```

9. **Use read-only filesystem when possible:**
   ```bash
   docker run --read-only myapp
   ```

10. **Set resource limits:**
    ```bash
    docker run --memory=512m --cpus=1.0 myapp
    ```

---

## Q10: What is the difference between `docker stop` and `docker kill`?

**Answer:**

- **`docker stop`** — Sends SIGTERM (graceful shutdown), waits 10 seconds, then sends SIGKILL
- **`docker kill`** — Sends SIGKILL immediately (force stop)

```bash
docker stop mycontainer     # Graceful: app can clean up
docker kill mycontainer     # Forceful: immediate termination

docker stop -t 30 mycontainer  # Wait 30 seconds before force killing
```

**Always prefer `docker stop`** — it gives the application time to finish requests, close connections, and save state.

---

## Q11: What is Docker BuildKit?

**Answer:**
**BuildKit** is Docker's modern build engine (default since Docker 23.0). It's faster and more feature-rich than the legacy builder.

**Key features:**
- **Parallel builds** — Independent stages build simultaneously
- **Better caching** — More intelligent layer caching
- **Build secrets** — Mount secrets during build without storing in layers
- **SSH forwarding** — Use SSH keys during build securely
- **Cache mounts** — Cache package manager downloads between builds

```dockerfile
# Using build secrets (never stored in the image)
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm install

# Using cache mounts (speeds up package installs)
RUN --mount=type=cache,target=/root/.npm npm install
```

```bash
# Enable BuildKit (if not default)
DOCKER_BUILDKIT=1 docker build .
```