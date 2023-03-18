---
title: "Installation"
description: "Guide to install cloudflare-operator"
weight: 30
---

This guide walks you through installing cloudflare-operator.

## Prerequisites

- Install Helm 3
- Kubernetes cluster

## Install cloudflare-operator

### Helm repository

Add the cloudflare-operator Helm chart repository:

```bash
helm repo add containeroo https://charts.containeroo.ch
```

Update the Helm chart repository:

```bash
helm repo update
```

### Custom Resource Definitions

As per the Helm best practices, cloudflare-operator Helm chart doesn't ship with CRDs.

To install the latest CRDs, run the following command:

```bash
kubectl apply -f https://github.com/containeroo/cloudflare-operator/releases/latest/download/crds.yaml
```

If you want to install a specific version of CRDs, run the following command:

```bash
export VERSION=x.y.z
kubectl apply -f https://github.com/containeroo/cloudflare-operator/releases/download/v${VERSION}/crds.yaml
```

### Operator installation

#### Default installation

To install the latest version of cloudflare-operator, run the following command:

```bash
helm install \
  cloudflare-operator containeroo/cloudflare-operator \
  --namespace cloudflare-operator \
  --create-namespace
```

If you want to install a specific version of cloudflare-operator, run the following command:

```bash
export VERSION=x.y.z
helm install \
  cloudflare-operator containeroo/cloudflare-operator \
  --namespace cloudflare-operator \
  --create-namespace \
  --version v${VERSION}
```

#### Customized installation

Create a `values.yaml` file.  
A full list of all supported Helm values can be found <a href="https://artifacthub.io/packages/helm/containeroo/cloudflare-operator" target="blank">here</a>.

Example `values.yaml` file:

```yaml
---
image:
  repository: ghcr.io/containeroo/cloudflare-operator
  tag: latest
  pullPolicy: IfNotPresent
```

Run the following command to install cloudflare-operator with the customized Helm values:

```bash
helm install \
  cloudflare-operator containeroo/cloudflare-operator \
  --namespace cloudflare-operator \
  --create-namespace \
  --values values.yaml
```
