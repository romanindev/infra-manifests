# infra-manifests

This repository is the source of truth for Kubernetes deployments.

Managed via Argo CD.

## 🎯 Purpose

This repository is **not a production service**.

It is a **learning-oriented project** designed to explore:
- GitOps workflows with Argo CD
- Containerized applications with Docker
- Kubernetes deployments and networking
- Service-to-database communication (MongoDB)

---

Structure:
- base/ → reusable resources
- overlays/dev → environment-specific config
- argocd/ → Argo CD applications

## Architecture (context)

This API is part of a multi-repo setup:

- `todo-api` → this repository (backend)
- `todo-ui` → frontend (planned)
- `infra-manifests` → Kubernetes manifests (GitOps source of truth)

Argo CD watches the `infra-manifests` repo and deploys this service into the cluster.

---


## Local Development

This section explains how to fully recreate the local environment from scratch.

### Step 1 — create a cluster

Run:
```bash
k3d cluster create gitops-demo \
--agents 1 \
-p "8080:80@loadbalancer" \
-p "8443:443@loadbalancer"
```

Check the cluster:
```bash
kubectl get nodes
```

Expected output:

```text
NAME                       STATUS   ROLES                  AGE   VERSION
k3d-gitops-demo-server-0   Ready    control-plane,master   ...
k3d-gitops-demo-agent-0    Ready    <none>                 ...
```


### Step 2 — Bootstrap namespace for Argo CD
```bash
kubectl create namespace argocd
```

> Note: Application namespaces (e.g. todo-dev) are managed via Git (Kubernetes manifests), not created manually.

Check:
```bash
kubectl get ns
```

### Step 3 - Install Argo CD

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait until all pods are running:
```bash
kubectl get pods -n argocd
```

Expected:
```text
STATUS: Running
```

### Step 4 — Access Argo CD

Get admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d && echo
```

Start port-forward:
```bash
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Open in browser:
```text
https://localhost:8081
```

### Step 5 — Apply Argo CD Application

Apply the GitOps application:
```bash
kubectl apply -f argocd/todo-dev-application.yaml
```

After this, Argo CD will:

- create the todo-dev namespace
- deploy MongoDB
- deploy todo-api
- deploy todo-ui
- configure Ingress

### Step 6 — Import local Docker images (k3d only)

If using locally built images:
```bash
k3d image import todo-api:local -c gitops-demo
k3d image import todo-ui:local -c gitops-demo
```

Then restart deployments if needed:
```bash
kubectl rollout restart deployment/todo-api -n todo-dev
kubectl rollout restart deployment/todo-ui -n todo-dev
```

### Step 7 — Verify
```bash
kubectl get all -n todo-dev
kubectl get ingress -n todo-dev
```

Open:
```text
http://demo.local:8080
```

### Notes

> - Do not create application resources manually
> - All infrastructure (except Argo CD bootstrap) is managed via Git
> - If the cluster is deleted, this process fully restores the system
