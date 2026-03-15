# grafana-stack

> Helm umbrella chart for a **Grafana LGTM observability stack** on Kubernetes.

| Component | Sub-chart | Default |
|-----------|-----------|---------|
| [Grafana](https://grafana.com/grafana/) | `grafana/grafana` | enabled |
| [Loki](https://grafana.com/oss/loki/) | `grafana/loki` | enabled |
| [Tempo](https://grafana.com/oss/tempo/) | `grafana/tempo-distributed` | enabled |
| [Mimir](https://grafana.com/oss/mimir/) | `grafana/mimir-distributed` | enabled |
| [Alloy](https://grafana.com/oss/alloy/) | `grafana/alloy` | enabled |
| [Beyla](https://grafana.com/oss/beyla/) | `grafana/beyla` | disabled |

## Design Goals

- Simple public Helm chart repository — no private registry required
- **No** Prometheus Operator dependency
- **No** `ServiceMonitor` / `PodMonitor` requirement
- Metrics scraped by **Grafana Alloy**
- Traces sent via **OpenTelemetry → Alloy → Tempo**
- Logs collected via **Alloy → Loki**
- Metrics stored in **Mimir**
- Object storage via **S3 / MinIO / Ceph RGW**

---

## Installation

### From the OCI registry (recommended)

```bash
helm install grafana-stack \
  oci://ghcr.io/chiliChonka/charts/grafana-stack \
  --version 0.1.0 \
  -n observability \
  --create-namespace
```

### From source

```bash
git clone https://github.com/ChiliChonka/grafana-stack-helm-chart.git
cd grafana-stack-helm-chart

helm dependency update grafana-stack

helm upgrade --install grafana-stack grafana-stack \
  -n observability \
  --create-namespace
```

### Using a custom values file

```bash
helm upgrade --install grafana-stack \
  oci://ghcr.io/chiliChonka/charts/grafana-stack \
  --version 0.1.0 \
  -n observability \
  --create-namespace \
  --values my-values.yaml
```

---

## Configuration

All values below are in addition to the upstream sub-chart values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.clusterName` | Human-readable cluster name shown in dashboards | `home-cluster` |
| `grafana.enabled` | Deploy Grafana | `true` |
| `loki.enabled` | Deploy Loki | `true` |
| `tempo.enabled` | Deploy Tempo (distributed) | `true` |
| `mimir.enabled` | Deploy Mimir (distributed) | `true` |
| `alloy.enabled` | Deploy Grafana Alloy | `true` |
| `alloy.controller.type` | Alloy controller type (`deployment`/`daemonset`/`statefulset`) | `deployment` |
| `alloy.controller.replicas` | Number of Alloy replicas | `2` |
| `beyla.enabled` | Deploy Beyla (eBPF auto-instrumentation) | `false` |

Each component passes all of its values through to the respective upstream sub-chart.
Consult the upstream chart documentation for a full list of options:

- [grafana/grafana](https://github.com/grafana/helm-charts/tree/main/charts/grafana)
- [grafana/loki](https://github.com/grafana/loki/tree/main/production/helm/loki)
- [grafana/tempo-distributed](https://github.com/grafana/helm-charts/tree/main/charts/tempo-distributed)
- [grafana/mimir-distributed](https://github.com/grafana/mimir/tree/main/operations/helm/charts/mimir-distributed)
- [grafana/alloy](https://github.com/grafana/alloy/tree/main/operations/helm/charts/alloy)
- [grafana/beyla](https://github.com/grafana/beyla/tree/main/charts/beyla)

---

## Access Grafana

After installation, retrieve the admin password and forward the port:

```bash
# Get admin password
kubectl get secret --namespace observability \
  grafana-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode; echo

# Port-forward
kubectl port-forward --namespace observability svc/grafana-stack-grafana 3000:80
```

Open <http://localhost:3000> and log in with `admin` and the password above.

---

## Uninstallation

```bash
helm uninstall grafana-stack -n observability
```

> **Note:** PersistentVolumeClaims are **not** deleted automatically. Remove them manually if no longer needed.

---

## License

[MIT](../../LICENSE)
