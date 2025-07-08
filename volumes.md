## 📦 Overview of Kubernetes Storage Concepts

| Resource                        | Purpose                                                   |
| ------------------------------- | --------------------------------------------------------- |
| **Volume**                      | Ephemeral storage shared within a pod                     |
| **PersistentVolume (PV)**       | Cluster-wide provisioned storage (NFS, EBS, etc.)         |
| **PersistentVolumeClaim (PVC)** | A pod’s request for persistent storage                    |
| **StorageClass**                | Defines *how* storage is provisioned (dynamically/static) |

---

## 🔹 1. Volume

### ✅ What is it?

A **Kubernetes volume** is a piece of storage that is accessible to containers in a pod.

🧠 Unlike Docker volumes, these:

* Can survive container restarts (but not pod restarts)
* Are ephemeral (deleted with pod)

### 🧪 Example: EmptyDir (default type)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-vol-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: /data
          name: shared-volume
  volumes:
    - name: shared-volume
      emptyDir: {}
```

✅ This creates a **temporary volume** `/data` shared across containers in the same pod.

🧨 **Deleted when the pod is deleted**.

---

## 🔹 2. PersistentVolume (PV)

### ✅ What is it?

A **PersistentVolume** is a **cluster-level storage resource**.
It represents **actual physical storage** like:

* AWS EBS, GCP Persistent Disk
* NFS share
* CephFS
* Local disk

PV is **provisioned by admin** (static) or by StorageClass (dynamic).

### 🧪 Example: Static NFS PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 10.10.10.10
    path: /exports/data
  persistentVolumeReclaimPolicy: Retain
```

✅ Now the cluster has a 1Gi PV backed by NFS.

---

## 🔹 3. PersistentVolumeClaim (PVC)

### ✅ What is it?

A **PVC** is a **request for storage** by a pod:

* Size
* Access mode (RWO, ROX, RWX)
* StorageClass (optional)

Kubernetes matches a **PVC to a suitable PV**.

### 🧪 Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

✅ Kubernetes binds this PVC to a matching available PV.

---

## 🔹 4. StorageClass

### ✅ What is it?

A **StorageClass** defines how PVs are **dynamically provisioned**.

Instead of pre-creating PVs, you let Kubernetes **auto-provision** using cloud-native volumes like:

* AWS: EBS
* GCP: pd-standard / pd-ssd
* Azure: managed disk

---

### 🔧 StorageClass Example (GCE SSD):

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

Then your PVC looks like:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: fast
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

✅ Kubernetes automatically provisions SSD-backed PV via the `fast` StorageClass.

---

## ⚡ Access Modes

| Mode                  | Description               |
| --------------------- | ------------------------- |
| `ReadWriteOnce` (RWO) | One node can read/write   |
| `ReadOnlyMany` (ROX)  | Many nodes can read       |
| `ReadWriteMany` (RWX) | Many nodes can read/write |

✅ Choose based on workload. RWX is useful for shared logs or NFS.

---

## 🛠️ Putting It All Together

### Full Dynamic Provisioning Example (using StorageClass):

```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: standard

---
apiVersion: v1
kind: Pod
metadata:
  name: volume-demo
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo Hello > /mnt/data/hello && sleep 3600"]
      volumeMounts:
        - mountPath: /mnt/data
          name: app-storage
  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: data-pvc
```

✅ The pod writes to `/mnt/data`, which persists even after pod restarts.

---

## 🔄 Reclaim Policy (PV Cleanup)

| Policy    | Behavior                              |
| --------- | ------------------------------------- |
| `Retain`  | Keeps the data after PVC is deleted   |
| `Recycle` | Wipes data (deprecated)               |
| `Delete`  | Deletes the storage (common in cloud) |

---

## 📌 Volume Types Summary

| Volume Type             | Description                         | Persistent?      |
| ----------------------- | ----------------------------------- | ---------------- |
| `emptyDir`              | Shared temp data for a pod          | ❌                |
| `hostPath`              | Mounts host file system             | ❌ (not portable) |
| `nfs`                   | Network file system                 | ✅                |
| `awsElasticBlockStore`  | AWS EBS Volume                      | ✅                |
| `persistentVolumeClaim` | Refers PVC                          | ✅                |
| `configMap` / `secret`  | Inject config or secrets as volumes | N/A              |

---

## 👨‍💻 Real-World Interview Scenarios

**Q1:**

> Your app pod gets deleted and restarts. How do you retain user uploads?

✅ Use a **PVC** mounted to `/uploads`.

---

**Q2:**

> How do you provision storage dynamically in Kubernetes?

✅ Define a **StorageClass**, then create a PVC that refers to it.

---

**Q3:**

> How can multiple pods access the same storage?

✅ Use an RWX-capable backend (like NFS) and request it in PVC.

---

## ✅ Summary Table

| Concept                         | What it Does                 | Key Point        |
| ------------------------------- | ---------------------------- | ---------------- |
| **Volume**                      | Temporary storage inside pod | Ephemeral        |
| **PersistentVolume (PV)**       | Pre-provisioned storage      | Cluster resource |
| **PersistentVolumeClaim (PVC)** | Request for storage          | Bound to PV      |
| **StorageClass**                | Defines dynamic provisioning | Used in PVC      |
| **AccessModes**                 | Who can read/write           | RWO, ROX, RWX    |
| **ReclaimPolicy**               | What happens on delete       | Delete / Retain  |

---
