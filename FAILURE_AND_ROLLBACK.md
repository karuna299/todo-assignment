# FAILURE_AND_ROLLBACK.md

## 1. A Faulty Version Is Deployed to Production — How Do You Roll Back?

**Rollback is performed using Git.**

### What Happens

* A faulty image version is detected after deployment.
* Kubernetes deployments are managed by **Argo CD**, not Jenkins.

### Rollback Steps

1. Identify the last stable Git commit in the GitOps repository.
2. Revert the commit that introduced the faulty change:

   ```bash
   git revert <bad-commit-sha>
   git push
   ```
3. Argo CD detects the Git change.
4. Argo CD automatically reconciles the cluster back to the previous stable state.

### Key Point

* **No manual kubectl rollback**
* **No cluster access required**
* Git history acts as the rollback mechanism

---

## 2. Application Crashes After Deployment — What Happens?

### What Happens Automatically

* Kubernetes **liveness probe fails**
* The container is marked unhealthy
* Kubernetes **restarts the pod automatically**

### If Crash Persists

* Pod enters `CrashLoopBackOff`
* Argo CD keeps the desired state intact
* No new deployments occur unless Git changes

### Operator Action

* Check logs:

  ```bash
  kubectl logs <pod-name>
  ```
* If issue is due to bad configuration or image:

  * Fix in Git
  * Commit and push
  * Argo CD redeploys corrected version

---

## 3. Jenkins Is Down — Can Deployment Still Occur?

**Yes, deployments can still occur.**

### Why

* Jenkins is used **only for CI**
* Jenkins does **not deploy to Kubernetes**
* Kubernetes deployments are handled by **Argo CD**

### Impact

* New images cannot be built or pushed while Jenkins is down
* Existing images and GitOps rollbacks **still work**

### Result

* Git-based rollback and redeployments remain functional
* Cluster stability is not affected by Jenkins downtime

---

## 4. Secrets Are Leaked — What Steps Do You Take?

### Immediate Actions

1. **Revoke compromised secrets**

   * Rotate DB credentials
   * Revoke API keys (Slack, Cohere)
2. Update secrets in secure storage (Kubernetes Secret / Jenkins Credentials).

### GitOps Action

* Secrets are **not hardcoded in Git**
* Secret manifests contain placeholders only
* No sensitive data exists in Git history

### Recovery

* Update Kubernetes Secret values
* Argo CD reconciles without redeploying code
* Application restarts with new credentials

---

## 5. Kubernetes Node Fails — How Does the System Recover?

### What Happens Automatically

* Kubernetes detects node failure
* Pods running on failed node are marked unavailable
* Scheduler reschedules pods to healthy nodes

### Why Recovery Works

* Deployments define desired replica count
* Kubernetes ensures replicas are maintained
* Argo CD continuously validates desired state

### Result

* No manual intervention required
* Application availability is restored automatically

---

## Summary Table

| Failure Scenario  | Recovery Mechanism           |
| ----------------- | ---------------------------- |
| Faulty Deployment | Git revert + Argo CD         |
| Application Crash | Liveness probe + Pod restart |
| Jenkins Down      | GitOps continues             |
| Secrets Leak      | Secret rotation              |
| Node Failure      | Kubernetes rescheduling      |

---

## Key Design Guarantees

* **Git is the single source of truth**
* **CI and CD are decoupled**
* **Rollbacks are deterministic and auditable**
* **Infrastructure failures are self-healing**
