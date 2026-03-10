# Containerization Concepts — Scenario-Based Questions

---

## S1: Your team is debating whether to use containers or virtual machines for a new project. How do you decide?

**Answer:**

**Use containers when:**
- The application is cloud-native or microservices-based
- You need fast startup times and frequent deployments
- You want consistent environments across dev/staging/production
- You're running many small services
- You need efficient resource utilization

**Use VMs when:**
- You need to run a different operating system (e.g., Windows on Linux host)
- The application requires strong security isolation
- You're running legacy applications that can't be containerized
- You need full OS-level customization
- Compliance requires VM-level isolation

**Hybrid approach (common):**
- Run containers inside VMs for an extra layer of isolation
- Use VMs for databases and stateful services
- Use containers for stateless application services

---

## S2: A container keeps restarting in a crash loop. How do you debug it?

**Answer:**

1. **Check the container status and recent events:**
   ```bash
   docker ps -a                    # See container status and exit codes
   docker inspect <container>      # Detailed container info
   ```

2. **Check container logs:**
   ```bash
   docker logs <container>         # View stdout/stderr
   docker logs --tail 100 <container>  # Last 100 lines
   ```

3. **Common causes and fixes:**

   | Exit Code | Meaning | Common Cause |
   |-----------|---------|-------------|
   | 0 | Success | Container finished its task (might need to keep running) |
   | 1 | General error | Application crash, missing config |
   | 137 | Killed (OOM) | Out of memory — increase memory limit |
   | 139 | Segfault | Application bug |
   | 143 | Terminated | Graceful shutdown signal received |

4. **If the container exits too fast to inspect:**
   ```bash
   # Override the entrypoint to keep it running
   docker run -it --entrypoint /bin/sh myimage
   # Now you can look around inside the container
   ```

5. **Check resource limits:**
   ```bash
   docker stats <container>       # Real-time resource usage
   # If memory usage hits the limit, the container gets killed (OOM)
   ```

6. **Check if dependencies are available:**
   - Can the container reach the database?
   - Are environment variables set correctly?
   - Are required files/volumes mounted?

---

## S3: You need to reduce your Docker image size from 1.2GB to under 200MB. How?

**Answer:**

1. **Use a smaller base image:**
   ```dockerfile
   # Bad: Full Ubuntu (77MB base, grows fast)
   FROM ubuntu:22.04
   
   # Better: Slim variant (30MB)
   FROM python:3.12-slim
   
   # Best: Alpine (5MB)
   FROM python:3.12-alpine
   
   # Best for compiled languages: Scratch (0MB)
   FROM scratch
   ```

2. **Use multi-stage builds:**
   ```dockerfile
   # Stage 1: Build
   FROM node:20 AS builder
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci
   COPY . .
   RUN npm run build
   
   # Stage 2: Production (only the built files)
   FROM nginx:alpine
   COPY --from=builder /app/dist /usr/share/nginx/html
   # Final image is ~25MB instead of 1GB+
   ```

3. **Minimize layers and clean up:**
   ```dockerfile
   # Bad: Multiple RUN commands (each creates a layer)
   RUN apt-get update
   RUN apt-get install -y curl
   RUN apt-get clean
   
   # Good: Single RUN with cleanup
   RUN apt-get update && \
       apt-get install -y --no-install-recommends curl && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/*
   ```

4. **Use .dockerignore:**
   ```
   node_modules
   .git
   *.md
   tests/
   .env
   ```

5. **Don't install unnecessary packages** — only what the app needs to run.