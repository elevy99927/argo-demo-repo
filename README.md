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

systems/
├── team-a/
│   ├── k8s-dev/  frontend-ns/{application-a,application-b}-values.yaml  backend-ns/application-c-values.yaml
│   ├── k8s-qa/   frontend-ns/{application-a,application-b}-values.yaml  backend-ns/application-c-values.yaml
│   └── k8s-prd/  frontend-ns/{application-a,application-b}-values.yaml  backend-ns/application-c-values.yaml
└── team-b/
    ├── k8s-dev/  payments-ns/{application-d,application-e}-values.yaml
    ├── k8s-qa/   payments-ns/{application-d,application-e}-values.yaml
    └── k8s-prd/  payments-ns/{application-d,application-e}-values.yaml
```

Each values file is three lines:

```yaml
replicaCount: 1
ui:
  message: "application-a | team-a | k8s-dev | frontend-ns"
```

| Cluster | replicaCount |
|---------|--------------|
| k8s-dev | 1 |
| k8s-qa  | 2 |
| k8s-prd | 3 |

### ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: helm-apps
  namespace: argocd
spec:
  goTemplate: true
  generators:
  - git:
      repoURL: https://github.com/elevy99927/argo-demo-repo.git
      revision: example-3-helm-values
      files:
      - path: 'systems/**/*-values.yaml'
  template:
    metadata:
      # systems/team-a/k8s-dev/frontend-ns/application-a-values.yaml
      name: '{{ index .path.segments 1 }}-{{ index .path.segments 2 }}-{{ .path.filenameNormalized | trimSuffix "-values.yaml" }}'
    spec:
      project: default
      sources:
      - repoURL: <helm-chart-repo>
        chart: <chart-name>
        targetRevision: <chart-version>
        helm:
          valueFiles:
          - '$values/{{ .path.path }}/{{ .path.filename }}'
      - repoURL: https://github.com/elevy99927/argo-demo-repo.git
        targetRevision: example-3-helm-values
        ref: values
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{ .path.basename }}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
        - CreateNamespace=true
```

Replace the `<helm-chart-repo>` / `<chart-name>` / `<chart-version>` placeholders with the chart
from the main tutorial repo. The namespace is the folder holding the values file, and the file's
own keys (`replicaCount`, `ui.message`) are also exposed to the template if you need them.

---
## Contact

For questions or feedback, feel free to reach out:

- **Email**: eyal@levys.co.il
- **GitHub**: [https://github.com/elevy99927](https://github.com/elevy99927)
