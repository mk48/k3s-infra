# k3s-infra

GitOps infrastructure for a [k3s](https://k3s.io/) cluster, managed with [ArgoCD](https://argo-cd.readthedocs.io/), Kustomize, and Helm.

## Structure

```
argocd/               # ArgoCD root app + Helm chart that generates child apps per environment
projects/
  simple-rest-api/
    common/           # Shared base manifests (deployment, service, ingress)
    stage/            # Stage overlay - 1 replica, hello-stage.local, pi-stage namespace
    prod/             # Prod overlay  - 2 replicas, hello.local, pi-prod namespace
```

## How it works

1. `argocd/root-app.yaml` - a single root ArgoCD Application that watches `argocd/apps/`
2. `argocd/apps/` - a Helm chart that renders one ArgoCD Application per environment (`stage`, `prod`)
3. Each environment app points ArgoCD at the matching Kustomize overlay in `projects/<app>/<env>/`
4. Kustomize overlays patch the base: namespace, ingress host, replica count, image tag, and ConfigMap env vars

## Environments

| Env   | Namespace | Host              | Replicas |
| ----- | --------- | ----------------- | -------- |
| stage | pi-stage  | hello-stage.local | 1        |
| prod  | pi-prod   | hello.local       | 2        |

## Adding a new app

1. Create `projects/<app>/common/` with base manifests and a `kustomization.yaml`
2. Create `projects/<app>/stage/` and `projects/<app>/prod/` overlays
3. Add a template to `argocd/apps/templates/` following the pattern in `simple-rest-api.yaml`
