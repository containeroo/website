---
title: "Create DNS records from Gateway API HTTPRoute"
description: "Learn how to create DNS records from HTTPRoute objects"
weight: 11
---

cloudflare-operator can create DNS records from Gateway API `HTTPRoute` resources. This is an opt-in feature and is available starting with `cloudflare-operator` v1.8.0; earlier releases ignore HTTPRoute resources.

## Prerequisites

- Install the Gateway API CRDs (for example `gateway.networking.k8s.io/v1` from the upstream install manifests).
- Start cloudflare-operator with the flag `--enable-gateway-api`.

## HTTPRoute annotations

The same annotations used for Ingress are honored on HTTPRoute objects. One of these is required:\
`cloudflare-operator.io/content` or `cloudflare-operator.io/ip-ref`

HTTPRoute objects that do not have one of these annotations will be ignored by cloudflare-operator.

| Annotation                        | Value                     | Description                                                 | Required                    |
| --------------------------------- | ------------------------- | ----------------------------------------------------------- | --------------------------- |
| `cloudflare-operator.io/content`  | IP address or domain      | DNS record content (e.g. `69.42.0.69`)                      | yes if `ip-ref` is not set  |
| `cloudflare-operator.io/ip-ref`   | Reference to an IP object | e.g. `my-external-ip`                                       | yes if `content` is not set |
| `cloudflare-operator.io/proxied`  | `true` or `false`         | Whether the record should be proxied                        | no                          |
| `cloudflare-operator.io/ttl`      | `1` or `60` - `86400`     | TTL of the DNS record                                       | no                          |
| `cloudflare-operator.io/type`     | `A`, `AAAA` or `CNAME`    | Desired DNS record type                                     | no                          |
| `cloudflare-operator.io/interval` | e.g. `5m0s`               | Interval at which the DNSRecord object should be reconciled | no                          |
| `cloudflare-operator.io/comment`  | e.g. `hello world`        | An optional comment to add to the DNS record                | no                          |

Example HTTPRoute:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: blog
  namespace: cloudflare-operator-system
  annotations:
    cloudflare-operator.io/content: example.com
    cloudflare-operator.io/type: CNAME
spec:
  parentRefs:
    - name: blog-gateway
  hostnames:
    - blog.example.com
  rules:
    - backendRefs:
        - name: blog
          port: 80
```

This creates a DNS record for the host `blog.example.com` with content `example.com` and type `CNAME`.
