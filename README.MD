# Todo Summary Assistant – DevOps Pipeline

## Project Overview

This repository contains a production-ready **DevOps delivery pipeline** for the **Todo Summary Assistant** application.
The focus of this work is **DevOps automation, CI/CD, containerization, Kubernetes deployment, and GitOps**, not application complexity.

The application consists of:

* **Backend**: Spring Boot (Java, Maven)
* **Frontend**: React
* **Database**: MySQL

---

## Tech Stack & Tools

* **Git** – Source control
* **Maven** – Backend build tool
* **Docker** – Containerization (multi-stage, non-root)
* **Jenkins** – Continuous Integration
* **Kubernetes** – Container orchestration
* **GitOps (ArgoCD – design level)** – Git-driven deployments
* **MySQL** – Relational database

---

## Dependencies (Local Setup)

You need the following installed locally:

* Git
* Java 17
* Maven 3.9+
* Docker & Docker Compose
* Node.js 18+
* kubectl
* Minikube (for local Kubernetes testing)

---

## Local Run Instructions

### 1️⃣ Clone Repository

```bash
git clone https://github.com/<your-username>/todo-assignment.git
cd todo-assignment
```

---

### 2️⃣ Run MySQL (Docker)

```bash
docker run -d \
  --name mysql-todo \
  -e MYSQL_ROOT_PASSWORD=tiger \
  -e MYSQL_DATABASE=todo_db \
  -p 3306:3306 \
  mysql:8
```

---

### 3️⃣ Run Backend Locally

```bash
cd Backend/todo-summary-assistant
mvn spring-boot:run
```

Backend runs on:

```
http://localhost:8082
```

---

### 4️⃣ Run Frontend Locally

```bash
cd Frontend/todo
npm install
npm start
```

Frontend runs on:

```
http://localhost:3000
```

---

## Containerization (Docker)

### Docker Design Choices

* **Multi-stage builds** to reduce image size
* **Non-root users** for security hardening
* **Externalized configuration** via environment variables
* **Optimized base images** (`alpine`, `eclipse-temurin`)
* No secrets hardcoded in images

Both backend and frontend images are built and pushed via CI.

---

## Continuous Integration – Jenkins

### Pipeline Stages

1. **Checkout** – Pull source from Git
2. **Build** – Maven build for backend
3. **Test** – Run unit tests (if present)
4. **Docker Build** – Build backend & frontend images
5. **Tagging** – Tag images using Git commit hash
6. **Push** – Push images to Docker Hub
7. **Fail Fast** – Pipeline stops on any error

🔒 Secrets (Docker Hub credentials, API keys) are managed via **Jenkins Credentials**, not in code.

---

## Kubernetes Deployment

Kubernetes manifests are stored under:

```
k8s/
```

Includes:

* Deployments (frontend, backend, MySQL)
* Services
* ConfigMaps
* Secrets (dummy values only)
* Resource requests & limits
* Readiness & liveness probes

Kubernetes was tested locally using **Minikube**.

---

## GitOps Model (Design)

### How Deployments Are Triggered

* CI pipeline **does NOT deploy to Kubernetes**
* CI only builds & pushes container images
* Kubernetes state is defined declaratively in Git
* ArgoCD (conceptually) watches the Git repository
* Any Git change triggers synchronization

### Rollback Strategy (GitOps)

* Revert Git commit to last known good version
* ArgoCD automatically syncs previous state
* No manual cluster intervention required

---

## Assumptions Made

* The application business logic was not modified; only DevOps-related changes were made.
* The full application stack (frontend, backend, MySQL) was successfully run locally.
* Slack webhook and Cohere API were configured and tested during local execution.
* Secrets (DB credentials, API keys, webhook URLs) are externalized for CI/CD and Kubernetes and are not committed to    Git.
* Local ports used during development are configurable and may differ from production.
* Local Kubernetes (Minikube) was used to validate manifests; production clusters are environment-specific.
---

## Failure & Rollback

See:

```
FAILURE_AND_ROLLBACK.md
```

---

## Monitoring & Operations

See:

```
MONITORING_AND_OPERATIONS.md
```

---

## Conclusion

This project demonstrates a **production-grade DevOps workflow** including:

* Secure containerization
* CI automation
* Kubernetes deployment
* GitOps-based delivery model
* Clear rollback and operational strategies
