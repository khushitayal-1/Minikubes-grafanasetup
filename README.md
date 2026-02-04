# 🚀 Kubernetes Observability Setup on Windows

This README documents the complete step-by-step setup to deploy a Python (Flask) application on Kubernetes (Minikube) and view application logs in Grafana using Loki, starting from tool installation using Chocolatey.

> ✅ Assumptions:
>
> * You are on **Windows**
> * **Docker Desktop is already installed and running**
> * Installation is done using **Chocolatey (choco)**

---

## 🧱 Architecture Overview

```
Flask App (stdout logs)
        ↓
Kubernetes Pod
        ↓
Promtail (log collector)
        ↓
Loki (log storage)
        ↓
Grafana (log visualization)
```

---

## 1️⃣ Install Required Tools (Using Chocolatey)

Open **PowerShell as Administrator**.

### Install Chocolatey (if not installed)

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; \
[System.Net.ServicePointManager]::SecurityProtocol = \
[System.Net.ServicePointManager]::SecurityProtocol -bor 3072; \
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Restart PowerShell after installation.

---

### Install Kubernetes Tools

```powershell
choco install minikube -y
choco install kubernetes-cli -y
choco install kubernetes-helm -y
```

Verify installations:

```powershell
minikube version
kubectl version --client
helm version
docker --version
```

---

## 2️⃣ Start Minikube (Using Docker Driver)

```powershell
minikube start --driver=docker
```

Verify cluster:

```powershell
kubectl get nodes
```

---

## 3️⃣ Build Docker Image Inside Minikube

We build the image **inside Minikube** so Kubernetes can use it **without Docker Hub**.

### Point Docker to Minikube

```powershell
minikube docker-env | Invoke-Expression
```

### Build Image (from project root)

```powershell
docker build -t my-python-app:1.0 .
```

Verify:

```powershell
docker images
```

---

## 4️⃣ Create Helm Chart for Python App

```powershell
helm create my-python-chart
```

Directory structure:

```
my-python-chart/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
```

### Update `values.yaml`

```yaml
image:
  repository: my-python-app
  tag: "1.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 5000
```

---

## 5️⃣ Deploy Python App Using Helm

Run from the directory **above the chart**:

```powershell
helm install my-python-release my-python-chart
```

Verify:

```powershell
kubectl get pods
kubectl get svc
```

---

## 6️⃣ Install Loki + Promtail + Grafana

### Add Grafana Helm Repository

```powershell
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### Install Loki Stack

```powershell
helm install loki grafana/loki-stack --set grafana.enabled=true
```

Verify pods:

```powershell
kubectl get pods
```

Expected:

* loki
* loki-promtail
* loki-grafana

---

## 7️⃣ Access Grafana UI

```powershell
minikube service loki-grafana
```

⚠️ Keep this terminal open (required on Windows).

---

## 8️⃣ Grafana Login Credentials

### Get admin password (PowerShell-safe way)

```powershell
$pwd = kubectl get secret loki-grafana -o jsonpath="{.data.admin-password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($pwd))
```

Login:

* **Username:** `admin`
* **Password:** output from above command

---

## 9️⃣ Configure Loki Data Source in Grafana

Grafana → **Settings → Data Sources → Add data source**

* Type: **Loki**
* URL:

  ```
  http://loki:3100
  ```
* Click **Save & Test** ✅

---

## 🔍 10️⃣ View Application Logs in Grafana

Go to **Explore → Loki**

### View all logs

```logql
{namespace="default"}
```

### View logs from Python app

```logql
{pod=~"my-python-release.*"}
```

### Filter errors

```logql
{pod=~"my-python-release.*"} |= "ERROR"
```

Make sure **time range** includes recent logs.

---

## 🧪 11️⃣ Verify Logs from Terminal

```powershell
kubectl logs <your-python-pod-name>
```

If logs appear here, they **will appear in Grafana**.

---

## 📌 Key Concepts Explained

| Component | Purpose                        |
| --------- | ------------------------------ |
| Minikube  | Local Kubernetes cluster       |
| Helm      | Package manager for Kubernetes |
| Promtail  | Collects pod logs              |
| Loki      | Stores logs                    |
| Grafana   | UI for logs & metrics          |

---

## 🏁 Conclusion

You now have:

* Tools installed via **Chocolatey**
* Kubernetes running locally with **Minikube**
* Python app deployed via **Helm**
* Centralized logging using **Loki + Grafana**

This setup closely mirrors **real production observability systems**.

---

