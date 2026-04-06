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
