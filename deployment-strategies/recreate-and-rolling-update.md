## 🚀 What is a **Deployment Strategy**?

A **deployment strategy** defines **how Kubernetes replaces old pods with new ones** when you make changes (e.g., image update).

---

## ✅ 1. **Rolling Update (Default Strategy)**

### 📘 What is it?

* Updates **pods incrementally**, without downtime.
* Ensures **high availability** during updates.
* Old pods are replaced **one at a time** with new ones.

### 🔧 Configuration:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1      # max old pods that can go down during update
    maxSurge: 1            # max extra pods that can be created during update
```

### 🧪 Example:

You update `nginx:1.21` → `nginx:1.23`. If you have 3 pods:

* Step 1: Start a new pod with `1.23` (total = 4 pods temporarily)
* Step 2: Delete 1 old pod (`1.21`) (back to 3 pods)
* Repeats until all old pods are replaced

### ✅ Pros:

* Zero downtime
* Safe for production workloads

### ❌ Cons:

* Slightly more complex to monitor
* Requires pod readiness probes for optimal results

---

## 🔄 2. **Recreate**

### 📘 What is it?

* **Deletes all old pods first**, then creates new pods
* **Downtime happens** during this switch

### 🔧 Configuration:

```yaml
strategy:
  type: Recreate
```

### 🧪 Example:

You update `myapp:v1` → `myapp:v2`:

* All `v1` pods are deleted
* Then `v2` pods are created

### ✅ Pros:

* Simple logic
* Useful if **only one pod can run at a time** (e.g., DB migrations)

### ❌ Cons:

* **Downtime** during deployment
* Not suitable for critical services

---

## 🧠 Comparison: RollingUpdate vs Recreate

| Feature          | RollingUpdate        | Recreate                   |
| ---------------- | -------------------- | -------------------------- |
| Downtime         | ❌ No                 | ✅ Yes                      |
| Pod availability | Always maintained    | Temporarily zero           |
| Use case         | Production, web apps | Single-instance apps, jobs |
| Default          | ✅ Yes                | ❌ No                       |

---

## 🧪 YAML with Both Strategies

### 🔹 Rolling Update Deployment YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.23
```

---

### 🔹 Recreate Strategy YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: single-pod-app
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: job-runner
  template:
    metadata:
      labels:
        app: job-runner
    spec:
      containers:
        - name: job
          image: myjob:latest
```

---

## 👀 Advanced Deployment Strategies (outside native K8s, but commonly used)

| Strategy        | Tool Used              | Description                                                              |
| --------------- | ---------------------- | ------------------------------------------------------------------------ |
| **Blue-Green**  | ArgoCD, Spinnaker      | Two environments (blue = old, green = new); switch traffic after testing |
| **Canary**      | Argo Rollouts, Flagger | Gradual rollout (e.g., 10%, 25%, 100%) to test stability                 |
| **A/B Testing** | Istio, Linkerd         | Route traffic based on user behavior or headers                          |

These strategies need **advanced traffic routing** (via service mesh or ingress).

---

## 🔚 Summary Table

| Strategy      | Downtime | Use Case                 | Native in K8s?   |
| ------------- | -------- | ------------------------ | ---------------- |
| RollingUpdate | ❌ No     | Web apps, APIs           | ✅ Yes            |
| Recreate      | ✅ Yes    | DB migration, batch jobs | ✅ Yes            |
| Blue-Green    | ❌ No     | Version switching        | ❌ External tools |
| Canary        | ❌ No     | Risk-controlled releases | ❌ External tools |

---

## ✅ Commands to Monitor Deployment Strategy

```bash
kubectl rollout status deployment myapp
kubectl get replicaset
kubectl get pods
kubectl describe deployment myapp
```

---
