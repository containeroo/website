---
title: "Monitoring with Prometheus"
linkTitle: "Monitoring with Prometheus"
description: "Monitor cloudflare-operator with Prometheus"
weight: 20
---

## Prerequisites

- [Prometheus](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus)
- [Grafana](https://github.com/grafana/helm-charts/tree/main/charts/grafana)
- [kube-state-metrics](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics)

The easiest way to deploy all the necessary components is to use
[kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack).

## Enable metrics

In order to enable metrics and automatically deploy the required resources, you need to reconfigure the Helm chart.

Create a `values.yaml` file with the following content:

```yaml
---
metrics:
  podMonitor:
    enabled: true
  prometheusRule:
    enabled: true
```

Now you can install / upgrade the Helm chart by following [the installation guide](/docs/cloudflare-operator/installation/#customized-installation).

## Install cloudflare-operator Grafana dashboard

```bash
# Download Grafana dashboard
wget https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/grafana/dashboards/overview.json -O /tmp/grafana-dashboard-cloudflare-operator.json

# Create the configmap
kubectl create configmap grafana-dashboard-cloudflare-operator --from-file=/tmp/grafana-dashboard-cloudflare-operator.json

# Add label so Grafana can fetch dashboard
kubectl label configmap grafana-dashboard-cloudflare-operator grafana_dashboard="1"
```

## Available metrics

For each `cloudflare-operator.io` kind, the controller exposes a gauge metric to track the status condition.

Ready status metrics:

```text
cloudflare_operator_account_status
cloudflare_operator_dns_record_status
cloudflare_operator_ip_status
cloudflare_operator_zone_status
```

## Alerting

The following alerting rule can be used to monitor DNS record failures:

```yaml
groups:
  - alert: DNSRecordFailures
    annotations:
      summary:
        DNSRecord {{ $labels.name }} ({{ $labels.record_name }}) in namespace
        {{ $labels.exported_namespace }} failed
    expr: cloudflare_operator_dns_record_status > 0
    for: 1m
    labels:
      severity: critical
```
