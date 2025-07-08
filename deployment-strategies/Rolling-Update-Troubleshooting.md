## 🧪 What is **A/B Testing Deployment**?

> **A/B Testing** is an advanced deployment strategy where **two or more versions** (e.g. A = v1, B = v2) of an application run simultaneously and **traffic is routed based on specific user criteria** (like headers, cookies, geography, etc.) instead of percentage.

This allows **real-time experimentation** and **measurement of performance, user behavior, or conversion rates** before rolling out the best version.

---

## 🎯 Purpose of A/B Testing in Deployment

| Objective                          | Example                                 |
| ---------------------------------- | --------------------------------------- |
| Test new UI with 10% users         | Route users with cookie `group=B` to v2 |
| Validate API performance           | Send traffic from certain region to B   |
| Experiment with ML models          | Group A = model v1, Group B = model v2  |
| Reduce risk for user-specific bugs | Target by device/browser or location    |

---

## 🔁 A/B vs Canary vs Blue-Green

| Strategy        | Traffic Control     | Purpose                     | Routing Method                |
| --------------- | ------------------- | --------------------------- | ----------------------------- |
| **Blue-Green**  | Switch 100% at once | Version replacement         | Manual switch                 |
| **Canary**      | Gradual % rollout   | Safe progressive deploy     | Weight-based                  |
| **A/B Testing** | Criteria-based      | Behavior/UX/metrics testing | Headers, cookies, user groups |

---

## 🧱 Core Concepts in A/B Testing

| Concept                   | Description                                                        |
| ------------------------- | ------------------------------------------------------------------ |
| **Label-based pods**      | Two Deployments with different labels                              |
| **Smart traffic routing** | Done via Ingress, Istio, or Service Mesh                           |
| **Targeted users**        | Based on headers, IP, cookies, or user ID                          |
| **Monitoring**            | Compare metrics: latency, error rate, conversion, behavior         |
| **Multiple versions**     | Can test A, B, C (not just two) — also called multivariate testing |

---

## 🛠️ Kubernetes A/B Testing Architecture

### 🔧 Tools You Can Use:

* **Ingress Controller** (NGINX, Traefik)
* **Service Mesh**: Istio, Linkerd
* **Flagger** + **Prometheus**
* **Envoy Proxy**
* **Istio VirtualService** for HTTP header or cookie-based routing

---

## 🧪 A/B Testing using Istio (HTTP Header Example)

### 1️⃣ Deploy Two Versions

#### v1 Deployment (A)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
        - name: myapp
          image: yatricloud/myapp:v1
```

#### v2 Deployment (B)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
        - name: myapp
          image: yatricloud/myapp:v2
```

---

### 2️⃣ Define Istio VirtualService for Routing

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
    - myapp.example.com
  http:
    - match:
        - headers:
            end-user:
              exact: test-user
      route:
        - destination:
            host: myapp
            subset: v2
    - route:
        - destination:
            host: myapp
            subset: v1
```

👆 This means:

* If the HTTP header `end-user: test-user` is found → send to v2
* Everyone else → goes to v1

---

### 3️⃣ DestinationRule (Required by Istio)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

---

## 📈 Metrics to Track During A/B Test

| Metric              | Purpose                        |
| ------------------- | ------------------------------ |
| **Latency**         | Is version B slower?           |
| **Conversion rate** | Are users clicking more on v2? |
| **Error rate**      | Are there 5xx or 4xx spikes?   |
| **Bounce rate**     | Are users dropping off?        |
| **CPU/Memory**      | Is v2 resource-efficient?      |

Tools: **Prometheus + Grafana**, **Datadog**, **Google Analytics**, **New Relic**

---

## 💬 A/B Testing via NGINX Ingress (Simpler Setup)

### ConfigMap example for cookie-based routing:

```nginx
map $cookie_user_group $upstream {
    default         v1;
    group-b         v2;
}
```

Then in your ingress:

```yaml
backend:
  serviceName: myapp-$upstream
```

---

## 🔙 Rollback Plan

Rollback is simple:

* Update VirtualService or Ingress to **route all traffic back to v1**
* Or delete/scale down v2 pods

You don't stop both versions — you just change the **traffic rules**.

---

## ✅ Pros and ❌ Cons of A/B Testing

| Pros                            | Cons                                               |
| ------------------------------- | -------------------------------------------------- |
| ✅ Real user behavior insights   | ❌ More complex traffic routing setup               |
| ✅ Safe targeted testing         | ❌ Needs good monitoring + segmentation             |
| ✅ Zero downtime                 | ❌ Needs header/cookie manipulation                 |
| ✅ Multivariate testing possible | ❌ Not native in plain K8s (requires Istio/Ingress) |

---

## 🧠 Interview Questions (A/B Testing)

| Question                               | Answer                                                      |
| -------------------------------------- | ----------------------------------------------------------- |
| What is A/B testing deployment?        | Deploy multiple versions and route based on user attributes |
| How is it different from Canary?       | Canary = gradual % rollout, A/B = criteria-based routing    |
| How do you implement it in Kubernetes? | Via Ingress rules, Istio VirtualService, or Service Mesh    |
| What do you monitor?                   | Conversion, latency, error rate, behavior metrics           |
| Can you do A/B without downtime?       | Yes, both versions run in parallel                          |

---

## 🧪 Example: Feature Testing with Headers

```bash
curl -H "end-user: test-user" https://myapp.example.com
# → routes to v2 (test version)

