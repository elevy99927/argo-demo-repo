# argo-demo-repo
## This is a Demo repository for ARGOCD tutorial.
## for the full content of this project, please refer to:
## [https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD](https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD)

---
## Example 2: Git Directory Generator ApplicationSet

Branch: `example-2-dynamic-generator`

Folders are discovered from Git, so adding a team or cluster is just adding a directory.
The ApplicationSet uses a **git directory generator** on `systems/*/*` and reads
the team and cluster names from the path.

```
systems/
├── team-a/
│   ├── k8s-dev/  application-a.yaml  application-b.yaml
│   ├── k8s-qa/   application-a.yaml  application-b.yaml
│   └── k8s-prd/  application-a.yaml  application-b.yaml
└── team-b/
    ├── k8s-dev/  application-c.yaml  application-d.yaml
    ├── k8s-qa/   application-c.yaml  application-d.yaml
    └── k8s-prd/  application-c.yaml  application-d.yaml
```

| Team   | App           | Image                       |
|--------|---------------|-----------------------------|
| team-a | application-a | `elevy99927/color:blue`     |
| team-a | application-b | `elevy99927/color:yellow`   |
| team-b | application-c | `elevy99927/color:red`      |
| team-b | application-d | `elevy99927/color:blue`     |

Each `application-*.yaml` is a full app in one file: ServiceAccount, ConfigMap, Secret,
Deployment, Service, and Ingress. The Deployment runs as its ServiceAccount and loads the
ConfigMap and Secret via `envFrom`. The Ingress host is `<app>.<team>.<cluster>.local`.

Each folder becomes one Argo CD Application named `<team>-<cluster>`, deployed into a
namespace of the same name.

### ApplicationSet

The ApplicationSet lives in [ApplicationSet/systems-dynamic.yaml](ApplicationSet/systems-dynamic.yaml).

```bash
kubectl apply -f ApplicationSet/systems-dynamic.yaml
```

`.path.segments` for `systems/team-a/k8s-dev` is `[systems, team-a, k8s-dev]`,
so index 1 is the team and `.path.basename` is the cluster.

---
## Contact

For questions or feedback, feel free to reach out:

- **Email**: eyal@levys.co.il
- **GitHub**: [https://github.com/elevy99927](https://github.com/elevy99927)
