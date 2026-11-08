# argo-demo-repo
## This is a Demo repository for ARGOCD tutorial.
## for the full content of this project, please refer to:
## [https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD](https://github.com/elevy99927/Jenkins-k8s/tree/main/Part4-CICD/04-ArgoCD)

---
## Example 4: Real-Life Organisation (Git Files Generator + Helm Multi-Source)

Branch: `example-4-real-live-example`

Same mechanics as Example 3: one Helm chart, one values file per app, and every values file in Git
becomes one Application. The difference is that the folders follow a real organisation:
five business units, each owning its own apps and namespaces.

| Business unit | Namespace(s) | Apps |
|---------------|--------------|------|
| `hr` | `hr` | `payroll`, `recruiting` |
| `finance` | `finance` | `invoicing`, `ledger` |
| `legal` | `legal` | `contracts` |
| `digital-ui` | `web` | `storefront`, `customer-portal` |
| `digital-backend` | `api`, `payments` | `orders-api`, `customers-api`, `payments-api` |

```
systems/<business-unit>/<cluster>/<namespace>/<app>-values.yaml

└── systems
    ├── hr
    │   └── dev | qa | prd
    │       └── hr
    │           ├── payroll-values.yaml
    │           └── recruiting-values.yaml
    ├── finance
    │   └── dev | qa | prd
    │       └── finance
    │           ├── invoicing-values.yaml
    │           └── ledger-values.yaml
    ├── legal
    │   └── dev | qa | prd
    │       └── legal
    │           └── contracts-values.yaml
    ├── digital-ui
    │   └── dev | qa | prd
    │       └── web
    │           ├── storefront-values.yaml
    │           └── customer-portal-values.yaml
    ├── digital-backend
    │   └── dev | qa | prd
    │       ├── api
    │       │   ├── orders-api-values.yaml
    │       │   └── customers-api-values.yaml
    │       └── payments
    │           └── payments-api-values.yaml
    └── devops
        └── dev | qa | prd
            └── ingress-nginx
                └── ingress.yaml
```

10 apps × 3 clusters = 30 Applications from one ApplicationSet. Each values file is four lines;
`ui.color` is the business unit's color, so each podinfo UI shows which unit it belongs to:

```yaml
replicaCount: 1
ui:
  color: "#2e7d32"
  message: "payroll | hr | dev | hr"
```

| Cluster | replicaCount |
|---------|--------------|
| dev | 1 |
| qa  | 2 |
| prd | 3 |

### ApplicationSets

- [ApplicationSet/systems-helm.yaml](ApplicationSet/systems-helm.yaml): business-unit apps (podinfo chart).
  Generates `hr-qa-hr-payroll`, `digital-backend-qa-payments-payments-api`, ...
- [ApplicationSet/infra-helm.yaml](ApplicationSet/infra-helm.yaml): DevOps infra (ingress-nginx chart).

Both match only `qa` (`systems/*/qa/*/*-values.yaml`) because the lab runs on one kind cluster.
For all clusters, change the path to `systems/*/*/*/*-values.yaml` and the destination to
`name: '{{index .path.segments 2}}'` (cluster folder name == cluster name registered in ArgoCD).

Onboarding a new app is one file: add `systems/<unit>/<cluster>/<namespace>/<app>-values.yaml`,
push, and ArgoCD creates the Application. Deleting the file removes it.

---
## Contact

For questions or feedback, feel free to reach out:

- **Email**: eyal@levys.co.il
- **GitHub**: [https://github.com/elevy99927](https://github.com/elevy99927)
