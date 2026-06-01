# Kubernetes Local Dev Setup

MongoDB + Node.js webapp running on minikube (Pop!_OS 24.04, x86_64).

---

## Part 1 — Install Required Tools

### Prerequisites

| Tool | Status | Purpose |
|------|--------|---------|
| **Docker** | ✅ Already installed | Container runtime |
| **minikube** | ✅ Already installed | Local single-node K8s cluster |
| **kubectl** | Install below | CLI to control the cluster |
| **k9s** | Install below | Terminal dashboard to view cluster state |

---

### Install kubectl

Source: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get install -y kubectl
```

> To install a different K8s version replace `v1.36` in both URLs.

---

### Install k9s

Source: https://k9scli.io/topics/install/

```bash
curl -L -o /tmp/k9s.deb https://github.com/derailed/k9s/releases/download/v0.50.18/k9s_linux_amd64.deb
sudo apt-get install -y /tmp/k9s.deb
```

> Check https://github.com/derailed/k9s/releases/latest for a newer version tag.

---

### Verify installation

```bash
kubectl version --client
k9s version
kubectl get nodes       # should show minikube node as Ready
```

---

## Part 2 — Run the App on Kubernetes

### What gets deployed

```
Browser
   │ :30100
   ▼
webapp-service  (NodePort — accessible from your machine)
   │
   ▼
webapp Pod      (Node.js app — reads DB credentials + URL from K8s)
   │
   ▼
mongo-service   (ClusterIP — internal only)
   │
   ▼
mongo Pod       (MongoDB 5.0)
```

### File overview

| File | Kind | Purpose |
|------|------|---------|
| `mongo-service.yaml` | Secret | MongoDB credentials (username + password) |
| `mongo-config.yaml` | ConfigMap | MongoDB hostname (`mongo-service`) |
| `mongo.yaml` | Deployment + Service | MongoDB pod + internal service |
| `webapp.yaml` | Deployment + Service | Webapp pod + external NodePort service |

---

### Start minikube (if not already running)

```bash
minikube start
```

---

### Apply manifests — order matters

Dependencies (Secret, ConfigMap) must exist before the Deployments that reference them.

```bash
kubectl apply -f mongo-service.yaml   # 1. Secret — credentials
kubectl apply -f mongo-config.yaml    # 2. ConfigMap — DB URL
kubectl apply -f mongo.yaml           # 3. MongoDB Deployment + Service
kubectl apply -f webapp.yaml          # 4. Webapp Deployment + Service
```

---

### Verify everything is running

```bash
kubectl get pods        # both pods should show STATUS: Running
kubectl get services    # mongo-service and webapp-service should be listed
```

Expected output:

```
NAME                                READY   STATUS    RESTARTS
mongo-deployment-xxxx-xxxx          1/1     Running   0
webapp-deployment-xxxx-xxxx         1/1     Running   0
```

---

### Open the app in browser

```bash
minikube service webapp-service
```

This opens the webapp at `http://<minikube-ip>:30100` automatically.

---

### Teardown

```bash
kubectl delete -f .     # removes all deployed resources
minikube stop           # stops the cluster (does not delete it)
```

To fully delete the cluster:

```bash
minikube delete
```
