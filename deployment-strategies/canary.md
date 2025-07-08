## 🐦 What is **Canary Deployment**?

> A **Canary Deployment** is a technique to **gradually roll out a new version** of an application to a **small subset of users**, monitor its performance, and then roll it out to everyone if everything looks good.

🔁 The term comes from "canary in a coal mine" — using a small group (canary) to test the safety of a new change.

---

## ✅ Why Canary is Important

| Benefit              | Explanation                                          |
| -------------------- | ---------------------------------------------------- |
| ✅ Safer Releases     | You can catch bugs early before full rollout         |
| ✅ Metrics-driven     | You can monitor CPU, latency, error rate             |
| ✅ Easy Rollback      | If something breaks, only a small subset is impacted |
| ✅ Controlled Rollout | You can go from 5% → 25% → 50% → 100% safely         |

---

## 🧠 Real-World Example

Imagine you have a running app `v1` in production, and you want to deploy `v2`.

### Canary Steps:

1. Deploy `v2` to **10% of users**
2. Monitor performance, logs, metrics
3. If stable, increase to **25%**, then **50%**, then **100%**
4. If failure occurs, **rollback to v1** instantly

---

## 🧱 Core Concepts of Canary Deployment

| Concept                            | Description                                                 |
| ---------------------------------- | ----------------------------------------------------------- |
| **Weight-based traffic splitting** | Route 5%, 10%, etc. of traffic to the new version           |
| **Label-based pod selection**      | Use labels like `version: v1`, `version: v2`                |
| **Monitoring & metrics**           | Use Prometheus, Grafana, or Service Mesh to observe traffic |
| **Rollback logic**                 | If metrics cross threshold, stop and revert                 |
| **Progressive rollout**            | Controlled increase of traffic over time                    |

---

## 🔧 Canary via Kubernetes (Manual)

Let’s simulate it manually using **2 Deployments** and **1 Service**.

---

### 🧪 Step 1: Deploy Stable Version (v1)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 3
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

---

### 🧪 Step 2: Deploy Canary Version (v2)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v2
spec:
  replicas: 1
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

### 🧪 Step 3: Use a Shared Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
```

👆 This service sends traffic to both `v1` and `v2` (default load balancing based on pod count).

If you have:

* 3 pods of v1
* 1 pod of v2

Then \~25% of traffic goes to v2 (canary).

---

## 📈 How to Monitor?

* Use **Prometheus** + **Grafana** for metrics
* Use **Datadog**, **New Relic**, or **Elastic APM**
* Watch:

  * Error rate
  * Latency
  * Logs (stdout, stderr)
  * Resource usage

---

## 🔄 Canary Rollout Flow (Manual)

```bash
# Increase v2 replicas to 3, v1 to 2 (60% v2)
kubectl scale deployment myapp-v2 --replicas=3
kubectl scale deployment myapp-v1 --replicas=2

# Final step: scale down v1
kubectl scale deployment myapp-v1 --replicas=0
```

---

## 🧰 Advanced Canary (Automated via Tools)

| Tool                | Feature                                        |
| ------------------- | ---------------------------------------------- |
| **Argo Rollouts**   | Native Kubernetes CRD for Canary & Blue-Green  |
| **Flagger**         | Kubernetes controller for progressive delivery |
| **Istio / Linkerd** | Traffic shaping + routing for Canary testing   |
| **Spinnaker**       | Full-featured CD with Canary configs           |
| **LaunchDarkly**    | Feature flag-based gradual rollouts            |

---

## 🧪 Canary via Argo Rollouts (Example)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: { duration: 60s }
        - setWeight: 50
        - pause: { duration: 60s }
        - setWeight: 100
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: yatricloud/myapp:v2
```

➡ This rollout does:

* 20% traffic
* Pause 60s
* 50%
* Pause again
* Full 100% rollout

🔥 Argo Rollouts also supports:

* Webhooks
* Analysis templates
* Auto rollback

---

## 🔙 Canary Rollback Plan

1. Monitor metrics after each step
2. If error rate > threshold → stop rollout
3. Scale down canary pods
4. Restore stable deployment (v1)

Rollback in `kubectl`:

```bash
kubectl rollout undo deployment myapp
```

Or with Argo:

```bash
kubectl argo rollouts undo myapp
```

---

## ✅ Pros and ❌ Cons

| Pros                 | Cons                               |
| -------------------- | ---------------------------------- |
| ✅ Low risk           | ❌ Needs monitoring integration     |
| ✅ Quick feedback     | ❌ Slightly complex setup           |
| ✅ Controlled release | ❌ Service must load-balance evenly |
| ✅ Easy rollback      | ❌ Manual steps if no tooling used  |

---

## 🧠 Interview-Ready Summary

| Question                             | Answer                                             |
| ------------------------------------ | -------------------------------------------------- |
| What is Canary Deployment?           | Gradual rollout of new version to small % of users |
| Why use it?                          | Catch bugs early, safe rollback, reduce impact     |
| How do you implement?                | Multiple deployments or tools like Argo Rollouts   |
| What's needed to support it?         | Traffic routing + monitoring setup                 |
| How is it different from Blue-Green? | Canary = gradual; Blue-Green = full switch         |

---

## 🎯 Real DevOps Use Case

* **App**: Frontend React app
* **Current version**: v1 with 5 pods
* **New version**: v2 with 1 pod
* **Service**: Balancing traffic to all pods

🚦 If metrics on v2 look good after 5 minutes, gradually scale up v2 and scale down v1 — without ever dropping production.

---
