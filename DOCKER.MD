# Docker Design Choices

## Overview

This project uses Docker to containerize the **frontend** and **backend** services independently.
Each service has its own `Dockerfile` and `.dockerignore` to ensure isolated, reproducible, and efficient builds.

---

## Why Docker Was Used

* To ensure **environment consistency** across local, CI, and Kubernetes environments
* To package application code along with its runtime dependencies
* To simplify CI/CD integration (Jenkins builds and pushes Docker images)
* To make the application deployable on Kubernetes

---

## Multi-stage Docker Builds (Why We Used Them)

Both frontend and backend Dockerfiles use **multi-stage builds**.

### Reasons:

* **Smaller final image size**
* **Improved security** (build tools are not included in runtime image)
* **Faster deployments and pulls**
* Clear separation between **build-time** and **runtime** concerns

### Example Flow:

1. **Build stage**

   * Uses Node/Maven image
   * Installs dependencies
   * Compiles/builds the application
2. **Runtime stage**

   * Uses a lightweight base image
   * Copies only the compiled output
   * Runs the application

---

## Backend Dockerfile Design (Spring Boot)

### Key Decisions:

* **Java 17 base image** to match the project’s runtime requirements
* Application packaged as a **fat JAR**
* Non-root user used for better container security
* Exposed port matches the application server port (`8080`)

### Benefits:

* Minimal runtime image
* Secure execution (non-root)
* Compatible with Kubernetes health probes

---

## Frontend Dockerfile Design

### Key Decisions:

* Node image used only in the build stage
* Static build output served directly (no development server)
* Lightweight runtime image for serving frontend assets
* Port exposed matches frontend service expectations

### Benefits:

* Fast startup
* Reduced attack surface
* Clear separation of build and runtime logic

---

## Why `.dockerignore` Was Used

Each service includes a `.dockerignore` file.

### Reasons:

* Prevents unnecessary files from being sent to the Docker daemon
* Reduces image build time
* Avoids leaking sensitive or irrelevant files into images

### Commonly Ignored Items:

* `node_modules`
* `.git`
* Build caches
* Logs
* IDE/editor configuration files
* Local environment files

---

## Why Docker Images Are Built Separately

* Frontend and backend scale independently
* Failures in one service do not affect the other
* Easier CI/CD control
* Aligns with microservice-style deployment on Kubernetes

---

## CI/CD Compatibility

The Docker images are:

* Built during the Jenkins pipeline
* Tagged appropriately
* Pushed to Docker Hub
* Pulled by Kubernetes during deployment

This design ensures the same images are used across:

* Local testing
* CI pipelines
* Kubernetes deployments

---

## Summary

The Docker setup was designed to be:

* **Minimal**
* **Secure**
* **CI/CD friendly**
* **Kubernetes ready**
