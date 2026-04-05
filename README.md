# data-platform-manifests
Helm values and ArgoCD Application resources for the data platform. Managed by ArgoCD via app-of-apps pattern.

```
data-platform-manifests/
│
├── README.md
├── docs/
│   ├── kubernetes_upgrading.md
│   └── server_hardening.md
│
└── apps/
    ├── argocd/
    │   ├── argocd-values.yaml
    │   ├── argocd-application.yaml
    │   ├── data-platform-manifests-app.yaml
    │   └── data-platform-configs-app.yaml
    │
    ├── vikunja/
    │   ├── vikunja-application.yaml
    │   ├── vikunja-values.yaml
    │   └── vikunja-work-values.yaml
    │
    └── skyhook-radar/
        ├── radar-application.yaml
        └── radar-values.yaml
```
