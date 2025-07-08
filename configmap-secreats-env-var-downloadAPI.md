## 🧠 Why Configuration Management?

> In Kubernetes, **configuration should be externalized**, reusable, and securely managed — **not hardcoded inside the image**.

---

## 📦 1. **ConfigMap**

### ✅ Purpose:

* Store **non-sensitive config data** like:

  * App settings
  * Hostnames
  * Feature toggles
  * URLs, flags, etc.

---

### 🧱 How to Create a ConfigMap

#### 🧪 A. From literal values:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_VERSION=v2.1
```

#### 🧪 B. From a file:

```bash
# config.txt
APP_ENV=dev
DB_HOST=localhost
```

```bash
kubectl create configmap app-config --from-env-file=config.txt
```

#### 🧪 C. From a YAML:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: staging
  APP_PORT: "8080"
```

---

### 🔌 Use ConfigMap in Pod (3 ways)

#### ✅ 1. As Environment Variables:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

Or specific one:

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

#### ✅ 2. As Command-line Argument:

```yaml
args: ["--env=$(APP_ENV)"]
```

#### ✅ 3. Mount as Volume:

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config

volumes:
  - name: config-volume
    configMap:
      name: app-config
```

---

## 🔐 2. **Secret**

### ✅ Purpose:

* Store **sensitive data**:

  * API keys
  * Passwords
  * TLS certs
  * Tokens

Secrets are **base64-encoded**, and access is **restricted via RBAC**.

---

### 🧱 How to Create a Secret

#### 🧪 A. From literal:

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=pass123
```

#### 🧪 B. YAML-Based:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=       # admin
  password: cGFzczEyMw==   # pass123
```

> ✅ Use `echo -n "value" | base64` to encode

---

### 🔌 Use Secrets in Pods

#### ✅ 1. As Environment Variables:

```yaml
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: username
```

#### ✅ 2. As Volume:

```yaml
volumeMounts:
  - name: secret-vol
    mountPath: "/etc/secret"

volumes:
  - name: secret-vol
    secret:
      secretName: db-secret
```

> 🧠 Mounted secrets are **read-only** and more secure.

---

## 🌿 3. Environment Variables in Pods

You can define environment variables **directly in pod spec**:

```yaml
env:
  - name: NODE_ENV
    value: production
  - name: PORT
    value: "3000"
```

✅ Or from:

* ConfigMap
* Secret
* Downward API (see next)

---

## 🔍 4. Downward API

### ✅ Purpose:

Expose **pod metadata** (like name, labels, CPU limit) as **env vars** or **files**.

---

### 🧪 Use Case: Inject Pod Name

```yaml
env:
  - name: MY_POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name
```

🧪 Use Case: Inject Namespace

```yaml
env:
  - name: MY_NAMESPACE
    valueFrom:
      fieldRef:
        fieldPath: metadata.namespace
```

🧪 Use Case: Inject CPU Request via Volume

```yaml
volumes:
  - name: podinfo
    downwardAPI:
      items:
        - path: "cpu_request"
          resourceFieldRef:
            containerName: app
            resource: requests.cpu
```

Mounts file to `/etc/podinfo/cpu_request`

---

## 🧠 Real DevOps Use Case

Let’s say you have a Node.js app that:

* Uses environment configs
* Reads DB credentials securely
* Needs to know its own pod name

✅ Use:

* **ConfigMap** → `APP_ENV`, `LOG_LEVEL`
* **Secret** → `DB_USER`, `DB_PASS`
* **Downward API** → Pod name, namespace

---

## 📉 Debugging Config Issues

| Command                                         | Use                         |
| ----------------------------------------------- | --------------------------- |
| `kubectl describe configmap <name>`             | See configmap details       |
| `kubectl describe secret <name>`                | See secret (base64 encoded) |
| `kubectl get pod <pod> -o yaml`                 | View env/volume mount       |
| `kubectl exec <pod> -- env`                     | Inspect environment         |
| `kubectl exec <pod> -- cat /etc/config/APP_ENV` | Check mounted config        |

---

## ✅ Best Practices

| Area              | Best Practice                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------- |
| 🔐 Secrets        | Use external vaults for critical data (e.g., HashiCorp Vault, AWS Secrets Manager)                                  |
| 🔁 Config changes | Re-deploy pod or use config hot-reload pattern                                                                      |
| 🔍 Access         | Restrict secret access via RBAC                                                                                     |
| 🔐 Secrets in Git | NEVER commit secrets to Git                                                                                         |
| 🔄 Updates        | ConfigMap & Secret changes do **not** automatically reload into existing pods (use checksum annotations or reapply) |

---

## 🎯 Interview-Ready Summary Table

| Concept          | Description                      | Use Example                        |
| ---------------- | -------------------------------- | ---------------------------------- |
| **ConfigMap**    | Non-secret config (env, props)   | `APP_ENV=dev`, `APP_PORT=8080`     |
| **Secret**       | Sensitive data (passwords, keys) | `DB_USER=admin`, `token=abc123`    |
| **Env vars**     | Set runtime settings             | `env: value: "prod"`               |
| **Downward API** | Inject pod metadata              | `env: pod name, namespace, limits` |

---
