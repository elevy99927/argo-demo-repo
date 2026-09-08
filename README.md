# argo-demo-repo
## This is a Demo repository for ARGOCD tutorial.
## for the full content of this project, please refer to:
## [https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD](https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD)

---
## Example 2: Git Directory Generator ApplicationSet

Branch: `example-2-dynamic-generator`

Folders are discovered from Git, so adding a team, environment, namespace, or app is just
adding a directory. The ApplicationSet uses a **git directory generator** on
`systems/*/qa/*/*` and reads team, environment, namespace, and app name from the path.
The environment is pinned per ApplicationSet, so one ApplicationSet serves one environment.

```
systems/<team>/<env>/<ns>/<app>/<app>.yaml

systems/
├── team-a/
│   ├── dev/frontend-ns/  application-a/  application-b/
│   ├── qa/frontend-ns/   application-a/  application-b/
│   └── prd/frontend-ns/  application-a/  application-b/
└── team-b/
    ├── dev/payments-ns/  application-c/  application-d/
    ├── qa/payments-ns/   application-c/  application-d/
    └── prd/payments-ns/  application-c/  application-d/
```

| Team   | Namespace   | App           | Image                       |
|--------|-------------|---------------|-----------------------------|
| team-a | frontend-ns | application-a | `elevy99927/color:blue`     |
| team-a | frontend-ns | application-b | `elevy99927/color:yellow`   |
| team-b | payments-ns | application-c | `elevy99927/color:red`      |
| team-b | payments-ns | application-d | `elevy99927/color:blue`     |

Each `<app>.yaml` is a full app in one file: ServiceAccount, ConfigMap, Secret,
Deployment, Service, and Ingress. The Deployment runs as its ServiceAccount and loads the
ConfigMap and Secret via `envFrom`. The Ingress host is `<app>.<ns>.<team>.<env>.local`.

Each app folder becomes one Argo CD Application named `<team>-<env>-<ns>-<app>`
(for example `team-a-qa-frontend-ns-application-a`), deployed into `<ns>`.

### ApplicationSet

The ApplicationSet lives in [ApplicationSet/systems-dynamic.yaml](ApplicationSet/systems-dynamic.yaml).

```bash
kubectl apply -f ApplicationSet/systems-dynamic.yaml
```

`.path.segments` for `systems/team-a/qa/frontend-ns/application-a` is
`[systems, team-a, qa, frontend-ns, application-a]`, so index 1 is the team,
index 2 the environment, index 3 the namespace, and `.path.basename` is the app.

---
## Contact

For questions or feedback, feel free to reach out:

- **Email**: eyal@levys.co.il
- **GitHub**: [https://github.com/elevy99927](https://github.com/elevy99927)
