# Kubernetes

## 📘 **Kubernetes Concepts Roadmap**

(Organized from foundational to advanced with real-world use cases)

---

### 🔹 **1. Kubernetes Basics**

* What is Kubernetes and Why Use It?
* Kubernetes Architecture:

  * Master Node / Control Plane
  * Worker Nodes
  * kube-apiserver, etcd, scheduler, controller-manager
* kubectl CLI & basic commands

---

### 🔹 **2. Core Objects (Resources)**

| Resource       | Purpose                                         |
| -------------- | ----------------------------------------------- |
| **Pod**        | Smallest deployable unit (1 or more containers) |
| **ReplicaSet** | Ensures a specific number of pod replicas       |
| **Deployment** | Declarative updates to Pods and ReplicaSets     |
| **Service**    | Exposes a pod to internal/external network      |
| **Namespace**  | Virtual clusters within a cluster (multi-team)  |

---

### 🔹 **3. Configuration Management**

* **ConfigMap** – Inject config data into pods
* **Secret** – Securely inject passwords, keys, tokens
* **Environment variables** in pods
* **Downward API**

---

### 🔹 **4. Networking in Kubernetes**

| Concept          | Description                         |
| ---------------- | ----------------------------------- |
| ClusterIP        | Default internal service type       |
| NodePort         | Exposes service on node IPs         |
| LoadBalancer     | External access via cloud LB        |
| Ingress          | Layer 7 routing via URL or hostname |
| Network Policies | Control traffic between pods        |

---

### 🔹 **5. Storage**

| Resource                    | Description                              |
| --------------------------- | ---------------------------------------- |
| Volume                      | Temporary storage inside pod             |
| PersistentVolume (PV)       | Cluster-level storage resource           |
| PersistentVolumeClaim (PVC) | Pod's request for storage                |
| StorageClass                | Abstracts storage types (SSD, NFS, etc.) |

---

### 🔹 **6. Scheduling & Scaling**

* Labels & Selectors
* Node Affinity / Taints & Tolerations
* Horizontal Pod Autoscaler (HPA)
* Vertical Pod Autoscaler (VPA)
* Cluster Autoscaler (with cloud setup)

---

### 🔹 **7. Helm (K8s Package Manager)**

* Helm Charts – Reusable K8s templates
* `values.yaml` overrides
* Deploying apps using Helm
* Chart repositories

---

### 🔹 **8. Logging, Monitoring, & Observability**

* Liveness & Readiness Probes
* Logs via `kubectl logs`
* Prometheus + Grafana
* EFK Stack (Elasticsearch, Fluentd, Kibana)
* Jaeger (Tracing)

---

### 🔹 **9. Security**

| Feature                                  | Description               |
| ---------------------------------------- | ------------------------- |
| RBAC                                     | Role-based access control |
| Service Accounts                         | Identity inside cluster   |
| Network Policies                         | Pod-level firewall        |
| Pod Security Standards (PSPs deprecated) |                           |
| Secrets management (Vault integration)   |                           |

---

### 🔹 **10. CI/CD with Kubernetes**

* GitOps (ArgoCD, Flux)
* Image Pull Policies
* Canary Deployments
* Blue-Green Deployments
* Rolling updates and rollbacks

---

### 🔹 **11. Advanced Concepts**

* Custom Resource Definitions (CRDs)
* Operators (K8s automation)
* StatefulSets (Databases, etc.)
* DaemonSets (logging/monitoring agents)
* Jobs & CronJobs
* API Gateway (e.g., Istio, Ambassador)
* Service Mesh (Istio, Linkerd)

---

### 🔹 **12. Cloud-Specific K8s**

* GKE (Google Kubernetes Engine)
* EKS (AWS)
* AKS (Azure)
* Integrating with cloud storage/load balancers

---

### 🔹 **13. Real-World Deployment & Troubleshooting**

* Debugging pods (`kubectl exec`, `describe`, `logs`)
* CrashLoopBackOff, ImagePullBackOff
* Resource limits and OOM errors
* DNS issues inside pods
* Health check failures

---

### 🔹 **14. YAML Writing Mastery**

* Declarative syntax for:

  * Deployment
  * Service
  * Ingress
  * HPA
  * Volume
  * ConfigMap
  * Helm overrides

---
