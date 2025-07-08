## 💡 What is **Blue-Green Deployment**?

> Blue-Green Deployment is a strategy where **two identical production environments** (Blue and Green) exist:

* **Blue**: Currently live version
* **Green**: New version being deployed

Once the **Green** environment is fully tested and stable, traffic is **switched** from Blue to Green — instantly and without downtime.

---

## 🧱 Key Concepts of Blue-Green

| Concept              | Description                                          |
| -------------------- | ---------------------------------------------------- |
| **Two environments** | Separate live (Blue) and staging (Green)             |
| **Instant switch**   | Traffic is switched only after full validation       |
| **Zero-downtime**    | No service disruption during deployment              |
| **Rollback safety**  | Just switch back to Blue if Green has issues         |
| **Traffic Routing**  | Managed via `Service`, `Ingress`, or `Load Balancer` |

---

## 🎯 Use Case

You have a production app `v1` (Blue).
You want to deploy `v2` (Green).
You:

1. Deploy v2 (Green) on new pods
2. Test and verify internally
3. Switch traffic from Blue → Green
4. Optionally delete Blue pods after switch

---

## 🧪 Kubernetes Blue-Green Deployment (Step-by-Step)

### Step 1️⃣: Deploy the Blue Environment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
        - name: myapp
          image: yatricloud/myapp:v1
```

---

### Step 2️⃣: Create a Service Pointing to Blue

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

🔁 All traffic goes to the `blue` version pods.

---

### Step 3️⃣: Deploy the Green Version (v2)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
        - name: myapp
          image: yatricloud/myapp:v2
```

🧪 You can now test v2 **internally** (e.g., port-forward or test via staging service).

---

### Step 4️⃣: Switch the Service Selector to Green

Update the `Service` selector to:

```yaml
spec:
  selector:
    app: myapp
    version: green
```

💥 Boom! Traffic now flows to `v2` pods (Green).
Blue is still running — safe for rollback.

---

### Step 5️⃣: (Optional) Remove Blue

Once you're confident, clean up Blue deployment:

```bash
kubectl delete deployment myapp-blue
```

Or retain it temporarily for safety.

---

## 🔙 Rollback in Blue-Green

If users face issues after switch to Green:

* Change Service selector **back to Blue**
* Traffic instantly returns to old version
* No re-deployment needed
* No rollback delays or complications

---

## 🔀 Traffic Switching in Blue-Green

| Method                       | How                                    |
| ---------------------------- | -------------------------------------- |
| Kubernetes `Service`         | Change label selector                  |
| Ingress Controller           | Route based on headers or paths        |
| Load Balancer                | Update backend pools                   |
| Service Mesh (Istio/Linkerd) | Advanced traffic rules, gradual switch |

---

## ✅ Pros and ❌ Cons

| Pros                                     | Cons                                      |
| ---------------------------------------- | ----------------------------------------- |
| ✅ Zero downtime                          | ❌ Double resource usage (temp)            |
| ✅ Instant rollback                       | ❌ Needs traffic control via LB or Service |
| ✅ Safer testing in live-like environment | ❌ More setup complexity                   |
| ✅ Clean separation of old vs new         | ❌ No gradual rollout (like Canary)        |

---

## 👨‍💻 Real-World Tip

In production:

* Use **CI/CD** tools like **ArgoCD, Spinnaker, GitHub Actions** to automate Blue-Green
* Use **Ingress** or **Istio VirtualService** for clean traffic routing

---

## 🧠 Interview-Ready Summary

| Question                       | Answer                                                     |
| ------------------------------ | ---------------------------------------------------------- |
| What is Blue-Green deployment? | Two parallel environments, switch traffic after validation |
| How do you switch traffic?     | Update Kubernetes `Service` selector or LB config          |
| Can you rollback instantly?    | Yes, by pointing traffic back to previous version          |
| What's the benefit?            | Zero downtime + easy rollback                              |

---

## ⚙️ Commands You Should Know

```bash
kubectl get deployments
kubectl edit service myapp-service     # switch selector
kubectl rollout status deployment myapp-green
kubectl delete deployment myapp-blue   # cleanup old version
```

---
