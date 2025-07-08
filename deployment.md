## 🚀 What is a **Deployment** in Kubernetes?

> A **Deployment** is a Kubernetes controller that **manages ReplicaSets and Pods declaratively**.

It ensures that:

* The **desired number** of replicas are running.
* You can **roll out updates**, **roll back**, and **scale** easily.
* It handles **zero-downtime rolling updates** by default.

📌 It wraps around a ReplicaSet, which in turn manages Pods.

---

## 📦 Key Features of Deployment

| Feature              | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| **Declarative**      | You declare the desired state (e.g. 3 replicas of version X) |
| **Self-healing**     | Replaces failed pods                                         |
| **Rolling Updates**  | Update app version with **zero downtime**                    |
| **Rollbacks**        | Revert to previous revision if something breaks              |
| **Scaling**          | Increase/decrease replica count                              |
| **Revision History** | Tracks versions of your Deployment                           |

---

## 🧪 Full Deployment YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
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
          image: nginx:1.21
          ports:
            - containerPort: 80
```

---

## 🔍 In-Depth Explanation of Each Field

### 🔹 `apiVersion: apps/v1`

* Required for Deployments (use `apps/v1`, not `v1`)

### 🔹 `kind: Deployment`

* Tells Kubernetes this is a Deployment object.

### 🔹 `metadata`

* Name, labels for identifying this resource.

### 🔹 `spec.replicas`

* Number of desired pod replicas (3 means 3 identical pods).

### 🔹 `spec.selector.matchLabels`

* Matches pods that this Deployment controls. **Must match `template.metadata.labels`**.

> ❗ If this doesn't match, the Deployment won't manage the pods.

### 🔹 `spec.template`

* Describes the Pod template to use when creating replicas.

---

## 🔄 Deployment Workflow Internally

1. You `kubectl apply -f deployment.yaml`
2. Deployment is created
3. It creates a **ReplicaSet**
4. The ReplicaSet creates **N pods**
5. If pods crash or get deleted, the RS (via Deployment) re-creates them

---

## 🔄 Deployment Strategies (Updates)

### ✅ Default: **RollingUpdate**

* Replaces pods **one at a time**
* Ensures **zero downtime**
* Controlled using:

  ```yaml
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1       # one extra pod allowed temporarily
      maxUnavailable: 1 # one pod can go down during update
  ```

### 🔁 **Recreate**

* Deletes all old pods, then starts new pods (causes downtime)

  ```yaml
  strategy:
    type: Recreate
  ```

---

## 🔙 Rollbacks

You can roll back to a previous version easily.

### 🔧 Commands:

```bash
kubectl rollout undo deployment myapp-deployment
kubectl rollout history deployment myapp-deployment
kubectl rollout status deployment myapp-deployment
```

📌 *Great interview Q: What happens if a Deployment rollout fails?*

---

## ⚖️ Scaling

You can change replica count to scale up/down.

```bash
kubectl scale deployment myapp-deployment --replicas=5
```

Or in YAML:

```yaml
spec:
  replicas: 5
```

---

## 🧪 Example: Update App Version

You want to update from `nginx:1.21` → `nginx:1.23`.

Just change this line:

```yaml
image: nginx:1.23
```

And apply again:

```bash
kubectl apply -f deployment.yaml
```

The **rolling update** will begin, replacing pods one by one.

---

## 📌 Deployment Lifecycle Phases

| Phase       | Meaning                             |
| ----------- | ----------------------------------- |
| Pending     | RS is being created                 |
| Progressing | New pods being created              |
| Complete    | New RS has desired replicas running |
| Failed      | Timeout or probe failure            |

---

## 🧠 Common Interview Questions

| Question                                                 | Answer                                                             |
| -------------------------------------------------------- | ------------------------------------------------------------------ |
| What’s the difference between Deployment and ReplicaSet? | Deployment manages ReplicaSets declaratively; RS only manages pods |
| Can I use multiple containers in a Deployment?           | Yes, inside the pod template                                       |
| How does rolling update work?                            | Replaces pods one-by-one with maxSurge and maxUnavailable          |
| What happens if I kill a pod in a Deployment?            | It’s recreated automatically                                       |
| Can I rollback?                                          | Yes, using `kubectl rollout undo`                                  |

---

## 📝 Real-World Deployment Use Case

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-api
spec:
  replicas: 4
  selector:
    matchLabels:
      app: node-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: node-api
    spec:
      containers:
        - name: api-container
          image: yatricloud/node-api:v2
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
```

---

## ✅ Summary Table

| Field              | Purpose                               |
| ------------------ | ------------------------------------- |
| `replicas`         | Desired number of pods                |
| `template`         | Pod spec (container, volumes, probes) |
| `strategy`         | Update method (rolling/recreate)      |
| `selector`         | Pod matching logic                    |
| `revision history` | Auto rollback and track changes       |

---

## 🔚 Final Notes for Interview

* Deployments are used for **stateless** apps (web apps, APIs).
* For **stateful apps** like databases, use `StatefulSet`.
* Learn `kubectl rollout`, `kubectl describe`, and scaling commands.
