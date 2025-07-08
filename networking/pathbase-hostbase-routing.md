## 🔀 Types of Ingress Routing in Kubernetes

There are **two main types** of routing in Kubernetes Ingress:

### 1. **Host-Based Routing**

### 2. **Path-Based Routing**

Each uses a specific **pathType** to define how the Ingress controller matches incoming requests.

---

## 🧭 1. **Host-Based Routing**

> Routes based on the **domain name (host header)**.

### ✅ Use Case:

You want different subdomains for different services:

```
api.example.com  →  api-service
web.example.com  →  frontend-service
```

### 🧪 Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  rules:
    - host: web.yatri-cloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-svc
                port:
                  number: 80
    - host: api.yatri-cloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 80
```

✅ Now:

* `curl http://web.yatri-cloud.com` → routes to `frontend-svc`
* `curl http://api.yatri-cloud.com` → routes to `api-svc`

🔐 **You must set the Host header manually for testing** or configure DNS:

```bash
curl -H "Host: web.yatri-cloud.com" http://<INGRESS_IP>
```

---

## 🛣️ 2. **Path-Based Routing**

> Routes traffic based on the **URL path** requested.

### ✅ Use Case:

You want one domain with multiple services behind different paths:

```
example.com/app1 → app1-service
example.com/app2 → app2-service
```

---

### 🧪 Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: example.com
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

✅ Now:

* `example.com/app1` → routed to `app1-svc`
* `example.com/app2` → routed to `app2-svc`

Use rewrite-target `/` if the app doesn't expect a path prefix.

---

## 📘 Ingress `pathType` in Detail

| pathType                   | Behavior                                       | Example Path                      | Notes                  |
| -------------------------- | ---------------------------------------------- | --------------------------------- | ---------------------- |
| **Prefix**                 | Matches any path that starts with given prefix | `/app` matches `/app`, `/app/xyz` | Most used              |
| **Exact**                  | Matches exact path only                        | `/app` matches only `/app`        | Does NOT match `/app/` |
| **ImplementationSpecific** | Depends on controller (not portable)           | *Controller-defined*              | ⚠️ Avoid in production |

---

### ✅ Prefix Match Example:

```yaml
- path: /api
  pathType: Prefix
  backend:
    service:
      name: api-svc
      port:
        number: 80
```

Matches:

* `/api`
* `/api/v1/users`
* `/api/anything/here`

---

### ✅ Exact Match Example:

```yaml
- path: /healthz
  pathType: Exact
  backend:
    service:
      name: health-svc
      port:
        number: 80
```

Matches:

* ✅ `/healthz`
  Does **not match**:
* ❌ `/healthz/`
* ❌ `/healthz/status`

---

### ⚠️ ImplementationSpecific (Not Recommended)

If you **don’t define** `pathType`, it defaults to this, but behavior varies between controllers.

```yaml
pathType: ImplementationSpecific
```

* May match `/api`, `/api/`, or even `/api/v1`
* Use **Prefix** or **Exact** to ensure predictability

---

## 🧪 Bonus: Combine Host & Path Routing

You can combine **host + path** to build powerful ingress routing:

```yaml
rules:
  - host: dev.yatri-cloud.com
    http:
      paths:
        - path: /v1
          pathType: Prefix
          backend:
            service:
              name: dev-v1-svc
              port:
                number: 80
  - host: prod.yatri-cloud.com
    http:
      paths:
        - path: /v1
          pathType: Prefix
          backend:
            service:
              name: prod-v1-svc
              port:
                number: 80
```

✅ This way:

* `dev.yatri-cloud.com/v1` → dev-v1-svc
* `prod.yatri-cloud.com/v1` → prod-v1-svc

---

## 🔍 Tips for Testing Locally

1. Get Ingress IP:

```bash
kubectl get ingress
```

2. Add to `/etc/hosts`:

```bash
<INGRESS-IP> demo.local
```

3. Test with:

```bash
curl http://demo.local/app1
```

---

## 🎯 Summary Table

| Routing Type    | Example                           | Use Case                      |
| --------------- | --------------------------------- | ----------------------------- |
| **Host-based**  | `api.myapp.com` → `api-svc`       | Microservices with subdomains |
| **Path-based**  | `/auth` → `auth-svc`              | All apps under one domain     |
| **Prefix**      | `/user` matches `/user/123`       | Most common                   |
| **Exact**       | `/healthz` matches only that      | Healthcheck endpoints         |
| **Host + Path** | `dev.app.com/api` → `dev-api-svc` | Per-env routing               |

---
