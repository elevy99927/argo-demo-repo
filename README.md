# argo-demo-repo
## This is a Demo repository for ARGOCD tutorial.
## for the full content of this project, please refer to:
## [https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD](https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD)

---
## Example 3: Git Files Generator + Helm Multi-Source ApplicationSet

Branch: `example-3-helm-values`

This branch holds **only Helm values files**, no manifests. One Helm chart is deployed many
times, and every values file in Git becomes one Application. The ApplicationSet uses a
**git files generator** matching `systems/**/*-values.yaml` and a **multi-source** Application:
source 1 is the chart, source 2 is this repo, referenced as `$values`.

```
systems/<team>/<cluster>/<namespace>/<app>-values.yaml

└── systems
    ├── team-a
    │   ├── dev
    │   │   ├── frontend-ns
    │   │   │   ├── app-a-values.yaml
    │   │   │   └── app-b-values.yaml
    │   │   └── backend-ns
    │   │       └── app-c-values.yaml
    │   ├── qa
    │   │   ├── frontend-ns
    │   │   │   ├── app-a-values.yaml
    │   │   │   └── app-b-values.yaml
    │   │   └── backend-ns
    │   │       └── app-c-values.yaml
    │   └── prd
    │       ├── frontend-ns
    │       │   ├── app-a-values.yaml
    │       │   └── app-b-values.yaml
    │       └── backend-ns
    │           └── app-c-values.yaml
    └── team-b
        ├── dev
        │   └── payments-ns
        │       ├── app-d-values.yaml
        │       └── app-e-values.yaml
        ├── qa
        │   └── payments-ns
        │       ├── app-d-values.yaml
        │       └── app-e-values.yaml
        └── prd
            └── payments-ns
                ├── app-d-values.yaml
                └── app-e-values.yaml
```

Each values file is three lines:

```yaml
replicaCount: 1
ui:
  message: "app-a | team-a | dev | frontend-ns"
```

| Cluster | replicaCount |
|---------|--------------|
| dev | 1 |
| qa  | 2 |
| prd | 3 |

### ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: systems-helm
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
    - git:
        repoURL: https://github.com/elevy99927/argo-demo-repo.git
        revision: example-3-helm-values
        files:
          - path: "systems/*/*/*/*-values.yaml"
  # For every matched file:
  #   .path.path     = systems/team-a/dev/frontend-ns
  #   .path.segments = [systems, team-a, dev, frontend-ns]
  #   .path.filename = app-a-values.yaml
  template:
    metadata:
      # team-a-dev-frontend-ns-app-a
      name: '{{index .path.segments 1}}-{{index .path.segments 2}}-{{index .path.segments 3}}-{{.path.filename | trimSuffix "-values.yaml"}}'
      labels:
        team: '{{index .path.segments 1}}'
        cluster: '{{index .path.segments 2}}'
        app: '{{.path.filename | trimSuffix "-values.yaml"}}'
    spec:
      project: default
      sources:
        # 1. Chart from the Helm repo
        - repoURL: https://stefanprodan.github.io/podinfo
          chart: podinfo
          targetRevision: 6.5.0
          helm:
            releaseName: '{{.path.filename | trimSuffix "-values.yaml"}}'
            valueFiles:
              - $values/{{.path.path}}/{{.path.filename}}
        # 2. Values from the GitOps repo
        - repoURL: https://github.com/elevy99927/argo-demo-repo.git
          targetRevision: example-3-helm-values
          ref: values
      destination:
        # cluster folder name == cluster name registered in ArgoCD
        # (argocd cluster add <kube-context> --name dev)
        # single-cluster lab: replace with  server: https://kubernetes.default.svc
        name: '{{index .path.segments 2}}'
        namespace: '{{index .path.segments 3}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true```

Replace the `<helm-chart-repo>` / `<chart-name>` / `<chart-version>` placeholders with the chart
from the main tutorial repo. The namespace is the folder holding the values file, and the file's
own keys (`replicaCount`, `ui.message`) are also exposed to the template if you need them.

---
## Contact

For questions or feedback, feel free to reach out:

- **Email**: eyal@levys.co.il
- **GitHub**: [https://github.com/elevy99927](https://github.com/elevy99927)
