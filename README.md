# README.md — Kubernetes Scheduling Lab

# ☸ Kubernetes Scheduling — Hands-On Lab

A step-by-step hands-on lab to understand **Kubernetes pod scheduling concepts** using a local **kind (Kubernetes IN Docker)** cluster.

## You will learn

- NodeSelector scheduling
- Node Affinity (soft & hard rules)
- Taints & Tolerations
- Pod scheduling behavior and debugging

---

## 📌 Prerequisites

Make sure the following tools are installed and running:

- 🐳 **Docker Desktop** → must be running
    
    [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
    
- ☸ **kind (Kubernetes IN Docker)**
    
    [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/)
    
- ⚙️ **kubectl (Kubernetes CLI)**
    
    [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)
    

---

## 📂 Lab File Structure (Follow in Order)

| Step | File | Concept |
| --- | --- | --- |
| 01 | `01-kind-cluster.yaml` | Create local Kubernetes cluster |
| 02 | `02-node-selector.yaml` | Basic node label-based scheduling |
| 03 | `03-node-affinity-preferred.yaml` | Soft scheduling (preferred rules) |
| 04 | `04-node-affinity-required.yaml` | Hard scheduling (required rules) |
| 05 | `05-no-schedule-taint-toleration.yaml` | Taints & Tolerations |

---

## 🚀 Step-by-Step Lab Guide

### 1️⃣ Create Kubernetes Cluster

```bash
# Create a local multi-node cluster
kind create cluster --name scheduling-lab --config 01-kind-cluster.yaml

# Verify nodes
kubectl get nodes

# Switch context (if required)
kubectl config use-context kind-scheduling-lab
```

---

### 2️⃣ NodeSelector (Basic Scheduling)

NodeSelector schedules pods only on nodes with matching labels.

```bash
# Deploy application
kubectl apply -f 02-node-selector.yaml

# Check pod status (likely Pending initially)
kubectl get pods
```

➕ **Add label to worker node**

```bash
kubectl label node scheduling-lab-worker foo=bar
```

✅ **Verify scheduling**

```bash
kubectl get pods -o wide
```

💡 Alternative: edit node manually

```bash
kubectl edit node scheduling-lab-worker
```

Add:

```yaml
metadata:
  labels:
    foo: bar
```

---

### 3️⃣ Node Affinity (Preferred / Soft Rule)

Pods prefer a node but can still run elsewhere if not available.

```bash
kubectl apply -f 03-node-affinity-preferred.yaml
kubectl get pods -o wide
```

**Add preferred label**

```bash
kubectl label node scheduling-lab-worker foo=bar1
```

**Trigger rescheduling**

```bash
kubectl rollout restart deployment nginx-deployment
kubectl rollout status deployment nginx-deployment
```

**Verify placement**

```bash
kubectl get pods -o wide
```

---

### 4️⃣ Node Affinity (Required / Hard Rule)

Pods will NOT schedule unless conditions are met.

```bash
kubectl apply -f 04-node-affinity-required.yaml
kubectl get pods
```

**Debug pending pods**

```bash
kubectl describe pod <pod-name>
```

**Fix by labeling node**

```bash
kubectl label node scheduling-lab-worker foo=bar1
```

**Restart deployment**

```bash
kubectl rollout restart deployment nginx-deployment
kubectl rollout status deployment nginx-deployment
```

---

### 5️⃣ Taints & Tolerations (NoSchedule)

Taints repel pods unless they have matching tolerations.

**Step 1: Taint node**

```bash
kubectl taint nodes scheduling-lab-worker gpu=true:NoSchedule
```

**Step 2: Deploy application**

```bash
kubectl apply -f 05-no-schedule-taint-toleration.yaml
```

**Step 3: Restart pods**

```bash
kubectl rollout restart deployment nginx-deployment
kubectl rollout status deployment nginx-deployment
```

**Step 4: Verify scheduling**

```bash
kubectl get pods -o wide
```

**Step 5: Remove taint (cleanup)**

```bash
kubectl taint nodes scheduling-lab-worker gpu=true:NoSchedule-
```

---

## 🛠 Useful Debug Commands

```bash
# View node labels
kubectl get nodes --show-labels

# View taints
kubectl describe node <node-name> | grep Taints

# Debug pending pods
kubectl describe pod <pod-name>

# Remove node label
kubectl label node <node-name> foo-

# Remove taint
kubectl taint nodes <node-name> key=value:NoSchedule-

# Deployment rollout control
kubectl rollout restart deployment <name>
kubectl rollout status deployment <name>
kubectl rollout undo deployment <name>
```

---

## 🧹 Cleanup

```bash
kind delete cluster --name scheduling-lab
```