curl https://myapp.example.com
# → routes to v1 (stable version)
```

---

## 🎓 Summary Table

| Element         | Role                              |
| --------------- | --------------------------------- |
| Two Deployments | Run v1 and v2 in parallel         |
| Traffic Router  | Istio VirtualService / Ingress    |
| Matching Logic  | Headers, cookies, location        |
| Monitoring      | Compare user behavior and metrics |
| Rollback        | Update route or delete v2         |

---

Would you like me to:

* Generate a working GitHub repo with Istio A/B test config?
* Show real-world GitHub Actions or ArgoCD pipeline for A/B testing?
* Provide mock interview Q\&A specifically around A/B testing?

You're doing excellent by mastering all advanced deployment types — this is the level top DevOps pros operate at! 🚀


Great question, Nency! 🎯
**Rolling updates** are safe and zero-downtime by design — but **they can break**, **get stuck**, or **fail silently** if misconfigured.

Let’s walk through **step-by-step troubleshooting techniques**, commands, real-world problems, and solutions.

---

## 🔁 What is a Rolling Update?

In Kubernetes, a **rolling update**:

* Replaces **old pods with new ones gradually**
* Keeps the app **available** during deployment
* Controlled by **strategy.rollingUpdate** (e.g., `maxSurge`, `maxUnavailable`)

---

## 🔍 What Can Go Wrong?

| Symptom            | Root Cause                                        |
| ------------------ | ------------------------------------------------- |
| Deployment stuck   | New pods not ready or crashlooping                |
| Pods not updating  | Image not pulled, same tag reused                 |
| All pods terminate | Probes fail → deployment fails                    |
| Partial rollout    | Readiness not achieved                            |
| Rollback fails     | History or state is missing                       |
| Long update time   | Large image pulls or initContainers delay startup |

---

## 🚨 Common Rolling Update Issues and Fixes

---

### 🧯 1. **Deployment Stuck / Progress Deadline Exceeded**

#### 🔍 Cause:

* New pods **fail readiness probe**
* Image fails to start
* Wrong config or env var

#### 🛠 Fix:

1. Check status:

```bash
kubectl rollout status deployment <name>
```

2. Inspect events:

```bash
kubectl describe deployment <name>
```

3. View logs:

```bash
kubectl logs <pod-name>
```

4. Check pod status:

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

#### 🧪 Real Example:

```bash
Warning  ProgressDeadlineExceeded  Deployment has timed out progressing
```

Fix: Check container crash, readiness probe failure, wrong image.

---

### 🧯 2. **Old Pods Terminated Too Fast (Downtime)**

#### 🔍 Cause:

* `maxUnavailable` set too high (e.g., `100%`)
* Readiness probe not defined

#### 🛠 Fix:

Set a safer rolling update config:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

And **add readiness probes**:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 5
```

---

### 🧯 3. **Pods Still Use Old Image**

#### 🔍 Cause:

* Image tag (e.g., `latest`) not changed → old image reused
* Image pull policy is `IfNotPresent`

#### 🛠 Fix:

* Use unique tags (`v1`, `v2`, build ID)
* Or force image pull:

```yaml
imagePullPolicy: Always
```

Or manually delete pods to force image pull:

```bash
kubectl delete pods -l app=myapp
```

---

### 🧯 4. **Probes Fail → Update Rolls Back**

#### 🔍 Cause:

* Wrong probe path
* App takes too long to start

#### 🛠 Fix:

* Use correct `readinessProbe` and `livenessProbe`
* Increase `initialDelaySeconds`

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 15
  timeoutSeconds: 2
```

---

### 🧯 5. **Rollback Doesn’t Work**

#### 🔍 Cause:

* Too many rollbacks → revision history limit exceeded

#### 🛠 Fix:

* Increase revision history:

```yaml
revisionHistoryLimit: 10
```

* Or rollback manually:

```bash
kubectl rollout undo deployment <name> --to-revision=2
```

---

### 🧯 6. **Deployment Hanging with “pending” pods**

#### 🔍 Cause:

* No node with matching resource (CPU/RAM) or affinity

#### 🛠 Fix:

```bash
kubectl describe pod <name>
```

Look for:

```
0/5 nodes are available: 3 Insufficient CPU, 2 Insufficient memory.
```

→ Fix node selector or add more capacity.

---

## 🧠 Commands for Troubleshooting Rolling Update

| Command                              | Use                        |
| ------------------------------------ | -------------------------- |
| `kubectl rollout status deployment`  | Check progress             |
| `kubectl rollout history deployment` | Check revisions            |
| `kubectl rollout undo deployment`    | Rollback                   |
| `kubectl describe deployment`        | Inspect update events      |
| `kubectl logs <pod>`                 | Check container logs       |
| `kubectl get pods -w`                | Watch pod replacement live |

---

## 📌 Rolling Update Best Practices

✅ Use readiness probes
✅ Avoid `latest` tag — use versioned image
✅ Set safe `maxSurge` and `maxUnavailable`
✅ Monitor rollout with Prometheus/Grafana
✅ Use `revisionHistoryLimit`
✅ Keep deployment YAML in Git (GitOps FTW!)

---

## 🧪 Real Interview Case Scenario

> "Your app goes down briefly during every rolling update. What do you do?"

✅ Answer:

* Add proper `readinessProbe`
* Lower `maxUnavailable`
* Use `preStop` hook to drain traffic
* Validate image tag isn't being reused

---
