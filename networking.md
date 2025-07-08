## 🔗 1. ClusterIP (Default)

### ✅ What is it?

* **Default service type**.
* Exposes a service **only inside the cluster**.
* Automatically assigned a **virtual IP (VIP)** within the cluster.

### 📦 Use Case:

* Backend services (like `db`, `auth`, `api`) that don’t need external access.

### 🧪 Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

🔹 You can access it from another pod via:

```bash
curl http://backend-svc:80
```

---

## 🌐 2. NodePort

### ✅ What is it?

* Exposes the service on a **static port (30000–32767)** on **each worker node**.
* Allows **external access** using:

  ```
  http://<NodeIP>:<NodePort>
  ```

### 📦 Use Case:

* Access from outside the cluster (for dev or testing).
* Useful in bare-metal clusters.

### 🧪 Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-svc
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

🔹 You can now access it like:

```bash
curl http://<any-node-ip>:30080
```

---

## ☁️ 3. LoadBalancer

### ✅ What is it?

* Provisions a **cloud-based Load Balancer** (e.g., AWS ELB, Azure LB).
* Exposes your service **publicly** via external IP or DNS.
* Used in **production-grade cloud clusters**.

### 📦 Use Case:

* Frontend apps or public APIs.
* Deployments in cloud (EKS, AKS, GKE).

### 🧪 Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-svc
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

🔹 Result:

```bash
kubectl get svc
# Look for EXTERNAL-IP
```

🧠 Note: Works only if the cluster is integrated with cloud provider.

---

## 🌐 4. Ingress (Layer 7 HTTP Routing)

### ✅ What is it?

* **Ingress** is NOT a service — it’s a **Kubernetes resource** for:

  * **HTTP routing** (based on host/path)
  * **TLS termination**
  * **Single entry point** for multiple apps

* Requires an **Ingress Controller** like:

  * NGINX Ingress Controller
  * Traefik
  * Istio Gateway

---

### 📦 Use Case:

* Route multiple microservices using paths:

  * `/app1 → service1`
  * `/app2 → service2`
* Terminate HTTPS
* Use domain names per service

---

### 🧪 Example:

#### A. Deploy Ingress Controller (NGINX - 1-time setup)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.0/deploy/static/provider/cloud/deploy.yaml
```

#### B. Define Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app1-svc
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app2-svc
                port:
                  number: 80
```

✅ Now route traffic via:

```
http://myapp.example.com/app1 → app1-svc
http://myapp.example.com/app2 → app2-svc
```

---

## 🔐 5. Network Policies

### ✅ What is it?

* Used to **control traffic between pods**.
* Defines **who can talk to whom** (ingress/egress).
* By default, **all pods can communicate** (open).

To **enforce policies**, you must use a **network plugin** like Calico, Cilium, etc.

---

### 📦 Use Case:

* Block frontend from accessing `db` directly.
* Allow only `api` pod to talk to `db`.

---

### 🧪 Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: api
```

✅ Effect:

* Only pods with label `role=api` can connect to pods labeled `role=db`.

---

## 🧠 Visual Summary

```
                    Ingress (URL routing) 
                          |
                ┌─────────┴─────────┐
                |                   |
        /api → api-svc        /web → web-svc
                |                   |
         ClusterIP → Pods     ClusterIP → Pods

     LoadBalancer/NodePort → for external access
     NetworkPolicy → controls who can talk to whom
```

---

## 🧪 Debugging Networking Issues

| Command                                   | Purpose                         |
| ----------------------------------------- | ------------------------------- |
| `kubectl get svc`                         | Check service type & clusterIP  |
| `kubectl get ingress`                     | Check routing setup             |
| `kubectl get endpoints`                   | Ensure service connects to pods |
| `kubectl exec <pod> -- curl <svc>:<port>` | Test internal access            |
| `kubectl describe svc/ingress`            | Debug DNS, routing              |

---

## ✅ Interview Summary Table

| Concept           | Description                     | Use Case                   |
| ----------------- | ------------------------------- | -------------------------- |
| **ClusterIP**     | Internal-only service (default) | App-to-app traffic         |
| **NodePort**      | Exposes service on Node IP      | Local testing/dev          |
| **LoadBalancer**  | Cloud-external public IP        | Public app access          |
| **Ingress**       | Layer 7 routing (HTTP/HTTPS)    | Multiple apps behind 1 DNS |
| **NetworkPolicy** | Restrict pod-to-pod traffic     | Secure microservices       |

---
