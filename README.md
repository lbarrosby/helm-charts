# helm-charts

Production-ready Helm charts I use as a baseline for platform teams. Each chart ships with sane security and availability defaults, a JSON schema for its values, unit tests and manifest validation in CI.

| Chart | Description |
|---|---|
| [app](charts/app) | Stateless HTTP services: Gateway API or Ingress, HPA or KEDA, PDB, topology spread, NetworkPolicy, ServiceMonitor |

## Usage

```console
helm repo add lbarrosby https://lbarrosby.github.io/helm-charts
helm repo update
helm search repo lbarrosby
```

## How changes are validated

Every pull request runs:

- `helm lint --strict` and `ct lint` with the scenarios under `charts/*/ci`
- [helm-unittest](https://github.com/helm-unittest/helm-unittest) suites under `charts/*/tests`
- [kubeconform](https://github.com/yannh/kubeconform) against the Kubernetes schemas and the CRD catalog (Gateway API, KEDA, Prometheus Operator)
- a docs check that fails when `README.md` is out of date with `values.yaml`

Merges to `main` publish new chart versions through [chart-releaser](https://github.com/helm/chart-releaser).

## Conventions

- Charts follow [SemVer](https://semver.org). Any change to a chart bumps its `version`.
- Values are documented inline with `# --` comments and rendered with [helm-docs](https://github.com/norwoodj/helm-docs).
- Labels follow the `app.kubernetes.io/*` recommended set.
- Optional integrations stay off by default and never require CRDs to be installed.

## License

Apache 2.0
