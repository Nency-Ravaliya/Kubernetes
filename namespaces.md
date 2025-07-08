## 🧠 What is a **Namespace** in Kubernetes?

> A **namespace** is a **virtual cluster** within a physical Kubernetes cluster.
> It allows **logical separation of resources**, like deployments, services, secrets, and configmaps — just like having **multiple isolated environments** in a single cluster.

---

## 🎯 Why Use Namespaces?

| Purpose                       | Example                                 |
| ----------------------------- | --------------------------------------- |
| **Environment separation**    | `dev`, `qa`, `prod` namespaces          |
| **Multi-team isolation**      | Frontend team ↔ backend team            |
| **Resource limits per group** | Quotas for memory/CPU                   |
| **Security boundaries**       | RBAC per namespace                      |
| **Cleaner organization**      | Easier kubectl usage with `--namespace` |

---

## 🛠️ Default Namespaces in Kubernetes

```bash
kubectl get namespaces
```

You'll see:

| Namespace         | Purpose                                             |
| ----------------- | --------------------------------------------------- |
| `default`         | Default for objects without a namespace             |
| `kube-system`     | System components like kube-dns, controller-manager |
| `kube-public`     | Publicly readable data, mostly unused               |
| `kube-node-lease` | For node heartbeat leases                           |
| `<custom>`        | Your custom apps (e.g., `dev`, `staging`, etc.)     |

---

## 🧱 How to Create a Namespace

```bash
kubectl create namespace dev
```

Or via YAML:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

---

## 📦 Creating Resources Inside a Namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: dev
spec:
  containers:
    - name: app
      image: nginx
```

OR use CLI flag:

```bash
kubectl create deployment myapp --image=nginx -n dev
```

---

## 🔁 Switching Context to a Namespace

```bash
kubectl config set-context --current --namespace=dev
```

Then you don’t need to pass `-n dev` again and again.

---

## 🧪 Example: Isolated Frontend & Backend

```bash
kubectl create namespace frontend
kubectl create namespace backend
```

Now deploy each team’s resources in their own namespace:

```bash
kubectl create deployment react-app --image=nginx -n frontend
kubectl create deployment api --image=node -n backend
```

Each team manages their own resources **without conflict**.

---

## 🔐 Apply RBAC to Namespace (Security)

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: dev-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

Then bind it:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: dev
subjects:
  - kind: User
    name: alice
roleRef:
  kind: Role
  name: dev-reader
  apiGroup: rbac.authorization.k8s.io
```

✅ Now Alice can only `get` or `list` pods **in dev namespace only**.

---

## 📊 Resource Quotas per Namespace

Restrict how much CPU/memory a team can use:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 1Gi
    limits.cpu: "4"
    limits.memory: 2Gi
```

Prevents resource abuse in shared clusters.

---

## 📉 Check Resources by Namespace

```bash
kubectl get all -n dev
kubectl describe namespace dev
kubectl top pod -n dev
```

---

## 📦 Namespaced vs Cluster-wide Resources

| Resource                    | Scope                |
| --------------------------- | -------------------- |
| Pods, Deployments, Services | **Namespaced**       |
| Nodes, PersistentVolumes    | **Cluster-wide**     |
| Namespace itself            | **Cluster-wide**     |
| Roles (vs. ClusterRoles)    | **Namespace-scoped** |

---

## 🧠 Interview-Ready Summary

| Question                           | Answer                                                      |
| ---------------------------------- | ----------------------------------------------------------- |
| What is a Namespace?               | Logical isolation of resources in a Kubernetes cluster      |
| Why use it?                        | Multi-team separation, resource limits, better organization |
| Default namespaces?                | `default`, `kube-system`, `kube-public`                     |
| How to deploy into a namespace?    | Set `metadata.namespace`, or use `kubectl -n`               |
| Can RBAC be applied per namespace? | Yes, using Role and RoleBinding                             |

---

## ✅ Pros and ❌ Cons

| Pros                         | Cons                                                     |
| ---------------------------- | -------------------------------------------------------- |
| ✅ Isolate teams/environments | ❌ Still shares same control plane                        |
| ✅ Apply RBAC and quotas      | ❌ No hard network isolation unless with network policies |
| ✅ Easier resource management | ❌ Cross-namespace communication needs extra steps        |

---

## 🔍 Real-World Scenarios

### 1. CI/CD Per Namespace:

Each developer/team gets their own namespace (`dev-john`, `dev-nency`) and deploys in isolation.

### 2. Multiple environments:

```bash
kubectl create ns dev
kubectl create ns qa
kubectl create ns prod
```

Same app, deployed differently in each with different configs/secrets.

---

## 🧪 Useful Commands

```bash
kubectl get namespaces
kubectl get all -n dev
kubectl describe ns prod
kubectl delete ns staging
kubectl create ns team-x
kubectl apply -f deployment.yaml -n team-x
```

---
