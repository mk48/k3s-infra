# k3s-infra

A GitOps infrastructure repository for managing applications on a [k3s](https://k3s.io/) Kubernetes cluster using [ArgoCD](https://argo-cd.readthedocs.io/).

## Overview

This repo uses the **App-of-Apps** pattern - a single root ArgoCD Application watches the `argocd/apps/` directory and automatically deploys any child Applications it finds there. Each child Application then syncs its own Kubernetes manifests from the `projects/` directory.

```
argocd/root-app.yaml          ← apply this once to bootstrap everything
    └── watches argocd/apps/
            └── simple-rest-api.yaml  → syncs projects/simple-rest-api/
```

All syncs are automated with self-heal and prune enabled, so the cluster continuously reconciles to match the state in Git.

## Repository Structure

```
k3s-infra/
├── argocd/
│   ├── root-app.yaml           # Bootstrap - apply this manually once
│   ├── ingress.yaml            # Exposes ArgoCD UI at argocd.local
│   └── apps/
│       └── simple-rest-api.yaml
└── projects/
    └── simple-rest-api/
        ├── deployment.yaml
        ├── service.yaml
        └── ingress.yaml
```

## Getting Started

### Prerequisites

- k3s cluster running with Traefik as the ingress controller
- ArgoCD installed in the `argocd` namespace
- Local DNS resolving `*.local` domains to the cluster

### Bootstrap

Apply the root Application once to kick off the GitOps loop:

```bash
kubectl apply -f argocd/root-app.yaml
```

ArgoCD will pick up everything else automatically from there.

## Adding a New Application

1. Create a new directory under `projects/` with your Kubernetes manifests.
2. Add a new Application manifest to `argocd/apps/` pointing at that directory.
3. Push to `main` - ArgoCD will detect and deploy it automatically.

## Applications

### simple-rest-api

A sample HTTP API deployed in the `pi` namespace.

| Resource | Details                           |
| -------- | --------------------------------- |
| Image    | `ghcr.io/mk48/simple-http:latest` |
| Replicas | 2                                 |
| URL      | `http://hello.local`              |

## Stack

| Tool                                      | Role                                  |
| ----------------------------------------- | ------------------------------------- |
| [k3s](https://k3s.io/)                    | Lightweight Kubernetes distribution   |
| [ArgoCD](https://argo-cd.readthedocs.io/) | GitOps continuous delivery            |
| [Traefik](https://traefik.io/)            | Ingress controller (bundled with k3s) |
| [ghcr.io](https://ghcr.io)                | Container registry                    |
