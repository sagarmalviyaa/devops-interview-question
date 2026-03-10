# Docker — Scenario-Based Questions

---

## S1: Your application runs on multiple containers and suddenly stops responding. How would you debug it?

**Answer:**

1. **Check container status:**
   ```bash
   docker ps -a
   # Look for containers that are "Exited" or "Restarting"
   ```

2. **Check container logs:**
   ```bash
   docker logs myapp --tail 200
   docker logs myapp --since 10m    # Last 10 minutes
   ```

3. **Check resource usage:**
   ```bash
   docker stats
   # Look for containers hitting CPU or memory limits
   ```

4. **Check container health:**
   ```bash
   docker inspect myapp | grep -A 10 "Health"
   ```

5. **Get inside the container:**
   ```bash
   docker exec -it myapp /bin/sh
   # Check processes, network, disk space inside
   ps aux
   df -h
   curl localhost:3000/health
   ```

6. **Check networking:**
   ```bash
   # Can containers reach each other?
   docker exec myapp ping db
   docker exec myapp nc -zv db 5432
   
   # Check Docker networks
   docker network inspect mynetwork
   ```

7. **Check Docker daemon:**
   ```bash
   systemctl status docker
   journalctl -u docker --since "30 min ago"
   ```

8. **Common causes:**
   - Out of memory (OOM killed) → increase memory limits
   - Disk full on host → clean up images/volumes
   - Network issues between containers → check Docker network
   - Application crash → check logs for errors
   - Database connection pool exhausted → check connection limits

---

## S2: You need to migrate a Docker Compose setup to Kubernetes. What's your approach?

**Answer:**

1. **Map Docker Compose concepts to Kubernetes:**

   | Docker Compose | Kubernetes |
   |---------------|------------|
   | `services:` | Deployments + Services |
   | `ports:` | Service (ClusterIP/NodePort/LoadBalancer) |
   | `volumes:` | PersistentVolumeClaim |
   | `environment:` | ConfigMap / Secret |
   | `depends_on:` | Init containers / readiness probes |
   | `networks:` | Kubernetes networking (automatic) |
   | `replicas:` | Deployment `replicas` field |

2. **Convert each service:**
   ```yaml
   # Docker Compose
   services:
     web:
       image: myapp:1.0
       ports: ["3000:3000"]
       environment:
         DB_HOST: db
   ```
   
   ```yaml
   # Kubernetes Deployment
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: web
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: web
       spec:
         containers:
         - name: web
           image: myapp:1.0
           ports:
           - containerPort: 3000
           env:
           - name: DB_HOST
             value: "db"
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: web
   spec:
     selector:
       app: web
     ports:
     - port: 3000
       targetPort: 3000
   ```

3. **Use tools to help:**
   ```bash
   # Kompose converts docker-compose.yml to Kubernetes manifests
   kompose convert -f docker-compose.yml
   ```

4. **Add Kubernetes-specific features:**
   - Health checks (liveness/readiness probes)
   - Resource limits
   - Horizontal Pod Autoscaler
   - Ingress for external access

---

## S3: A Docker image build is taking 15 minutes. How do you speed it up?

**Answer:**

1. **Optimize layer ordering** (most impactful):
   ```dockerfile
   # Bad: Any code change invalidates npm install cache
   COPY . .
   RUN npm install
   
   # Good: Only re-install when package.json changes
   COPY package*.json ./
   RUN npm ci
   COPY . .
   ```

2. **Use .dockerignore:**
   ```
   node_modules
   .git
   dist
   *.md
   tests/
   .env
   ```

3. **Use BuildKit cache mounts:**
   ```dockerfile
   RUN --mount=type=cache,target=/root/.npm \
       npm ci --only=production
   ```

4. **Use multi-stage builds** to separate build and runtime

5. **Use a smaller base image:**
   ```dockerfile
   FROM node:20-alpine    # Instead of node:20 (saves ~800MB)
   ```

6. **Combine RUN commands:**
   ```dockerfile
   # Bad: 3 layers
   RUN apt-get update
   RUN apt-get install -y curl
   RUN apt-get clean
   
   # Good: 1 layer
   RUN apt-get update && apt-get install -y curl && apt-get clean
   ```

7. **Use Docker build cache in CI:**
   ```bash
   docker build --cache-from myapp:latest -t myapp:new .
   ```

---

## S4: Your Docker container works in development but fails in production with "file not found" errors. What's wrong?

**Answer:**

**Common causes:**

1. **Case sensitivity:** macOS/Windows are case-insensitive, Linux is case-sensitive
   ```
   import from './Components/Header'  # Works on Mac
   # File is actually: ./components/header.js  # Fails on Linux
   ```

2. **Files excluded by .dockerignore:**
   ```bash
   # Check if the file is being ignored
   cat .dockerignore
   # Make sure needed files aren't listed
   ```

3. **Files not copied in Dockerfile:**
   ```dockerfile
   COPY src/ ./src/          # Did you copy all needed directories?
   COPY config/ ./config/    # Don't forget config files
   ```

4. **Volume mounts hiding files:**
   ```yaml
   # A volume mount can hide files that exist in the image
   volumes:
     - ./data:/app/data    # This replaces /app/data in the container
   ```

5. **Different working directory:**
   ```dockerfile
   WORKDIR /app
   # Make sure all paths are relative to /app
   ```

**Debugging:**
```bash
# Check what files are in the container
docker run --rm myapp ls -la /app/
docker run --rm myapp find /app -name "missing-file"
```