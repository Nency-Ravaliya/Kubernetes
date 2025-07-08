## 🔐 What Are Network Policies?

> **NetworkPolicies** control **ingress (incoming)** and **egress (outgoing)** network traffic at the **pod level**.

By default:

* All pods **can talk to all other pods** in a Kubernetes cluster.

After applying a NetworkPolicy:

* You **restrict** who can **talk to or from a pod**.

But here's the catch:

> **They only work if your CNI (Container Network Interface) plugin supports them!**

✅ Supported CNIs:

* Calico
* Cilium
* Weave
* Antrea
* Kube-router

---

## 💡 Why Use Network Policies?

| Use Case                      | Example                                       |
| ----------------------------- | --------------------------------------------- |
| 🛡️ Block public access to DB | Only `backend` can access `db` pod            |
| 🛡️ Isolate namespace         | `teamA` pods can’t talk to `teamB`            |
| 🛡️ Allow only specific IPs   | Allow access from `monitoring` namespace only |
| 🛡️ Restrict internet access  | Allow only `egress` to specific URLs          |

---

## 🔧 Network Policy Structure

Here’s the typical YAML structure:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: <policy-name>
spec:
  podSelector:
    matchLabels:
      app: <target-pod-label>
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from: [...]   # who can talk to the pod
  egress:
    - to: [...]     # where the pod can talk to
```

---

## 🔁 INGRESS: Controlling Incoming Traffic

### 🔐 Example: Allow only `api` to talk to `db`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
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

✅ Only pods with label `role=api` can send traffic **to** pods labeled `role=db`.

---

### 🔐 Example: Allow only from a **namespace**

```yaml
ingress:
  - from:
      - namespaceSelector:
          matchLabels:
            name: monitoring
```

✅ Allows any pod from `monitoring` namespace to reach the target pod.

---

## 🔁 EGRESS: Controlling Outgoing Traffic

### 🔐 Example: Allow `app` pods to access only `mysql` service

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-egress
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: mysql
```

✅ Pods with label `app=myapp` can only **send traffic to mysql**.

---

### 🔐 Example: Allow egress to a specific IP (like external API)

```yaml
egress:
  - to:
      - ipBlock:
          cidr: 34.120.0.0/24
```

✅ Useful when you want a pod to access a 3rd-party service (like Stripe, S3, etc.)

---

## 🧱 ipBlock (For IP Whitelisting)

Allows specifying IP ranges directly:

```yaml
ipBlock:
  cidr: 10.10.0.0/16
  except:
    - 10.10.1.0/24
```

✅ Allow all traffic in `10.10.0.0/16` **except** `10.10.1.0/24`.

---

## 💥 What Happens When You Apply a Policy?

> Once a pod is selected by any **NetworkPolicy**, it **denies all traffic** **by default**, unless explicitly allowed.

So:

* ❌ No ingress if no `from` is defined.
* ❌ No egress if no `to` is defined.
* ✅ Unselected pods behave normally (all-allowed).

---

## 📦 Example: Deny All Ingress to `frontend`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
    - Ingress
  ingress: []
```

✅ `frontend` pod now rejects all incoming traffic.

---

## 👨‍💻 Real-World Scenario

> 💬 “Only allow traffic from `api` pod to `db`, block everything else”

1. Add labels:

```yaml
# db pod
labels:
  app: db

# api pod
labels:
  app: api
```

2. Create policy:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
```

---

## 🔍 Debugging NetworkPolicy Issues

| Command                                         | Purpose                              |
| ----------------------------------------------- | ------------------------------------ |
| `kubectl describe networkpolicy <name>`         | View policy rules                    |
| `kubectl get netpol`                            | List all policies                    |
| `kubectl exec <pod> -- curl <target>`           | Test access                          |
| Use `np-tracer`, `netshoot`, or `kubectl-debug` | For in-depth network troubleshooting |

---

## ✅ Best Practices

| Area            | Practice                                    |
| --------------- | ------------------------------------------- |
| Labeling        | Use consistent pod/namespace labels         |
| Policy Design   | Start with deny-all, then allow minimal     |
| Testing         | Always validate connectivity after applying |
| Namespaces      | Isolate via namespaceSelector               |
| External Access | Use `ipBlock` for CIDR/IP-specific rules    |

---

## 📘 Summary Table

| Feature             | Description                          | Use Case                  |
| ------------------- | ------------------------------------ | ------------------------- |
| `Ingress`           | Control **who can reach a pod**      | API → DB                  |
| `Egress`            | Control **where a pod can talk to**  | App → External API        |
| `podSelector`       | Match pods by labels                 | Restrict specific pods    |
| `namespaceSelector` | Match entire namespaces              | Team-based access control |
| `ipBlock`           | Match IP/CIDR ranges                 | Restrict to company IPs   |
| Default behavior    | No policy = all allowed              |                           |
| When applied        | Selected pods are blocked by default |                           |

---

## 🧠 Interview-Ready One-Liner:

> "In Kubernetes, NetworkPolicies act like **firewalls for pods**. They let you define **who can talk to whom**, both inside the cluster and to the outside world."

---
