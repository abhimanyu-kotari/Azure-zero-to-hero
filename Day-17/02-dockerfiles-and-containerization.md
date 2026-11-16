# Multi-Stage Dockerfiles for Polyglot Microservices

Writing lightweight, secure, and production-ready `Dockerfiles` is a prerequisite for deploying applications to Kubernetes.

---

## Dockerfile Best Practices for DevOps

1. **Use Multi-Stage Builds:** Separate build tools/compilers (SDKs, maven, gcc) from the final runtime image. This reduces image size from over 1 GB to under 50 MB.
2. **Run as Non-Root User:** Never run container processes as `root`. Create a dedicated system user (e.g., `appuser` with UID 10001).
3. **Optimize Layer Caching:** Copy dependency manifests (`package.json`, `requirements.txt`, `pom.xml`, `go.mod`) and run dependency installs before copying source code.
4. **Use Specific Base Image Tags:** Avoid `latest`. Use explicit, minimal base images (e.g., `alpine`, `distroless`, or `slim`).
5. **Use `.dockerignore`:** Exclude `node_modules`, `.git`, temporary files, and local logs from the build context.

---

## 1. Node.js Frontend (Web Service)

```dockerfile
# Stage 1: Build & Dependencies
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Final Minimal Runtime
FROM node:18-alpine
WORKDIR /app
# Security: Run as non-root user
USER node
COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --chown=node:node . .

EXPOSE 8080
ENV PORT=8080
CMD ["node", "server.js"]
```

---

## 2. Python Microservice (Cart / Payment)

```dockerfile
# Stage 1: Build Wheels
FROM python:3.10-slim AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends gcc python3-dev && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

# Stage 2: Minimal Runtime
FROM python:3.10-slim
WORKDIR /app
RUN addgroup --system appgroup && adduser --system --group appuser
COPY --from=builder /app/wheels /wheels
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache /wheels/*
COPY . .
USER appuser

EXPOSE 8080
CMD ["python", "app.py"]
```

---

## 3. Go Microservice (Catalogue / Dispatch)

```dockerfile
# Stage 1: Compile Binary
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o main .

# Stage 2: Distroless / Scratch Runtime (< 15 MB)
FROM gcr.io/distroless/static-debian11
WORKDIR /app
COPY --from=builder /app/main .
USER nonroot:nonroot

EXPOSE 8080
ENTRYPOINT ["/app/main"]
```
