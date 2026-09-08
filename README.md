# argo-demo-repo
## This is a Demo repository for ARGOCD tutorial.
## for the full content of this project, please refer to:
## [https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD](https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD)

---
## Example 2: Git Directory Generator ApplicationSet

Branch: `example-2-dynamic-generator`

Apps are discovered from Git, so adding a team, environment, or app is just adding a
folder or a file. The ApplicationSet uses a **git files generator** on
`systems/*/qa/*.yaml` and reads team, environment, and app name from the file path.
The environment is pinned per ApplicationSet, so one ApplicationSet serves one environment.

```
systems/<team>/<env>/<app>.yaml

systems/
├── team-a/
│   ├── dev/  application-a.yaml  application-b.yaml  application-c.yaml
│   ├── qa/   application-a.yaml  application-b.yaml  application-c.yaml
│   └── prd/  application-a.yaml  application-b.yaml  application-c.yaml
└── team-b/
    ├── dev/  application-d.yaml  application-e.yaml
    ├── qa/   application-d.yaml  application-e.yaml
    └── prd/  application-d.yaml  application-e.yaml
```

| Team   | App           | Image                       |
|--------|---------------|-----------------------------|
| team-a | application-a | `elevy99927/color:blue`     |
| team-a | application-b | `elevy99927/color:yellow`   |
| team-a | application-c | `elevy99927/color:red`      |
| team-b | application-d | `elevy99927/color:blue`     |
| team-b | application-e | `elevy99927/color:green`    |

Each `<app>.yaml` is a full app in one file: ServiceAccount, ConfigMap, Secret,
Deployment, Service, and Ingress. The Deployment runs as its ServiceAccount and loads the
ConfigMap and Secret via `envFrom`. The Ingress host is `<app>.<team>.<env>.local`.

Each file becomes one Argo CD Application named `<team>-<env>-<app>`
(for example `team-a-qa-application-a`), deployed into namespace `<team>-<env>`.
The source uses `directory.include` so each Application syncs only its own file.

### ApplicationSet

The ApplicationSet lives in [ApplicationSet/systems-dynamic.yaml](ApplicationSet/systems-dynamic.yaml).

```bash
kubectl apply -f ApplicationSet/systems-dynamic.yaml
```

For `systems/team-a/qa/application-a.yaml` the files generator gives `.path.path` =
`systems/team-a/qa`, `.path.segments` = `[systems, team-a, qa]`, `.path.basename` = `qa`,
and `.path.filename` = `application-a.yaml`. So index 1 is the team, basename is the
environment, and the filename minus `.yaml` is the app.

---
## Contact

For questions or feedback, feel free to reach out:

- **Email**: eyal@levys.co.il
- **GitHub**: [https://github.com/elevy99927](https://github.com/elevy99927)
