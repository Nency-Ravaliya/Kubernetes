## 🔧 **kubectl CLI & Basic Commands (Deep Dive)**

> `kubectl` is the command-line interface to interact with the Kubernetes cluster via the `kube-apiserver`.

---

### 🔹 **1. Cluster Info & Context**

| Command                            | Purpose                       |
| ---------------------------------- | ----------------------------- |
| `kubectl version`                  | Show client/server version    |
| `kubectl cluster-info`             | Display cluster details       |
| `kubectl config view`              | Show kubeconfig settings      |
| `kubectl config get-contexts`      | List available contexts       |
| `kubectl config use-context <ctx>` | Switch between clusters       |
| `kubectl get nodes`                | View all nodes in the cluster |

📌 **Example:**

```bash
kubectl version --short
kubectl config use-context aks-prod
```

---

### 🔹 **2. Get Cluster Resources**

| Command                 | Purpose                      |
| ----------------------- | ---------------------------- |
| `kubectl get pods`      | Show all pods                |
| `kubectl get svc`       | Show services                |
| `kubectl get deploy`    | Show deployments             |
| `kubectl get all`       | Show pods, services, deploys |
| `kubectl get events`    | Cluster-level event logs     |
| `kubectl get namespace` | Show namespaces              |

📌 **Example:**

```bash
kubectl get pods -n dev
kubectl get svc -o wide
```

---

### 🔹 **3. Describe Resources (Detailed Info)**

| Command                          | Purpose                 |
| -------------------------------- | ----------------------- |
| `kubectl describe pod <pod>`     | Detailed info + events  |
| `kubectl describe deploy <name>` | View deployment rollout |
| `kubectl describe node <node>`   | Node-level detail       |

📌 **Example:**

```bash
kubectl describe pod myapp-7b8d99c9
kubectl describe node node-1
```

---

### 🔹 **4. Create / Apply / Delete Resources**

| Command                         | Purpose                        |
| ------------------------------- | ------------------------------ |
| `kubectl apply -f <file.yaml>`  | Create/update from YAML        |
| `kubectl create -f <file.yaml>` | Create (fails if exists)       |
| `kubectl delete -f <file.yaml>` | Delete from file               |
| `kubectl delete pod <name>`     | Delete specific pod            |
| `kubectl delete all --all`      | Delete everything in namespace |

📌 **Example:**

```bash
kubectl apply -f myapp.yaml
kubectl delete svc frontend
```

---

### 🔹 **5. Exec & Logs (Troubleshooting)**

| Command                               | Purpose                |
| ------------------------------------- | ---------------------- |
| `kubectl logs <pod>`                  | View logs              |
| `kubectl logs -f <pod>`               | Tail logs              |
| `kubectl exec <pod> -- <cmd>`         | Run command inside pod |
| `kubectl exec -it <pod> -- /bin/bash` | Get shell inside pod   |

📌 **Example:**

```bash
kubectl logs myapp-pod
kubectl exec -it myapp-pod -- bash
```

---

### 🔹 **6. Port Forwarding**

| Command                                              | Purpose                 |
| ---------------------------------------------------- | ----------------------- |
| `kubectl port-forward pod/<pod> <local>:<container>` | Access services locally |

📌 **Example:**

```bash
kubectl port-forward pod/myapp-7b8d 8080:80
```

---

### 🔹 **7. Rollout & Scaling**

| Command                                    | Purpose                   |
| ------------------------------------------ | ------------------------- |
| `kubectl rollout status deploy/<name>`     | Check rollout status      |
| `kubectl rollout history deploy/<name>`    | View revision history     |
| `kubectl rollout undo deploy/<name>`       | Roll back to last version |
| `kubectl scale deploy <name> --replicas=N` | Change number of pods     |

📌 **Example:**

```bash
kubectl rollout undo deploy myapp
kubectl scale deployment myapp --replicas=5
```

---

### 🔹 **8. YAML Generation & Dry Run**

| Command                                              | Purpose                |
| ---------------------------------------------------- | ---------------------- |
| `kubectl create deploy ... --dry-run=client -o yaml` | Generate YAML          |
| `kubectl explain <resource>`                         | Explain fields in spec |

📌 **Example:**

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl explain pod.spec.containers
```

---

### 🔹 **9. Namespaces**

| Command                                                | Purpose               |
| ------------------------------------------------------ | --------------------- |
| `kubectl get ns`                                       | List namespaces       |
| `kubectl config set-context --current --namespace=dev` | Set default namespace |

📌 **Example:**

```bash
kubectl get pods -n prod
```

---

### 🔹 **10. Apply Patch / Edit Live**

| Command                    | Purpose                          |
| -------------------------- | -------------------------------- |
| `kubectl edit <resource>`  | Edit resource live in editor     |
| `kubectl patch <resource>` | JSON/YAML patch on live resource |

📌 **Example:**

```bash
kubectl edit deployment myapp
kubectl patch deployment myapp -p '{"spec": {"replicas": 4}}'
```

---

## 🧪 Bonus: Useful Shortcuts

| Shortcut | Expanded    |
| -------- | ----------- |
| `po`     | pods        |
| `svc`    | services    |
| `deploy` | deployments |
| `rs`     | ReplicaSet  |
| `ns`     | namespaces  |

📌 **Example:**

```bash
kubectl get po
kubectl get svc
```
