## 🔄 6. Scheduling & Scaling in Kubernetes

| Topic                | Purpose                                                        |
| -------------------- | -------------------------------------------------------------- |
| Labels & Selectors   | Define matching between resources                              |
| Node Affinity        | Control which nodes pods can run on                            |
| Taints & Tolerations | Keep pods away from specific nodes unless explicitly tolerated |
| HPA                  | Automatically scale pods horizontally (more replicas)          |
| VPA                  | Automatically adjust pod resource requests/limits              |
| Cluster Autoscaler   | Add/remove nodes dynamically in a cloud environment            |

---

## 🔹 1. **Labels & Selectors**

### ✅ What is it?

Labels are **key/value pairs** used to **group and identify** Kubernetes resources (pods, services, deployments, etc.).

Selectors are used to **filter** and **select** resources based on labels.

---

### 🧪 Example:

```yaml
metadata:
  labels:
    app: myapp
    env: prod
```

To select pods:

```yaml
selector:
  matchLabels:
    app: myapp
```

✅ Services, ReplicaSets, and NetworkPolicies use **label selectors** to match pods.

---

## 🔹 2. **Node Affinity**

### ✅ What is it?

Node Affinity lets you **schedule pods on specific nodes** based on node labels.

> It’s a replacement of `nodeSelector` with more power and flexibility.

### 🔸 Types:

* **RequiredDuringSchedulingIgnoredDuringExecution** (hard rule)
* **PreferredDuringSchedulingIgnoredDuringExecution** (soft preference)

---

### 🧪 Example:

Node labeled as:

```bash
kubectl label node node1 disktype=ssd
```

Pod spec with node affinity:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

✅ Pod will only be scheduled on nodes with `disktype=ssd`.

---

## 🔹 3. **Taints & Tolerations**

### ✅ What is it?

Taints **repel pods from nodes**, unless pods have **tolerations** to "tolerate" the taint.

Use case: run only critical workloads on GPU or memory-optimized nodes.

---

### 🔧 Add a taint:

```bash
kubectl taint nodes node1 dedicated=ml:NoSchedule
```

This says: **don’t schedule pods on `node1` unless they tolerate this taint**.

---

### ✅ Pod toleration:

```yaml
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "ml"
    effect: "NoSchedule"
```

✅ Now this pod can be scheduled on that tainted node.

---

### 🧠 Taint Effects:

| Effect             | Meaning                              |
| ------------------ | ------------------------------------ |
| `NoSchedule`       | Prevent scheduling unless tolerated  |
| `PreferNoSchedule` | Avoid scheduling if possible         |
| `NoExecute`        | Evict existing pods unless tolerated |

---

## 🔹 4. **Horizontal Pod Autoscaler (HPA)**

### ✅ What is it?

> Automatically increases or decreases the **number of pod replicas** based on CPU, memory, or custom metrics.

Use it to scale **apps**, **API servers**, **workers**, etc.

---

### 🧪 Enable Metrics Server:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

### 🧪 Example YAML:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

✅ If CPU usage exceeds 60%, new pods will be created.

---

## 🔹 5. **Vertical Pod Autoscaler (VPA)**

> Automatically adjusts **CPU and memory** **requests/limits** for pods.

Useful for:

* Batch jobs
* Long-running analytics
* ML workloads

---

### 🧪 Example VPA:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       myapp
  updatePolicy:
    updateMode: "Auto"
```

✅ VPA will update pod resource specs, causing restarts as needed.

---

🧨 **VPA is not recommended with HPA** for the same metric (CPU), but can be used for **memory + HPA (CPU)** together.

---

## 🔹 6. **Cluster Autoscaler**

> Automatically scales the **number of nodes** in a cluster based on pending pods.

✅ Common in **cloud environments** like:

* AWS (EKS)
* GCP (GKE)
* Azure (AKS)

---

### 🔧 Example Setup (GKE):

```bash
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5 \
  --node-pool default-pool
```

When new pods **can’t be scheduled**, the autoscaler adds a new node.

When nodes become **idle**, they are automatically removed.

---

## 🔍 Real Interview Scenarios

**Q1:**

> How would you ensure DB pods only run on nodes with SSD?

✅ Use **node affinity** with node label `disktype=ssd`.

---

**Q2:**

> You want only team-critical pods to run on high-cost GPU nodes. How?

✅ Taint the node, add **toleration** to the critical pods only.

---

**Q3:**

> Your app has spikes in traffic every morning. How to scale?

✅ Use **HPA** with CPU or custom Prometheus metrics.

---

**Q4:**

> How to reduce cloud bill for unused nodes?

✅ Enable **cluster autoscaler** to downscale idle nodes.

---

## ✅ Summary Table

| Feature              | Purpose                              | Type                   |
| -------------------- | ------------------------------------ | ---------------------- |
| Labels & Selectors   | Group resources                      | Scheduling             |
| Node Affinity        | Place pods on specific nodes         | Scheduling             |
| Taints & Tolerations | Keep pods away from nodes            | Scheduling             |
| HPA                  | Scale pod **count** based on metrics | Runtime scaling        |
| VPA                  | Adjust pod **resources** dynamically | Runtime scaling        |
| Cluster Autoscaler   | Add/remove **nodes** automatically   | Infrastructure scaling |

---
