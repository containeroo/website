---
title: "Fetch IP from a Kubernetes object"
description: "Learn how to use cloudflare-operator to fetch an IP address from a Kubernetes object"
weight: 40
---

If you have a Kubernetes object that contains an IP address, you can use cloudflare-operator to fetch the IP address from the object and use it as the target content for a DNS record.

This can be useful if you are using Istio or want to fetch the IP from a Kubernetes service.

This guide will show you how to configure an IP object to fetch the IP address from a Kubernetes service.

## Operator configuration

Add a kubectl sidecar to the operator pod by using the following Helm values:

```yaml
sidecars:
  - name: proxy
    image: bitnami/kubectl
    args: ["proxy", "--port=8858"]
```

The operator will need additional permissions to access services.

In order to grant the operator these permissions, add the following Helm values:

```yaml
clusterRole:
  extraRules:
    - apiGroups:
        - ""
      resources:
        - services
      verbs:
        - get
        - list
```

Install or update the Helm chart using [this guide](/docs/cloudflare-operator/installation).

## IP object

An IP object could then look like this:

```yaml
---
apiVersion: cloudflare-operator.io/v1
kind: IP
metadata:
  name: ingress-ip
spec:
  type: dynamic
  ipSources:
    - url: localhost:8858/api/v1/namespaces/ingress-nginx/services/ingress-nginx
      responseJQFilter: ".status.loadBalancer.ingress[0].ip"
```

This IP object will fetch the IP address from the Kubernetes service `ingress-nginx` in the namespace `ingress-nginx` using the kubectl proxy.
