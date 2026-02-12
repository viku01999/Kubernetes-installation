# Kubernetes Node Server (Dockerized)

This project is a Node.js + TypeScript backend application packaged using Docker and published to Docker Hub.

---

## 📦 Prerequisites

Make sure you have the following installed:

- Docker
- Docker Hub account
- Node.js (only required for local development, not for Docker)

Verify Docker installation:

```bash
docker --version
```

## Project Structure

```pgsql
kubernetes-node-server
├── Dockerfile
├── package.json
├── package-lock.json
├── src/
├── tsconfig.json
├── deployment.yaml
├── service.yaml

```

## Dockerfile Used

```dockerfile
FROM node:20-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

## 🏗️ Build Docker Image Locally

```bash
# Run this command from the project root directory:
docker build -t kubernetes-node-server:2.0 .

# Check image:
docker images

```

## 🏷️ Tag Image for Docker Hub

```bash
# Docker Hub requires images to be tagged with your username.
docker tag kubernetes-node-server:2.0 viku01999/kubernetes-node-server:2.0

# Verify tag:
docker images

# You should see:
kubernetes-node-server:2.0
viku01999/kubernetes-node-server:2.0

```

## 🔐 Login to Docker Hub

```bash
docker login

# Enter your Docker Hub username and password.
```

## 🚀 Push Image to Docker Hub

```bash
# ⚠️ Always push the image with your username.
docker push viku01999/kubernetes-node-server:2.0

# If successful, the image will appear in your Docker Hub repository.

```

## 📥 Pull Image from Docker Hub

```bash
# On any machine with Docker installed:
docker pull viku01999/kubernetes-node-server:2.0
```

## ▶️ Run Docker Container

```bash
# List images
docker images

```

## 🧠 Common Docker Commands

```bash
# List images
docker images

# List running containers
docker ps

# Stop container
docker stop <container_id>

# Remove container
docker rm <container_id>

# Remove image
docker rmi <image_id>

```

## ⚠️ Common Errors & Fixes

❌ denied: requested access to the resource is denied

✔️ Cause: Pushing image without Docker Hub username
✔️ Fix: Always tag and push like this:

```bash
docker push viku01999/kubernetes-node-server:2.0
```

❌ tag does not exist

✔️ Cause: Image not tagged correctly
✔️ Fix:

```bash
docker tag local-image:tag username/repo:tag
```

## ✅ Versioning Strategy

1.0 → Initial build

2.0 → Updated Dockerfile / fixes

Always increment tag for new changes