# MONITORING_AND_OPERATIONS.md

## 1. Metrics to Monitor (and **Why They Matter**)

### Application Metrics

* **HTTP request rate**
  Shows how much traffic the application is handling. Sudden drops may indicate outages; sudden spikes may indicate load issues.

* **HTTP error rate (4xx / 5xx)**
  A rise in errors directly indicates user-facing failures or backend bugs.

* **Request latency**
  High latency usually points to performance bottlenecks such as slow database queries or external API delays.

* **Application uptime**
  Confirms whether the service is continuously available.

* **JVM memory usage**
  Helps detect memory leaks or insufficient heap allocation before the application crashes.

---

### Kubernetes Metrics

* **Pod CPU usage**
  High CPU usage can cause throttling and slow responses.

* **Pod memory usage**
  Memory exhaustion leads to pod termination (OOMKilled).

* **Pod restart count**
  Repeated restarts indicate unstable applications or configuration errors.

* **Pod readiness status**
  Determines whether traffic is being routed to healthy pods.

* **Node CPU and memory usage**
  Prevents node-level resource exhaustion that can impact multiple services.

---

### Database Metrics

* **Database availability**
  Backend functionality depends directly on database uptime.

* **Active connection count**
  Connection pool exhaustion can bring down the application.

* **Query response time**
  Slow queries degrade overall application performance.

---

## 2. Critical Logs (and **Why They Are Critical**)

### Application Logs

* **Startup logs**
  Confirm successful application initialization and configuration loading.

* **Exception and stack traces**
  Essential for root-cause analysis when errors occur.

* **Database connection errors**
  Directly indicate configuration issues or DB outages.

* **External API failures (Slack / Cohere)**
  Show failures in third-party integrations that affect functionality.

---

### Kubernetes Logs

* **Pod logs**
  Primary source for diagnosing runtime failures inside containers.

* **Container restart reasons**
  Explain why pods are restarting (crash, OOM, probe failure).

* **Image pull failures**
  Indicate CI/CD or registry issues.

* **Readiness / liveness probe failures**
  Explain why traffic is blocked or pods are restarted.

---

### CI Logs (Jenkins)

* **Build logs**
  Identify compilation or dependency issues.

* **Test execution logs**
  Show why builds fail quality gates.

* **Docker build and push logs**
  Confirm image creation and registry publishing.

---

## 3. Alerts That Matter (and **Why They Matter**)

### Alerts That SHOULD Trigger

* **Pod in `CrashLoopBackOff`**
  Indicates a consistently failing application that requires immediate action.

* **Application failing readiness probes**
  Traffic is blocked; users are impacted.

* **Repeated pod restarts**
  Signals unstable code or misconfiguration.

* **Database unreachable**
  Application cannot function without DB access.

* **Kubernetes node not ready**
  Can affect multiple services running on the node.

* **Argo CD application `OutOfSync`**
  Indicates drift between Git and cluster state, violating GitOps principles.

---

### Alerts That Should NOT Trigger (Noise Reduction)

* **Single pod restart**
  Can happen during normal operations or rollouts.

* **Temporary CPU or memory spikes**
  Short spikes do not necessarily indicate failure.

* **Normal deployment rollouts**
  Expected behavior during releases.

* **Manually stopped Jenkins jobs**
  Do not indicate system failure.

*Reason:* Too many alerts cause alert fatigue and reduce response quality.

---

## 4. How Operational Issues Are Detected Early

* **Readiness probes**
  Ensure only healthy pods receive traffic, preventing user impact.

* **Liveness probes**
  Automatically recover stuck or crashed applications without manual intervention.

* **Pod restart metrics**
  Reveal instability trends before complete failure.

* **Argo CD sync status**
  Immediately detects configuration drift or failed deployments.

* **CI pipeline feedback**
  Prevents broken builds from ever reaching deployment stages.

---

## Summary

* **Metrics** provide early warning signals before failures occur.
* **Logs** enable fast root-cause analysis when issues happen.
* **Alerts** focus attention only on actionable problems.
* **Early detection mechanisms** reduce downtime and manual intervention.
