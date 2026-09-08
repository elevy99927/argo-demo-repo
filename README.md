# argo-demo-repo
## This is a Demo repository for ARGOCD tutorial.
## for the full content of this project, please refer to:
## [https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD](https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD)

---
## Example 1: List Generator ApplicationSet

Branch: `example-1-appset`

One Deployment (`demo-app`) copied per environment. The ApplicationSet uses a **list generator**
with a fixed element per cluster, and each element points at its own folder.

```
project-1/
├── k8s-dev/demo-app.yaml
├── k8s-qa/demo-app.yaml
└── k8s-prd/demo-app.yaml
```

Every `demo-app.yaml` is the same Deployment: image `elevy99927/color:blue`, 1 replica, port 80.
Edit one folder to change one environment only.

### ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: demo-app
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - env: k8s-dev
      - env: k8s-qa
      - env: k8s-prd
  template:
    metadata:
      name: 'demo-app-{{env}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/elevy99927/argo-demo-repo.git
        targetRevision: example-1-appset
        path: 'project-1/{{env}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{env}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
        - CreateNamespace=true
```

Adding a new environment means adding a folder **and** a list element.
Example 2 removes that second step.

---
## Contact

For questions or feedback, feel free to reach out:

- **Email**: eyal@levys.co.il
- **GitHub**: [https://github.com/elevy99927](https://github.com/elevy99927)
