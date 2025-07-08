## 🔹 1. **Pod** – Smallest Deployable Unit

### 📘 What is a Pod?

* A **Pod** is the basic unit in Kubernetes.
* It wraps **one or more containers** that:

  * Share the same **network namespace** (i.e., same IP)
  * Share **volumes** (shared storage)
  * Are **scheduled together** on the same node

### 🔧 Use Case:

* One container = main app
* Second container = sidecar (e.g., logging agent, reverse proxy)

### 🧪 YAML Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: app
      image: nginx
    - name: logger
      image: busybox
      command: ["sh", "-c", "while true; do echo log; sleep 5; done"]
```

---

## 🔹 2. **ReplicaSet** – Ensures Pod Availability

### 📘 What is a ReplicaSet?

* A **ReplicaSet** ensures a **specified number of pod replicas** are always running.
* Automatically **adds** or **removes** pods if needed.

### ⚠️ Note:

You usually don’t use ReplicaSet directly — it's managed by a Deployment.

### 🧪 YAML Example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
spec:
  replicas: 3
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
          image: nginx
```

---

## 🔹 3. **Deployment** – Declarative Rollouts & Updates

### 📘 What is a Deployment?

* A **Deployment** is the most common way to manage applications in Kubernetes.
* It wraps around ReplicaSets and Pods.
* Allows:

  * Declarative **updates**
  * **Rollbacks**
  * **Rollouts**
  * Scaling

### 🧪 YAML Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp-container
          image: nginx
```

### 🔁 Real Use Case:

* You want to update your app from `v1` to `v2`.
* Just change the image tag and apply — Kubernetes rolls it out with zero downtime.

---

## 🔹 4. **Service** – Networking for Pods

### 📘 What is a Service?

* A **Service** is a stable abstraction over ephemeral pods.
* Since pod IPs change frequently, services provide a **stable IP + DNS**.
* Can also load balance between pods.

### 🔄 Types of Services:

| Type             | Use Case                           |
| ---------------- | ---------------------------------- |
| **ClusterIP**    | Default, internal-only             |
| **NodePort**     | Exposes service on node’s IP\:Port |
| **LoadBalancer** | Uses cloud LB to expose publicly   |
| **ExternalName** | Maps service to external DNS name  |

### 🧪 YAML Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80       # Service port
      targetPort: 80 # Pod container port
      nodePort: 30080
```

---

## 🔹 5. **Namespace** – Logical Isolation

### 📘 What is a Namespace?

* Namespaces allow **logical separation** of resources.
* Useful for:

  * Multi-tenant clusters (dev/prod/test)
  * RBAC control
  * Resource quotas

### 🧪 Example Commands:

```bash
kubectl get pods --namespace=dev
kubectl create namespace dev
kubectl config set-context --current --namespace=dev
```

### 🧪 YAML Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

---

## 🎯 Real-World Flow:

Here’s how these resources work together:

1. **Namespace**: You create a namespace called `dev`
2. **Deployment**: You deploy an app using a Deployment (`replicas: 3`)
3. **ReplicaSet**: The Deployment automatically creates and manages a ReplicaSet
4. **Pod**: The ReplicaSet creates 3 pods
5. **Service**: You expose these pods using a Service (`NodePort` or `LoadBalancer`)

---

## ✅ Summary Table

| Resource   | Purpose                    | YAML Kind         | Key Fields                  |
| ---------- | -------------------------- | ----------------- | --------------------------- |
| Pod        | Run 1+ containers          | `v1/Pod`          | `containers`, `volumes`     |
| ReplicaSet | Maintain # of pods         | `apps/ReplicaSet` | `replicas`, `selector`      |
| Deployment | Declarative app management | `apps/Deployment` | `strategy`, `template`      |
| Service    | Expose pods to network     | `v1/Service`      | `type`, `ports`, `selector` |
| Namespace  | Logical grouping           | `v1/Namespace`    | `metadata.name`             |

---
