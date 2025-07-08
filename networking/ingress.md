## 🧠 What is Ingress in Kubernetes?

> **Ingress** is an API object that manages **external HTTP/S access to services** in a cluster, typically via **hostnames or paths**.

Unlike NodePort or LoadBalancer, Ingress:

* Operates at **Layer 7 (HTTP)**
* Allows **routing rules**, **TLS termination**, and **URL rewrites**
* Uses a single external IP to **route to multiple services**

---

## 🏗️ Core Ingress Concepts

| Concept                | Description                                                     |
| ---------------------- | --------------------------------------------------------------- |
| **Ingress Resource**   | Defines routing rules (host/path → service)                     |
| **Ingress Controller** | A pod that implements the logic (e.g., NGINX, Traefik, HAProxy) |
| **TLS Termination**    | Supports HTTPS at the ingress level                             |
| **Annotations**        | Fine-tune behavior (rewrite, timeout, rate-limiting)            |
| **Path Types**         | Exact, Prefix, ImplementationSpecific                           |
| **Default Backend**    | Catch-all if no rule matches                                    |

---

## ⚙️ Step-by-Step Setup

---

### ✅ 1. Install an Ingress Controller

Let’s use **NGINX Ingress Controller**:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.0/deploy/static/provider/cloud/deploy.yaml
```

Wait for the controller pod:

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx  # to get the external IP
```

---

### ✅ 2. Deploy Two Services (Example)

Let’s create two apps:

#### app1.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
        - name: app1
          image: hashicorp/http-echo
          args:
            - "-text=Welcome to App1"
---
apiVersion: v1
kind: Service
metadata:
  name: app1-svc
spec:
  selector:
    app: app1
  ports:
    - port: 80
      targetPort: 5678
```

Repeat for **app2** with `-text=Welcome to App2`.

---

### ✅ 3. Create Ingress Resource

#### ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: demo.local
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

---

### ✅ 4. Test Routing

Edit your `/etc/hosts` file (Linux/Mac):

```
<External-IP-of-Ingress>   demo.local
```

Then test:

```bash
curl http://demo.local/app1
curl http://demo.local/app2
```

---

## 🔐 TLS/HTTPS with Ingress

TLS termination is super easy using Ingress:

1. Create a TLS secret:

```bash
kubectl create secret tls my-tls-secret \
  --cert=tls.crt --key=tls.key
```

2. Update your Ingress:

```yaml
spec:
  tls:
    - hosts:
        - demo.local
      secretName: my-tls-secret
```

✅ Now, NGINX terminates HTTPS using your TLS certificate.

---

## 🧠 Path Types

| Type                     | Meaning                                                |
| ------------------------ | ------------------------------------------------------ |
| `Prefix`                 | Match any path with given prefix (`/app`, `/app1/api`) |
| `Exact`                  | Match exactly (`/healthz`)                             |
| `ImplementationSpecific` | Depends on the controller, not recommended             |

---

## 🔧 Common Annotations

| Annotation                                    | Purpose                 |
| --------------------------------------------- | ----------------------- |
| `nginx.ingress.kubernetes.io/rewrite-target`  | Rewrites `/app1` to `/` |
| `nginx.ingress.kubernetes.io/ssl-redirect`    | Force HTTPS             |
| `nginx.ingress.kubernetes.io/proxy-body-size` | Increase upload size    |
| `nginx.ingress.kubernetes.io/limit-rps`       | Rate limiting           |
| `nginx.ingress.kubernetes.io/auth-url`        | External auth URL       |

---

## 🧪 Example: Rewrite Target

If backend service doesn’t understand `/app1`, rewrite to `/`:

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
```

So `http://demo.local/app1` is internally routed as `/` to `app1-svc`.

---

## 🧠 Default Backend (Optional)

If no rule matches, you can define a default backend to show a 404 page or custom error.

```yaml
spec:
  defaultBackend:
    service:
      name: default-svc
      port:
        number: 80
```

---

## 🧪 Real Interview Scenario

> ❓“How would you expose 5 microservices over a single domain using Ingress?”

✅ Use a single Ingress resource with **path-based routing**:

```yaml
- path: /user → user-svc
- path: /order → order-svc
- path: /admin → admin-svc
```

---

## 🔍 Debugging Ingress

| Command                                               | Purpose                          |
| ----------------------------------------------------- | -------------------------------- |
| `kubectl get ingress`                                 | List ingresses and their address |
| `kubectl describe ingress <name>`                     | Show path rules, events          |
| `kubectl get svc -n ingress-nginx`                    | Check LB IP                      |
| `kubectl logs <ingress-controller-pod>`               | See ingress routing issues       |
| `curl -H "Host: demo.local" http://<ingress-ip>/app1` | Test without DNS setup           |

---

## 📌 Best Practices

| Practice                                    | Description             |
| ------------------------------------------- | ----------------------- |
| ✅ Use `Prefix` or `Exact` paths             | Avoid ambiguous routing |
| ✅ Enable TLS                                | Secure public traffic   |
| ✅ Set resource limits on Ingress controller | Prevent abuse           |
| ✅ Keep separate Ingress per team or app     | Better maintainability  |
| ✅ Version your ingress YAML with Git        | GitOps FTW              |

---

## 🧠 Interview Summary Table

| Feature            | Purpose                                      |
| ------------------ | -------------------------------------------- |
| Ingress            | HTTP/S entry point with routing              |
| Ingress Controller | Actual implementation (e.g., NGINX, Traefik) |
| TLS                | Secure connections (HTTPS)                   |
| Annotations        | Fine-tune routing and behavior               |
| Path Routing       | URL-based routing to multiple services       |
| Host Routing       | Route based on domain (`api.myapp.com`)      |

---
