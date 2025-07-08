# 🌐 1. StatefulSet – For Stateful Applications

## ✅ What is it?

A **StatefulSet** manages **stateful applications** — apps where:

* Each pod has a **unique identity** (e.g., pod-0, pod-1)
* Persistent storage is required for **data durability**
* Pods need to **start/stop in order** (important for clusters like Cassandra, Kafka, Redis)

---

## 🔁 Key Features

| Feature       | StatefulSet               | Deployment           |
| ------------- | ------------------------- | -------------------- |
| Pod Names     | Fixed (`pod-0`, `pod-1`)  | Random               |
| Storage       | Stable + Persistent       | Ephemeral unless PVC |
| Startup Order | Ordered                   | Parallel             |
| Scaling       | Pod identity preserved    | Generic replicas     |
| Use case      | DBs (MySQL, Redis, Kafka) | Web apps, APIs       |

---

## 📁 YAML Example: MySQL with StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"  # Required headless service
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "password"
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-persistent-storage
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 5Gi
```

---

## 🧠 Explanation:

1. **`serviceName: "mysql"`**: Links to a **headless service** which exposes each pod individually.
2. **Pod names**: Will be `mysql-0`, `mysql-1`, `mysql-2`
3. **volumeClaimTemplates**: Each pod gets a unique PVC (persistent volume claim).

   * PVC names: `mysql-persistent-storage-mysql-0`, `-1`, etc.
4. If pod-1 is deleted → it will be recreated as `mysql-1` with the **same volume**.

---

## 🧪 Test Stateful Identity:

```bash
kubectl exec -it mysql-1 -- bash
hostname   # Output will be "mysql-1"
```

---

## 🧰 Use Cases

✅ MySQL / PostgreSQL
✅ Kafka / Zookeeper
✅ Redis clusters
✅ Cassandra

---

---

# 🛠️ 2. DaemonSet – For Node-Level Agents

## ✅ What is it?

A **DaemonSet** ensures that **a copy of a pod runs on every node** (or selected nodes).

It’s used for:

* Log shippers (e.g., Fluentd, Logstash)
* Monitoring agents (e.g., Prometheus Node Exporter)
* Security tools (e.g., Falco)
* CSI storage drivers

---

## 🔁 Key Behavior

| Feature    | DaemonSet                       |
| ---------- | ------------------------------- |
| Scheduling | One pod per node                |
| Scaling    | Scales with the cluster         |
| Deletion   | Automatically cleans up pods    |
| Use case   | Node monitoring, log collection |

---

## 📁 YAML Example: Node Exporter

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter
          ports:
            - containerPort: 9100
```

---

## 🧠 Explanation:

* A pod will be scheduled on **every node** in the cluster.
* If a **new node** is added → a new pod is automatically created.
* If a **node is removed** → the DaemonSet pod is deleted too.

---

## ✍️ Target Specific Nodes

Use **node selectors / affinities** to restrict where DaemonSets run.

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
```

---

## 🧰 Use Cases

✅ Fluentd for log collection
✅ Filebeat
✅ Prometheus Node Exporter
✅ Nvidia GPU plugin
✅ Security monitoring tools

---

## ⚔️ StatefulSet vs DaemonSet vs Deployment

| Feature      | Deployment     | StatefulSet              | DaemonSet                |
| ------------ | -------------- | ------------------------ | ------------------------ |
| Pod Identity | Stateless      | Unique                   | One per node             |
| Storage      | Optional       | Persistent per pod       | Ephemeral or hostPath    |
| Order        | Parallel       | Ordered                  | All nodes                |
| Scaling      | Horizontal     | Horizontal with identity | One per node             |
| Use case     | APIs, web apps | DBs, brokers             | Agents, logging, metrics |

---

## ✅ Real Interview Questions

**Q1:**

> How would you deploy a distributed database like Cassandra in Kubernetes?

✅ Use a **StatefulSet** for identity and persistent volume per pod.

---

**Q2:**

> You want to run a log shipper on every node in the cluster. Which object to use?

✅ Use a **DaemonSet**, it will deploy one pod per node.

---

**Q3:**

> How is storage handled in StatefulSet?

✅ Through `volumeClaimTemplates`. Each pod gets a unique persistent volume.

---

**Q4:**

> If I delete a StatefulSet pod, what happens?

✅ The pod is recreated **with the same name and volume**, ensuring data is retained.

---

## 🧠 Pro Tip

* Always pair **StatefulSets with a headless service (`clusterIP: None`)**.
* Use **`volumeClaimTemplates`** only in StatefulSets (not in Deployment).
* DaemonSets are perfect for **cluster-wide tools**.

---
