## ✅ Remaining / Advanced K8s Topics

### 🔒 1. **Security (RBAC & Policies)**

| Concept                                      | Why it matters                                                            |
| -------------------------------------------- | ------------------------------------------------------------------------- |
| RBAC (Role-Based Access Control)             | Controls who can do what                                                  |
| ServiceAccounts                              | Auth for pods                                                             |
| PodSecurityPolicies / Pod Security Standards | Controls what pods are allowed to run (deprecated → PSA/PSP alternatives) |
| Network Policies                             | Controls pod-to-pod traffic                                               |
| Secrets Management (vault integration)       | Secure handling of sensitive data                                         |

✅ *Important for DevSecOps & interviews where security matters.*

---

### 🛡️ 2. **Pod Lifecycle, Probes, and Health**

| Concept         | Purpose                                           |
| --------------- | ------------------------------------------------- |
| Readiness Probe | Controls **when pod receives traffic**            |
| Liveness Probe  | Detects **when pod is unhealthy** and restarts it |
| Startup Probe   | Used for **slow-start apps**                      |
| Lifecycle Hooks | Run pre-stop/start actions                        |

✅ *Crucial for production stability.*

---

### 🔄 3. **Jobs & CronJobs**

| Concept | Purpose                                                      |
| ------- | ------------------------------------------------------------ |
| Job     | Run a pod **until completion** (e.g., batch task, migration) |
| CronJob | Scheduled Jobs (e.g., backups, reports)                      |

✅ *Essential for background or scheduled DevOps tasks.*

---

### 📦 4. **Kustomize**

| Concept          | Purpose                                                                                   |
| ---------------- | ----------------------------------------------------------------------------------------- |
| Kustomize        | Native Kubernetes way to manage **YAML overlays** (e.g., dev, staging, prod environments) |
| Bases & overlays | Inherit from base YAML and override what’s needed                                         |
| Built-in support | Used with `kubectl apply -k`                                                              |

✅ *Less popular than Helm but worth knowing, especially for CI/CD pipelines.*

---

### ☁️ 5. **Cluster-Level Management**

| Concept                           | Why it matters                                       |
| --------------------------------- | ---------------------------------------------------- |
| Namespaces                        | Isolation within cluster                             |
| ResourceQuotas & LimitRanges      | Prevent abuse by apps                                |
| Node Affinity & Taints            | Schedule pods on right nodes                         |
| Custom Resource Definitions (CRD) | Extend K8s with your own types (e.g., ArgoCD, Istio) |

✅ *For managing larger teams or multi-tenant clusters.*

---

### 🔍 6. **Monitoring, Logging & Observability**

| Tool                                  | Use                               |
| ------------------------------------- | --------------------------------- |
| Prometheus + Grafana                  | Metrics collection and dashboards |
| Loki / EFK (Elastic, Fluentd, Kibana) | Logs                              |
| OpenTelemetry                         | Tracing                           |
| Metrics Server                        | Required for HPA                  |

✅ *You should know how to install these via Helm or YAML.*

---

### 🧪 7. **Debugging & Troubleshooting**

| Skill               | Command                                                       |
| ------------------- | ------------------------------------------------------------- |
| Get pod events      | `kubectl describe pod`                                        |
| Logs                | `kubectl logs`                                                |
| Exec into container | `kubectl exec -it`                                            |
| Check DNS           | `nslookup`, `dig` inside pod                                  |
| Network debug       | `kubectl port-forward`, `tcpdump`, `iptables` (in node shell) |

✅ *Interviewers often ask “how would you debug \_\_\_ issue?”*

---

### 🧠 8. **GitOps & Tools Integration**

| Tool          | Use                                               |
| ------------- | ------------------------------------------------- |
| ArgoCD / Flux | GitOps: auto-deploy YAMLs from Git                |
| K9s           | Terminal UI to browse cluster                     |
| Lens          | GUI for managing clusters                         |
| Skaffold      | CI/CD for local K8s development                   |
| Telepresence  | Debug services locally while connected to cluster |

✅ *Nice to know if you're asked about ecosystem or tools.*

---

## 🧾 Final K8s Checklist for You

✅ Pods, ReplicaSets, Deployments
✅ StatefulSet, DaemonSet
✅ Services, Ingress, Network Policies
✅ ConfigMap, Secrets, Env Vars
✅ Volumes, PV, PVC, StorageClass
✅ HPA, VPA, Cluster Autoscaler
✅ Helm, Helm Charts
✅ Namespaces
✅ Deployment strategies (Rolling, Recreate, Canary, A/B)
✅ Job & CronJob
✅ Readiness/Liveness Probes
✅ RBAC, ServiceAccount
✅ Debugging (`logs`, `exec`, `describe`)
✅ Monitoring & Observability tools
✅ GitOps intro (ArgoCD, Flux)

---

## 💥 Bonus for Interview Edge

* Create your own K8s cluster on **kind** or **minikube**
* Practice CI/CD with **GitHub Actions + Helm**
* Try deploying a full stack (frontend + backend + DB) using:

  * StatefulSet (DB)
  * Deployment (app)
  * Service + Ingress
  * ConfigMap + Secret

---
