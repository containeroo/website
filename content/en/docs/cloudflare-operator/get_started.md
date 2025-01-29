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

## Create Cloudflare API token

The token can be created by following <a href="https://developers.cloudflare.com/fundamentals/api/get-started/create-token/" target="blank">this guide</a>.

The following permissions are required:

- `Zone:Zone:Read`
- `Zone:DNS:Edit`

Configure the following `Zone resources`:

- `Include:All zones`

or, if you want to limit the zones to which the token has access:

- `Include:Specific zone:example.com`

The summary should look similar to this:

`All zones - Zone:Read, DNS:Edit`

## Configure Cloudflare account

Create a secret with the previously created Cloudflare API token.

{{% alert color="info" %}}
**Note**\
The key in the secret must be named `apiToken`.
{{% /alert %}}

```yaml
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: cloudflare-api-token
  namespace: cloudflare-operator
stringData:
  apiToken: 1234 # change this to your Cloudflare API token
```

Or alternatively, you can use the following command:

```bash
kubectl create secret generic cloudflare-api-token --namespace=cloudflare-operator --from-literal=apiToken=<YOUR-CLOUDFLARE-API-TOKEN>
```

Next, create an account object:

```yaml
---
apiVersion: cloudflare-operator.io/v1
kind: Account
metadata:
  name: account-sample
spec:
  apiToken:
    secretRef:
      name: cloudflare-api-token
      namespace: cloudflare-operator
```

Check if the account is ready:

```bash
kubectl get accounts.cloudflare-operator.io
```

This should output the following:

```console
NAME             READY
account-sample   True
```

{{% alert color="warning" %}}
**Attention!** :warning:\
Note that after a successful installation and configuration, if the prune option is enabled, cloudflare-operator will delete **ALL** DNS records in **EVERY ZONE** for which you have created a Zone object!\
It is therefore highly recommended to <a href="https://developers.cloudflare.com/dns/manage-dns-records/how-to/import-and-export/#export-records" target="blank">export your existing DNS records</a> first!
You can migrate all your DNS records to cloudflare-operator by following <a href="/docs/cloudflare-operator/guides/migration" target="blank">this guide</a>.
{{% /alert %}}

Next, create a zone object:

```yaml
---
apiVersion: cloudflare-operator.io/v1
kind: Zone
metadata:
  name: example-com
spec:
  name: example.com
  prune: false # default value
```

Verify that the zone is ready:

```bash
kubectl get zones.cloudflare-operator.io
```

```console
NAME          ZONE NAME      ID                                 READY
example-com   example.com    12345678901234567890123456789012   True
```

## Create a DNS record

Now, we can create our first DNS record:

```yaml
---
apiVersion: cloudflare-operator.io/v1
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
  interval: 5m0s
```

Check the status of the DNS record:

```bash
kubectl get dnsrecords.cloudflare-operator.io --namespace cloudflare-operator
```

```console
NAME            RECORD NAME     TYPE    READY
example-com     example.com     A       True
```
