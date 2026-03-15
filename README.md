# grafana-stack-helm-chart

[![Helm Lint](https://github.com/ChiliChonka/grafana-stack-helm-chart/actions/workflows/helm-lint.yaml/badge.svg)](https://github.com/ChiliChonka/grafana-stack-helm-chart/actions/workflows/helm-lint.yaml)
[![Release](https://github.com/ChiliChonka/grafana-stack-helm-chart/actions/workflows/helm-release.yaml/badge.svg)](https://github.com/ChiliChonka/grafana-stack-helm-chart/actions/workflows/helm-release.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Helm umbrella chart for a **Grafana LGTM observability stack on Kubernetes**.

Deploys a full LGTM-style stack using official Grafana Helm charts:

| Component | Purpose |
|-----------|---------|
| [Grafana](https://grafana.com/grafana/) | Dashboards, alerting UI, data-source management |
| [Loki](https://grafana.com/oss/loki/) | Log aggregation & storage |
| [Tempo](https://grafana.com/oss/tempo/) | Distributed trace storage |
| [Mimir](https://grafana.com/oss/mimir/) | Long-term metrics storage |
| [Alloy](https://grafana.com/oss/alloy/) | Collector / agent (replaces Grafana Agent) |
| [Beyla](https://grafana.com/oss/beyla/) | eBPF auto-instrumentation *(optional)* |

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

---

## Configuration

Copy `grafana-stack/values.example.yaml` as a starting point:

```bash
cp grafana-stack/values.example.yaml my-values.yaml
# edit my-values.yaml …

helm upgrade --install grafana-stack grafana-stack \
  -n observability \
  --create-namespace \
  --values my-values.yaml
```

See [`grafana-stack/README.md`](grafana-stack/README.md) for the full parameter reference.

---

## Repository Layout

```
.
├── grafana-stack/
│   ├── Chart.yaml
│   ├── values.yaml          # default values (documented)
│   ├── values.example.yaml  # example / quickstart values
│   ├── .helmignore
│   └── templates/
│       ├── _helpers.tpl
│       └── NOTES.txt
├── .github/
│   └── workflows/
│       ├── helm-lint.yaml       # lint + render on every PR / push
│       └── helm-release.yaml    # package & push to ghcr.io on git tag
├── LICENSE
└── README.md
```

---

## Releasing a New Version

1. Update `version` in `grafana-stack/Chart.yaml`.
2. Commit and push the change.
3. Create a git tag matching `v*` (e.g. `v0.2.0`):

   ```bash
   git tag v0.2.0
   git push origin v0.2.0
   ```

The [helm-release workflow](.github/workflows/helm-release.yaml) will automatically package the chart and push it to the GitHub Container Registry:

```
oci://ghcr.io/chiliChonka/charts/grafana-stack
```

---

## Contributing

Pull requests are welcome. Please ensure:

- `helm lint grafana-stack` passes locally
- `helm dependency update grafana-stack` has been run if you changed `Chart.yaml`
- Changes are documented in `grafana-stack/README.md`

---

## License

[MIT](LICENSE)
