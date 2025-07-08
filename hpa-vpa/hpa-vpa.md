# ⚖️ HPA vs. VPA — Overview

| Feature               | Horizontal Pod Autoscaler (HPA)                | Vertical Pod Autoscaler (VPA)           |
| --------------------- | ---------------------------------------------- | --------------------------------------- |
| **What it does**      | Scales **pod count**                           | Scales **CPU/Memory requests & limits** |
| **Scaling direction** | Horizontal (more pods)                         | Vertical (bigger pods)                  |
| **Use case**          | Web servers, APIs, scalable apps               | Batch jobs, ML, analytics               |
| **Risk**              | Too many pods can overwhelm infra              | Pod restart can interrupt workload      |
| **Can use together?** | Partially (use CPU for HPA and memory for VPA) | ❗Caution required                       |

---

## 🔄 Horizontal Pod Autoscaler (HPA)

### ✅ What is it?

HPA automatically scales the **number of replicas of a Deployment, StatefulSet, or ReplicaSet** based on metrics like:

* CPU utilization
* Memory (since v2 API)
* Custom/external metrics (Prometheus, custom API)

---

### 🧪 Example: Auto-scale a Deployment based on CPU

#### 🔧 Step 1: Enable Metrics Server (only once per cluster)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

#### 🧾 Step 2: Sample Deployment (`myapp`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: nginx
          resources:
            requests:
              cpu: 100m
            limits:
              cpu: 200m
```

---

#### 🧾 Step 3: Create the HPA

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

🧠 **Explanation:**

* If average CPU usage across all pods > 60% → add pods
* If usage drops → remove pods (but keep min 2)

---

### 🔍 Test It:

```bash
kubectl get hpa
kubectl describe hpa myapp-hpa
```

Simulate load with:

```bash
kubectl run -i --tty load-generator --image=busybox /bin/sh
# Inside container:
while true; do wget -q -O- http://<MYAPP_SERVICE>; done
```

---

## 📈 Vertical Pod Autoscaler (VPA)

### ✅ What is it?

VPA automatically adjusts the **resource requests and limits (CPU and memory)** of your pods **based on usage**.

> Useful for apps that can’t scale horizontally — like DBs, AI workloads, legacy apps, etc.

---

### 🧾 VPA Modes:

| Mode      | Behavior                                             |
| --------- | ---------------------------------------------------- |
| `Off`     | Only monitors usage                                  |
| `Initial` | Sets recommended values **at creation time only**    |
| `Auto`    | **Actively updates** pod specs (may trigger restart) |

---

### 🧪 Example: VPA with `Auto` Mode

#### 🔧 Step 1: Install VPA (only once per cluster)

```bash
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml
```

---

#### 🧾 Step 2: Create VPA Resource

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"
```

✅ Kubernetes will:

* Analyze real usage metrics
* Update the pod resource requests
* Restart pods if needed

---

### 🧠 Behind the scenes:

* VPA recommends:

  * `target`: Ideal request
  * `lowerBound` / `upperBound`: Safe limits
* You can use:

```bash
kubectl describe vpa myapp-vpa
```

---

### ⚠️ HPA + VPA Together?

| Metric | Who owns it? | Can be combined?              |
| ------ | ------------ | ----------------------------- |
| CPU    | HPA or VPA   | ❌ Avoid both using CPU        |
| Memory | VPA only     | ✅ Works if HPA is on CPU only |

✅ Best practice:

* Use **HPA on CPU**
* Use **VPA on Memory**
* Disable VPA updates to avoid conflict (`updateMode: Off`)

---

## 🧠 Real-World Interview Scenarios

**Q1:**

> Your app has varying traffic. How would you ensure it scales automatically?

✅ Use HPA with CPU or request-based scaling.

---

**Q2:**

> You have a batch job running for hours but memory spikes crash it. What to do?

✅ Apply VPA to auto-adjust memory requests & prevent OOMKill.

---

**Q3:**

> Your app cannot scale horizontally. Still want autoscaling?

✅ Use VPA to scale vertically (increase CPU/memory).

---

## ✅ Summary Table

| Feature           | HPA                     | VPA                             |
| ----------------- | ----------------------- | ------------------------------- |
| Scales            | Pod **count**           | CPU/Memory **resources**        |
| Triggered by      | Metrics (CPU, custom)   | Usage over time                 |
| Acts on           | Deployment, StatefulSet | Deployment, StatefulSet         |
| Triggers restart? | ❌ No                    | ✅ Yes (in Auto mode)            |
| Use case          | APIs, web servers       | Batch jobs, ML, single-pod apps |
| Conflicts         | With VPA (CPU)          | With HPA (CPU)                  |

---
