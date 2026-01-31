# GitOps Architecture and Implementation (This Project)

## 1. What GitOps Means in *This Project*

In this project, **GitOps means**:

* **GitHub is the single source of truth for Kubernetes state**
* **No component (Jenkins, human, script) directly deploys to Kubernetes**
* **Kubernetes deployments are driven only by Git commits**
* **Argo CD continuously reconciles Git state with the cluster**

This was implemented **locally using Minikube + Argo CD**.

---

## 2. Why GitOps Was Required (Task Alignment)

The task explicitly required:

* CI pipeline **must not deploy directly** to Kubernetes
* Kubernetes state **must be driven from Git**
* Explanation of:

  * How deployments are triggered
  * How rollback is performed using Git
  * GitOps directory structure
  * Written GitOps flow explanation

All of this was implemented and demonstrated.

---

## 3. GitOps Architecture (As Implemented)

### Architecture Flow

```
Developer
   |
   | git push (manifests / image tag change)
   v
GitHub Repository (GitOps directory)
   |
   | watched continuously
   v
Argo CD (inside Minikube)
   |
   | reconciliation loop
   v
Kubernetes Cluster (Minikube)
```

**Important architectural rule**
👉 Kubernetes never pulls state from Jenkins
👉 Jenkins never talks to Kubernetes
👉 Only Argo CD talks to Kubernetes

---

## 4. GitOps Directory Structure (Used in This Project)

```
gitops/
├── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── secret.yaml
│
├── frontend/
│   ├── deployment.yaml
│   └── service.yaml
│
├── mysql/
│   ├── deployment.yaml
│   └── service.yaml
│
└── README.md (optional explanation)
```

### Why This Structure Was Chosen

* Clear separation of concerns
* Each component is independently deployable
* Easy rollback by folder or file
* Matches real GitOps repository layout

This directory is what **Argo CD tracks**, not application source code.

---

## 5. Installing Argo CD in Minikube (Exact Steps Taken)

### Step 1: Start Minikube

```bash
minikube start
```

---

### Step 2: Create Argo CD Namespace

```bash
kubectl create namespace argocd
```

---

### Step 3: Install Argo CD

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This installs:

* Argo CD server
* Repo server
* Application controller
* API server

All inside the cluster.

---

### Step 4: Verify Installation

```bash
kubectl get pods -n argocd
```

---

### Step 5: Access Argo CD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access:

```
https://localhost:8080
```

---

### Step 6: Get Admin Password

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

---

## 6. Connecting GitHub Repository to Argo CD

Argo CD was configured to:

* Watch the GitHub repository
* Track the `gitops/` directory
* Sync changes automatically

This establishes **Git → Cluster control**.

---

## 7. How Deployments Are Triggered (Very Important)

### Deployment Trigger Flow

1. Developer updates Kubernetes manifest
   (example: image tag change)
2. Developer runs:

   ```bash
   git commit
   git push
   ```
3. Argo CD detects Git change
4. Argo CD applies the change to Kubernetes
5. Cluster state updates automatically

### Key Point

❌ No `kubectl apply` in production flow
❌ No Jenkins → Kubernetes connection

✅ Git commit = deployment trigger

---

## 8. How Rollback Is Performed Using Git

### Rollback Flow

```
Problem detected
   ↓
git revert <commit>
   ↓
git push
   ↓
Argo CD detects rollback
   ↓
Cluster restored automatically
```

### Why This Works

* Kubernetes desired state = Git state
* Old commit = old desired state
* Argo CD reconciles back to it

### Result

* No manual rollback
* No cluster access needed
* Fully auditable rollback

---

## 9. Secret Handling Design (As Implemented)

### What Was Done

* Secrets were **not hardcoded**
* Secret manifests contain **placeholders**
* Real values are injected externally (local testing only)

### Why This Is GitOps-Correct

* Git history stays clean
* No credential leakage
* GitOps flow remains intact

---

## 10. Why Jenkins Is Not Part of GitOps

In this project:

* Jenkins = **CI only**
* Argo CD = **CD only**

Jenkins:

* Builds images
* Pushes to Docker Hub

Argo CD:

* Deploys images
* Manages cluster state

This strict separation was **intentional and required**.

---

## 11. GitOps Characteristics Achieved

✔ Declarative deployments
✔ Git as single source of truth
✔ Automatic reconciliation
✔ Git-based rollback
✔ No manual cluster changes

---

## 12. Summary (What Was Actually Done)

* Installed Argo CD in Minikube
* Created GitOps directory with manifests
* Connected GitHub repo to Argo CD
* Enabled auto-sync
* Deployed backend, frontend, MySQL via Git
* Demonstrated Git-based rollback
* Kept CI and CD strictly separated

---