# app

![Version: 1.0.1](https://img.shields.io/badge/Version-1.0.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.0.0](https://img.shields.io/badge/AppVersion-1.0.0-informational?style=flat-square)

Opinionated chart for stateless HTTP services, secure and highly available by default.

The defaults are the ones a platform team would sign off for production: non-root with a read-only filesystem, every capability dropped, no API token mounted, pods spread across zones and nodes, a PodDisruptionBudget, a preStop delay so the load balancer drains before the process stops, and a rollout that never goes below the desired replica count.

## Install

```console
helm repo add lbarrosby https://lbarrosby.github.io/helm-charts
helm upgrade --install my-api lbarrosby/app \
  --set image.repository=ghcr.io/acme/my-api \
  --set image.tag=1.4.2
```

## Common setups

Expose through the Gateway API and autoscale on CPU and memory:

```yaml
httpRoute:
  enabled: true
  parentRefs:
    - name: public
      namespace: gateway-system
      sectionName: https
  hostnames: [api.acme.com]
autoscaling:
  enabled: true
  maxReplicas: 20
  targetMemoryUtilizationPercentage: 80
```

Scale on a Prometheus query with KEDA and lock down the network:

```yaml
keda:
  enabled: true
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring:9090
        query: sum(rate(http_requests_total{service="my-api"}[1m]))
        threshold: "50"
networkPolicy:
  enabled: true
  ingressFrom:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: gateway-system
```

## Guard rails

The chart fails fast instead of rendering something that will misbehave in the cluster:

- `autoscaling` and `keda` cannot be enabled together, since KEDA manages its own HPA.
- `keda.enabled` needs at least one trigger.
- `podDisruptionBudget` takes either `minAvailable` or `maxUnavailable`.
- `values.schema.json` rejects missing resource requests, invalid ports and malformed digests.

## Requirements

Kubernetes: `>=1.30.0-0`

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` | Affinity rules. |
| autoscaling.behavior | object | `{"scaleDown":{"stabilizationWindowSeconds":300}}` | Scaling behavior. |
| autoscaling.enabled | bool | `false` | Enable a HorizontalPodAutoscaler. Mutually exclusive with keda.enabled. |
| autoscaling.maxReplicas | int | `10` |  |
| autoscaling.minReplicas | int | `2` |  |
| autoscaling.targetCPUUtilizationPercentage | int | `70` |  |
| autoscaling.targetMemoryUtilizationPercentage | string | `""` | Target memory utilization. Disabled when empty. |
| commonLabels | object | `{}` | Extra labels added to every resource. |
| config | object | `{}` | Key/value pairs rendered into a ConfigMap and loaded as environment variables. |
| env | list | `[]` | Extra environment variables. |
| envFrom | list | `[]` | Environment from ConfigMaps or Secrets. |
| extraVolumeMounts | list | `[]` | Extra volume mounts for the main container. |
| extraVolumes | list | `[]` | Extra volumes. |
| fullnameOverride | string | `""` | Overrides the fully qualified resource name. |
| httpRoute.annotations | object | `{}` |  |
| httpRoute.enabled | bool | `false` | Create a Gateway API HTTPRoute. |
| httpRoute.hostnames[0] | string | `"app.example.com"` |  |
| httpRoute.parentRefs | list | `[{"name":"public","namespace":"gateway-system","sectionName":"https"}]` | Gateways this route attaches to. |
| httpRoute.rules | list | `[]` | Route rules. When empty a single rule sends every path to the Service. |
| image.digest | string | `""` | Image digest. When set it takes precedence over the tag. |
| image.pullPolicy | string | `"IfNotPresent"` | Image pull policy. |
| image.repository | string | `"ghcr.io/stefanprodan/podinfo"` | Container image repository. |
| image.tag | string | `""` | Image tag. Defaults to the chart appVersion. |
| imagePullSecrets | list | `[]` | Secrets used to pull the image. |
| ingress.annotations | object | `{}` |  |
| ingress.className | string | `""` |  |
| ingress.enabled | bool | `false` | Create an Ingress. |
| ingress.hosts[0].host | string | `"app.example.com"` |  |
| ingress.hosts[0].paths[0].path | string | `"/"` |  |
| ingress.hosts[0].paths[0].pathType | string | `"Prefix"` |  |
| ingress.tls | list | `[]` |  |
| initContainers | list | `[]` | Init containers. |
| keda.cooldownPeriod | int | `300` |  |
| keda.enabled | bool | `false` | Enable a KEDA ScaledObject. Mutually exclusive with autoscaling.enabled. |
| keda.maxReplicaCount | int | `20` |  |
| keda.minReplicaCount | int | `2` |  |
| keda.pollingInterval | int | `30` |  |
| keda.triggers | list | `[]` | KEDA triggers, passed as is. |
| lifecycle.preStopSleepSeconds | int | `5` | Sleep before shutdown so load balancers stop sending traffic first. |
| nameOverride | string | `""` | Overrides the chart name used in resource names and labels. |
| networkPolicy.egressTo | list | `[]` | Allow egress only to these destinations plus DNS. Leave empty to allow all egress. |
| networkPolicy.enabled | bool | `false` | Create a NetworkPolicy that only allows the listed sources. |
| networkPolicy.ingressFrom | list | `[]` | Allowed ingress sources, as NetworkPolicyPeer entries. |
| nodeSelector | object | `{}` | Node selector. |
| podAnnotations | object | `{}` | Extra annotations added to the pods. |
| podDisruptionBudget.enabled | bool | `true` | Create a PodDisruptionBudget. |
| podDisruptionBudget.maxUnavailable | int | `1` | Maximum unavailable pods. |
| podDisruptionBudget.minAvailable | string | `""` | Minimum available pods. Ignored when maxUnavailable is set. |
| podLabels | object | `{}` | Extra labels added to the pods. |
| podSecurityContext | object | `{"fsGroup":10001,"runAsGroup":10001,"runAsNonRoot":true,"runAsUser":10001,"seccompProfile":{"type":"RuntimeDefault"}}` | Pod level security context. |
| ports | list | `[{"containerPort":9898,"name":"http","protocol":"TCP"}]` | Named container ports. The first one is the default target of the Service. |
| priorityClassName | string | `""` | Priority class name. |
| probes.liveness.enabled | bool | `true` |  |
| probes.liveness.failureThreshold | int | `3` |  |
| probes.liveness.httpGet.path | string | `"/healthz"` |  |
| probes.liveness.httpGet.port | string | `"http"` |  |
| probes.liveness.periodSeconds | int | `10` |  |
| probes.readiness.enabled | bool | `true` |  |
| probes.readiness.failureThreshold | int | `3` |  |
| probes.readiness.httpGet.path | string | `"/readyz"` |  |
| probes.readiness.httpGet.port | string | `"http"` |  |
| probes.readiness.periodSeconds | int | `5` |  |
| probes.startup.enabled | bool | `true` |  |
| probes.startup.failureThreshold | int | `30` |  |
| probes.startup.httpGet.path | string | `"/healthz"` |  |
| probes.startup.httpGet.port | string | `"http"` |  |
| probes.startup.periodSeconds | int | `5` |  |
| replicaCount | int | `2` | Number of replicas when no autoscaler is enabled. |
| resources | object | `{"limits":{"memory":"256Mi"},"requests":{"cpu":"100m","memory":"128Mi"}}` | CPU and memory. Requests are required so the scheduler and the autoscaler work. |
| revisionHistoryLimit | int | `5` | Revisions kept for rollback. |
| secretEnv | object | `{}` | Key/value pairs rendered into a Secret and loaded as environment variables. Prefer an external secret manager in production. |
| securityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]},"readOnlyRootFilesystem":true}` | Container level security context. |
| service.annotations | object | `{}` | Service annotations. |
| service.enabled | bool | `true` | Create a Service. |
| service.port | int | `80` | Service port. |
| service.targetPort | string | `"http"` | Container port name the Service targets. |
| service.type | string | `"ClusterIP"` | Service type. |
| serviceAccount.annotations | object | `{}` | Annotations, for example eks.amazonaws.com/role-arn for IRSA. |
| serviceAccount.automountServiceAccountToken | bool | `false` | Mount the API token only when the workload talks to the API server. |
| serviceAccount.create | bool | `true` | Create a dedicated ServiceAccount. |
| serviceAccount.name | string | `""` | Name of the ServiceAccount. Generated when empty. |
| serviceMonitor.enabled | bool | `false` | Create a Prometheus Operator ServiceMonitor. |
| serviceMonitor.interval | string | `"30s"` |  |
| serviceMonitor.labels | object | `{}` |  |
| serviceMonitor.path | string | `"/metrics"` |  |
| serviceMonitor.port | string | `"http"` |  |
| serviceMonitor.scrapeTimeout | string | `"10s"` |  |
| sidecars | list | `[]` | Sidecar containers. |
| strategy.rollingUpdate.maxSurge | string | `"25%"` |  |
| strategy.rollingUpdate.maxUnavailable | int | `0` |  |
| strategy.type | string | `"RollingUpdate"` |  |
| terminationGracePeriodSeconds | int | `30` | Seconds to wait before SIGKILL. Keep it above the preStop sleep. |
| tmpVolume | object | `{"enabled":true,"sizeLimit":"64Mi"}` | Writable emptyDir mounted at /tmp, needed with a read-only root filesystem. |
| tolerations | list | `[]` | Tolerations. |
| topologySpread.enabled | bool | `true` | Spread pods across zones and nodes. |
| topologySpread.maxSkew | int | `1` |  |
| topologySpread.whenUnsatisfiable | string | `"ScheduleAnyway"` | ScheduleAnyway keeps the rollout going when a zone is short on capacity. |
