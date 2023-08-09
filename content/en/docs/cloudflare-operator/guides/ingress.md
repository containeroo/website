---
title: "Create DNS records from Ingress"
description: "Learn how to create DNS records from Ingress"
weight: 10
---

cloudflare-operator can create DNS records from Ingress resources. This guide shows how to configure the controller to automatically create DNS records for your Ingress resources.

## Ingress annotations

The following annotation is required: `cf.containeroo.ch/content` or `cf.containeroo.ch/ip-ref`

To skip the creation of a DNS record, use the annotation `cf.containeroo.ch/ignore: "true"`\
If the DNSRecord was previously created, it get's deleted after setting the ignore annotation.

The following annotations are optional:

| Annotation                   | Value                     | Description                                                 |
| ---------------------------- | ------------------------- | ----------------------------------------------------------- |
| `cf.containeroo.ch/content`  | IP address or domain      | DNS record content (e.g. `69.42.0.69`)                      |
| `cf.containeroo.ch/ip-ref`   | Reference to an IP object | e.g. `my-external-ip`                                       |
| `cf.containeroo.ch/proxied`  | `true` or `false`         | Whether the record should be proxied                        |
| `cf.containeroo.ch/ttl`      | `1` or `60` - `86400`     | TTL of the DNS record                                       |
| `cf.containeroo.ch/type`     | `A`, `AAAA` or `CNAME`    | Desired DNS record type                                     |
| `cf.containeroo.ch/interval` | e.g. `5m`                 | Interval at which the DNSRecord object should be reconciled |

An example Ingress resource with annotations:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cf.containeroo.ch/type: CNAME
    cf.containeroo.ch/content: example.com
  name: blog
  namespace: blog
spec:
  rules:
    - host: blog.example.com
      http:
        paths:
          - backend:
              service:
                name: blog
                port:
                  name: http
            path: /
            pathType: Prefix
```

This will create a DNS record for the host `blog.example.com` with the content `example.com` and the type `CNAME`.

{{% alert color="info" %}}
**Note**\
cloudflare-operator only supports `networking.k8s.io/v1` Ingresses.
{{% /alert %}}
