# infra-manifests

This repository is the source of truth for Kubernetes deployments.

Managed via Argo CD.

Structure:
- base/ → reusable resources
- overlays/dev → environment-specific config
- argocd/ → Argo CD applications
