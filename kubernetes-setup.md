# Kubernetes (K8s) Learning Setup

Setup guide for Pop!_OS 24.04 (Ubuntu-based, x86_64).

## Current status

| Tool | Status | What it's for |
|------|--------|---------------|
| **Docker** | ✅ Installed (no sudo needed) | Container runtime — runs the actual containers |
| **minikube** | ✅ Installed + cluster Running | Spins up a local single-node K8s cluster |
| **kubectl** | ⬜ To install | The core CLI — used to talk to the cluster |
| **k9s** | ⬜ To install | Visual terminal dashboard for the cluster |

---

## Step 1 — Install kubectl (official Kubernetes apt repo)

Source: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get install -y kubectl
```

> Replace `v1.36` with a different Kubernetes minor version if needed.

---

## Step 2 — Install k9s (official .deb from GitHub release)

Source: https://k9scli.io/topics/install/ — release: https://github.com/derailed/k9s/releases

```bash
curl -L -o /tmp/k9s.deb https://github.com/derailed/k9s/releases/download/v0.50.18/k9s_linux_amd64.deb
sudo apt-get install -y /tmp/k9s.deb
```

> Check https://github.com/derailed/k9s/releases/latest for the newest version tag.

---

## Step 3 — Verify

```bash
kubectl version --client
k9s version
kubectl get nodes      # should show your running minikube node
```

If `kubectl get nodes` lists a node, kubectl is correctly talking to your minikube cluster.

---

## Final learning stack

- **Docker** — runtime
- **minikube** — local cluster (already running)
- **kubectl** — the CLI you'll use constantly
- **k9s** — visual dashboard to see pods/services

## Optional (add later)

- **Helm** — package manager for K8s ("apt for Kubernetes")
- **kind / k3d** — alternative local clusters (not needed; minikube covers this)