# data-platform-manifests docs

Reference for the Kubernetes manifest structure in this repo.

## Repo layout

```
data-platform-manifests/
├── <component>/
│   ├── base/              # Kustomize base
│   └── overlays/
│       └── <env>/         # Environment-specific patches
└── <helm-component>/
    └── values.yaml        # Helm values (managed by ArgoCD)
```

## Components

<!-- TODO: list components and their manifest locations once stable -->

| Component | Type | Location |
|---|---|---|
| Postgres (landing) | Kustomize | <!-- TODO --> |
| ClickHouse | Kustomize | <!-- TODO --> |
| Airflow metadata Postgres | Kustomize | <!-- TODO --> |
| Airflow | Helm | <!-- TODO --> |
| Spark Operator | Helm | <!-- TODO --> |

## Deployment

All changes are deployed via ArgoCD. Push to main — ArgoCD reconciles automatically.
Do not run `kubectl apply` directly in production.

See [ADR-003](https://github.com/yasadanavira/data-platform-docs/blob/main/adr/adr-003-argocd-gitops-deployment.md) for the GitOps decision.

## Adding a new component

<!-- TODO -->

## Hello World

hello world
