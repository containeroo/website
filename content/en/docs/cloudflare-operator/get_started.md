---
title: "Get Started"
description: "Get started with cloudflare-operator"
weight: 20
---

This tutorial shows you how to get started with using cloudflare-operator and create a sample DNS record.

## Before you begin

The following prerequisites are required to complete this tutorial:

- A Kubernetes cluster with cloudflare-operator installed (follow [the installation guide](/docs/cloudflare-operator/installation))
- A Cloudflare account

{{% alert color="warning" %}}
**Attention!** :warning:  
Note that after a successful installation and configuration, cloudflare-operator will delete **ALL** DNS records in **EVERY ZONE**!  
It is therefore highly recommended to <a href="https://developers.cloudflare.com/dns/manage-dns-records/how-to/import-and-export/#export-records" target="blank">export your existing DNS records</a> first!
{{% /alert %}}

## Configure Cloudflare account

Create a secret with your Cloudflare global API key. The key can be fetched by following <a href="https://developers.cloudflare.com/fundamentals/api/get-started/keys/#view-your-global-api-key" target="blank">this guide</a>.  
Support for API tokens is coming soon.

{{% alert color="info" %}}
**Note**  
The key in the secret must be named `apiKey`.  
{{% /alert %}}

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: cloudflare-global-api-key
  namespace: cloudflare-operator
stringData:
  apiKey: 1234
```

Next, create an account object:

```yaml
apiVersion: cf.containeroo.ch/v1beta1
kind: Account
metadata:
  name: account-sample
spec:
  email: mail@example.com
  globalAPIKey:
    secretRef:
      name: cloudflare-global-api-key
      namespace: cloudflare-operator
```

{{% alert color="warning" %}}
**Attention!** :warning:  
After this step cloudflare-operator will delete **ALL** DNS records in **EVERY ZONE**!
{{% /alert %}}

Check if the account is ready:

```bash
kubectl get accounts.cf.containeroo.ch
```

This should output the following:

```console
NAME   EMAIL              READY
main   mail@example.com   True
```

```bash
kubectl get zones.cf.containeroo.ch
```

```console
NAME          ZONE NAME      ID                                 READY
example-com   example.com    12345678901234567890123456789012   True
```

## Create a DNS record

Now, we can create our first DNS record:

```yaml
apiVersion: cf.containeroo.ch/v1beta1
kind: DNSRecord
metadata:
  name: example-com
  namespace: cloudflare-operator
spec:
  name: example.com
  type: A
  content: 69.42.0.69
  proxied: true
  ttl: 1
  interval: 5m
```

Check the status of the DNS record:

```bash
kubectl get dnsrecords.cf.containeroo.ch --namespace cloudflare-operator
```

```console
NAME            RECORD NAME     TYPE    READY
example-com     example.com     A       True
```
