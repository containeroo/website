---
title: "DNSRecord"
description: "Learn how to use the DNSRecord resource"
weight: 30
---

The API specification can be viewed [here](/docs/cloudflare-operator/api_reference/#cloudflare-operator.io/v1.DNSRecord).

DNSRecord resources represent DNS records in the Cloudflare account.

For each DNS record, you need to create a DNSRecord resource.

cloudflare-operator will create the DNS record in the Cloudflare account and keep it in sync with the DNSRecord resource.

As described in the Ingress and Gateway API guides, cloudflare-operator can create DNSRecords from Ingress and supported Gateway API route resources.

DNSRecords are reconciled through the Account selected by their matching Zone. If the Zone does not set `spec.accountRef.name`, a DNSRecord can set `spec.accountRef.name` directly. If both the Zone and DNSRecord set an Account, they must reference the same Account.

If you want to create DNSRecords manually, you can use the following example as a starting point:

```yaml
apiVersion: cloudflare-operator.io/v1
kind: DNSRecord
metadata:
  name: vpn
  namespace: cloudflare-operator
spec:
  name: vpn.example.com
  accountRef:
    name: account-sample
  type: A
  ipRef:
    name: dynamic-external-ipv4-address
  proxied: false
  ttl: 120
  interval: 5m0s
```

This example creates a DNS record for the domain `vpn.example.com` with the type `A` and the IP address from the `dynamic-external-ipv4-address` IP object.\
It also sets the `proxied` flag to `false` and the `ttl` to `120`.

Another example is the following:

```yaml
apiVersion: cloudflare-operator.io/v1
kind: DNSRecord
metadata:
  name: blog
  namespace: cloudflare-operator
spec:
  name: blog.example.com
  content: 69.42.0.69
  type: A
  proxied: true
  ttl: 1
  interval: 5m0s
```

This example creates a DNS record for the domain `blog.example.com` with the type `A` and the content `69.42.0.69`.
