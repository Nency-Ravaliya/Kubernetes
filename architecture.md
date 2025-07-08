## 🏗️ **Kubernetes Architecture (In Depth)**

Kubernetes follows a **master-worker architecture** with clear separation of concerns.

---

### 🎯 **High-Level Components:**

| Layer                           | Role                                          |
| ------------------------------- | --------------------------------------------- |
| **Control Plane (Master Node)** | Brain of the cluster — makes global decisions |
| **Worker Nodes**                | Runs application workloads (containers)       |

---

## 🔹 **1. Control Plane (Master Components)**

Responsible for the **overall cluster management**.

### 🧠 1.1 `kube-apiserver`

* **Gatekeeper** of Kubernetes
* Exposes the Kubernetes API (HTTP REST interface)
* All components (kubectl, scheduler, etc.) talk to the cluster via API server

📌 *Example:* When you run `kubectl apply -f deployment.yaml`, it's received by the **API server**.

---

### 📘 1.2 `etcd` (Key-Value Store)

* Stores all cluster **state** and **configuration**
* Highly available, distributed KV store
* Stores info like:

  * Nodes
  * Pods
  * Secrets
  * ConfigMaps

📌 *Example:* If the cluster restarts, `etcd` helps restore the entire state of the cluster.

---

### 📅 1.3 `kube-scheduler`

* Assigns new pods to nodes
* Makes decisions based on:

  * Resource availability (CPU/memory)
  * Affinity rules
  * Taints and tolerations

📌 *Example:* If a pod is created but not running yet, the scheduler picks the best node to place it on.

---

### 🔧 1.4 `kube-controller-manager`

* Runs various controllers to maintain the cluster's desired state:

  * **Node Controller** – handles node failures
  * **Replication Controller** – ensures the desired number of pods
  * **Endpoint Controller** – updates services
  * **Job Controller**, **Namespace Controller**, etc.

📌 *Example:* If a pod crashes, the replication controller creates a new one to maintain the replica count.

---

### 🔒 1.5 `cloud-controller-manager`

* Manages cloud-specific resources like:

  * Load balancers
  * Persistent storage
  * Networking routes

📌 *Example:* In AWS EKS or Azure AKS, it helps auto-create LoadBalancers.

---

## 🔹 **2. Worker Node Components**

Where the actual **application containers run**.

---

### 🧱 2.1 `kubelet`

* Primary **node agent**
* Talks to the API server
* Ensures containers are running in pods as expected
* Sends **heartbeat** to control plane

📌 *Example:* If a deployment requests 3 pods, kubelet makes sure the assigned pods run on the node.

---

### 📦 2.2 `container runtime`

* Responsible for running containers
* Examples: **containerd**, **Docker**, **CRI-O**

📌 *Example:* When kubelet tells it to start a container, `containerd` or Docker does the actual run.

---

### 🌐 2.3 `kube-proxy`

* Manages networking on the node
* Implements **service discovery** and **load balancing**
* Maintains iptables or ipvs rules for routing traffic to correct pod IP

📌 *Example:* When you access a service on `NodePort`, kube-proxy forwards the request to the right pod.

---

## 🔹 **3. Additional Concepts in Architecture**

---

### 🧠 **Pod**

* Smallest deployable unit
* Contains one or more containers that share:

  * Network
  * Storage
  * Lifecycle

📌 *Example:* A Node.js app + sidecar logging container inside one pod.

---

### 📦 **Service**

* Abstracts access to pods
* Types:

  * ClusterIP (default)
  * NodePort (exposes service on host)
  * LoadBalancer (cloud-integrated)

📌 *Example:* A frontend service that routes traffic to 3 replicas of a backend pod.

---

### 🗂️ **Namespace**

* Logical partition of cluster resources
* Used for multi-team isolation, resource quota, and RBAC

📌 *Example:* `dev`, `test`, and `prod` namespaces for isolation.

---

### 🧰 **Add-ons**

* Logging: EFK, Loki
* Monitoring: Prometheus, Grafana
* Dashboard: Kubernetes Dashboard
* Autoscaler: HPA, VPA, Cluster Autoscaler

---

## 🎯 Real-World Example

Let’s say you deploy an app:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: web
          image: yatricloud/web:latest
```

Here’s what happens:

1. `kubectl apply` → **API server** receives request
2. It stores the desired state in **etcd**
3. **Scheduler** assigns pods to best nodes
4. **kubelet** on each node pulls the image and starts containers
5. **kube-proxy** enables network routing
6. You access your app via a **Service**, maybe exposed using **LoadBalancer**

---

## ✅ Summary Table

| Component          | Role                                   |
| ------------------ | -------------------------------------- |
| kube-apiserver     | Frontend API for cluster               |
| etcd               | Persistent store for state             |
| kube-scheduler     | Assigns pods to nodes                  |
| controller-manager | Reconciles desired vs actual state     |
| kubelet            | Ensures containers are running on node |
| kube-proxy         | Handles service networking             |
| container runtime  | Actually runs the containers           |
