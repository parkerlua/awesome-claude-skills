---
name: docker
description: Write Dockerfiles, Compose files, and debug containers. Use when asked to containerize an app, write Docker config, or troubleshoot container issues.
---

You are containerizing applications. Your goal: images that are small, secure, fast to build, and behave identically in dev and production.

## Dockerfile Best Practices

### Layer Order — cache expensive steps first

```dockerfile
# ✅ Dependencies change less often than code — cache them
COPY package.json package-lock.json ./
RUN npm ci --production

COPY . .
RUN npm run build

# ❌ This busts the cache on every code change
COPY . .
RUN npm ci
RUN npm run build
```

### Use multi-stage builds to keep images small

```dockerfile
# Stage 1: build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: runtime — only what's needed to run
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Base image choices

| Use case | Image |
|----------|-------|
| Node.js | `node:20-alpine` |
| Python | `python:3.12-slim` |
| Go | `golang:1.22-alpine` → `scratch` or `distroless` |
| General Linux | `debian:bookworm-slim` |

Always pin to a specific version, never `latest`. Alpine for small size, slim for compatibility.

### Security

```dockerfile
# Run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Don't copy sensitive files
# Use .dockerignore:
# .env
# .git
# node_modules
# *.log

# Read-only filesystem where possible
# Set in docker run or compose: --read-only
```

### Always include a .dockerignore

```
.git
.env*
node_modules
*.log
.DS_Store
dist
coverage
.github
README.md
```

## Docker Compose

```yaml
version: "3.9"

services:
  app:
    build:
      context: .
      target: runner          # specify multi-stage target
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}   # from .env file, never hardcoded
    depends_on:
      db:
        condition: service_healthy    # wait for health check, not just start
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

**Rules:**
- Never hardcode secrets — use environment variables from `.env`
- Use named volumes for persistent data, not bind mounts in production
- Add health checks to stateful services so dependents wait properly
- Use `restart: unless-stopped` for production services

## Debugging Containers

```bash
# Get a shell in a running container
docker exec -it <container_name> sh

# See logs (follow mode)
docker logs -f <container_name>

# Inspect environment variables
docker exec <container_name> env

# Check what's inside an image layer by layer
docker run --rm -it <image> sh

# See resource usage
docker stats

# Inspect a stopped container
docker inspect <container_id>

# Check why a container exited
docker inspect <container_id> --format='{{.State.ExitCode}} {{.State.Error}}'
```

## Common Problems

**Container exits immediately:**
- Missing `CMD` or entrypoint
- Process exits (use `tail -f /dev/null` to keep alive for debugging)
- Permission denied on script (add `chmod +x`)

**Can't connect between containers:**
- Use the service name as hostname, not `localhost`
- Check they're on the same network (`docker network ls`)

**Image too large:**
- Switch to alpine/slim base
- Add multi-stage build
- Check `.dockerignore` is excluding `node_modules`

**Build cache not working:**
- Order matters — put COPY for deps before COPY for source
- `--no-cache` flag bypasses it entirely (useful for CI)
