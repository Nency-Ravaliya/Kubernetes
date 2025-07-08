# 🚢 What is Helm?

Helm is like **apt/yum/npm for Kubernetes**.

* It lets you **define, install, upgrade, and manage** Kubernetes applications using **Helm Charts**.
* Helm simplifies **templating**, **configuration management**, and **releases**.

---

## 🎯 Why Use Helm?

| Without Helm            | With Helm                       |
| ----------------------- | ------------------------------- |
| 10+ YAML files per app  | 1 command to install            |
| Hard-coded config       | Central `values.yaml`           |
| Manual version tracking | Built-in release history        |
| No rollback             | Easy rollback (`helm rollback`) |

---

## 🔸 1. Helm Charts – Reusable K8s Templates

> A **Helm Chart** is a **collection of YAML templates** that define a Kubernetes app.

📁 Typical chart structure:

```
my-chart/
├── Chart.yaml          # Metadata (name, version)
├── values.yaml         # Default config values
├── templates/          # Kubernetes YAML templates (Deployment, Service, etc.)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl    # Functions/macros
```

---

### 📘 Chart.yaml

```yaml
apiVersion: v2
name: myapp
description: A sample Helm chart
version: 0.1.0
appVersion: "1.16.0"
```

---

### 📘 templates/deployment.yaml (with templating)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: myapp
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

🧠 `.Values` is pulled from `values.yaml`.

---

### 📘 values.yaml (default configs)

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: stable
```

---

### ✅ Install the chart

```bash
helm install my-release ./my-chart
```

* `my-release` is the **release name**
* Helm will render the templates using values.yaml and apply them

---

## 🔸 2. values.yaml Overrides

### ✅ Customizing the deployment

Instead of modifying `values.yaml` inside the chart, you can pass a **custom file** or inline values.

#### 🔧 Use a custom values file:

`my-values.yaml`:

```yaml
replicaCount: 3
image:
  tag: latest
```

Install with:

```bash
helm install my-release ./my-chart -f my-values.yaml
```

#### 🔧 Inline override:

```bash
helm install my-release ./my-chart --set replicaCount=4,image.tag=1.21
```

---

### 🧠 Best Practice

✅ Keep your chart generic and **override values per environment**:

* `dev-values.yaml`
* `prod-values.yaml`

---

## 🔸 3. Deploying Apps Using Helm

Let's say you want to deploy Redis using Helm:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install redis-release bitnami/redis
```

✅ That single command will:

* Download the chart
* Render YAMLs
* Create StatefulSet, PVCs, Services, etc.
* Track the release

---

### 🔍 Inspect the release:

```bash
helm list                    # View all releases
helm status redis-release   # Check current status
helm get all redis-release  # View rendered templates
```

---

### 🔁 Upgrade / rollback:

```bash
helm upgrade redis-release bitnami/redis --set auth.password=securepass
helm rollback redis-release 1
```

---

## 🔸 4. Chart Repositories

> Chart repos are like npm or pip registries for Helm.

Popular public repos:

* Bitnami: `https://charts.bitnami.com/bitnami`
* Prometheus Community: `https://prometheus-community.github.io/helm-charts`
* Grafana: `https://grafana.github.io/helm-charts`

---

### 📦 Commands:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo redis
```

---

### 📁 Create your own chart:

```bash
helm create my-chart
```

Creates a complete template folder with example templates and values.

---

## 🧠 Real-World Use Cases

**1. Deploy Prometheus + Grafana monitoring stack:**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack
```

**2. Release versioned microservices:**

```bash
helm upgrade myapp ./my-chart --set image.tag=v1.2.3
```

**3. Create dev/prod environments:**

```bash
helm install myapp-dev ./my-chart -f dev-values.yaml
helm install myapp-prod ./my-chart -f prod-values.yaml
```

---

## ✅ Summary Table

| Concept         | Description                           |
| --------------- | ------------------------------------- |
| Helm Chart      | Package format for Kubernetes apps    |
| `Chart.yaml`    | Metadata (name, version, etc.)        |
| `values.yaml`   | Default config values                 |
| `templates/`    | Templated Kubernetes manifests        |
| `helm install`  | Deploy app                            |
| `helm upgrade`  | Update config/image                   |
| `helm rollback` | Roll back to previous version         |
| Chart Repo      | Source of community-maintained charts |
| `--set` / `-f`  | Override values                       |

---

## 🔥 Interview Ready Elevator Pitch

> “Helm simplifies Kubernetes deployments by packaging Kubernetes manifests into versioned, reusable charts. With Helm, I can deploy complex applications like Prometheus, NGINX, or custom microservices using templates and configuration overrides — making deployments consistent, repeatable, and environment-specific.”

---
