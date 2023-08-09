---
title: "Dynamic DNS with IP objects"
description: "Implement dynamic DNS with cloudflare-operator"
weight: 30
---

As described in the [core concept](/docs/cloudflare-operator/core_concepts/#ip-objects), IP objects allow you to
use cloudflare-operator as a dynamic DNS controller.

In this guide, we will configure an IP object to fetch the public IP address from the internet and use it as the target
content for a DNS record.

## Simple example

Create an IP object with the following content:

```yaml
---
apiVersion: cf.containeroo.ch/v1beta1
kind: IP
metadata:
  name: external-v4
spec:
  ipSources:
    - requestMethod: GET
      url: https://ifconfig.me/ip
    - requestMethod: GET
      url: https://ipecho.net/plain
    - requestMethod: GET
      url: https://myip.is/ip/
    - requestMethod: GET
      url: https://checkip.amazonaws.com
    - requestMethod: GET
      url: https://api.ipify.org
  type: dynamic
  interval: 5m0s
```

The IP object will fetch the public IP address from the internet using one of the specified URLs.\
If the request fails, the next URL will be tried. If all URLs fail, the last known IP will be used and
the ready condition will be set to `false`.

Now, create a DNS record object with the following content:

```yaml
---
apiVersion: cf.containeroo.ch/v1beta1
kind: DNSRecord
metadata:
  name: example-com
  namespace: cloudflare-operator
spec:
  name: example.com
  ipRef:
    name: external-v4 # reference to the IP object
  proxied: true
  ttl: 1
  type: A
```

## Response filtering

### Using jq

You can use [jq](https://stedolan.github.io/jq/) to filter the response content.

Here is an example for ipify.org:

Request content:

```bash
$ curl 'https://api.ipify.org?format=json'
{"ip":"69.42.0.69"}
```

The IP object would look like this:

```yaml
---
apiVersion: cf.containeroo.ch/v1beta1
kind: IP
metadata:
  name: external-v4
spec:
  ipSources:
    - requestMethod: GET
      url: https://api.ipify.org?format=json
      responseJQFilter: .ip
  type: dynamic
  interval: 5m0s
```

### Using regex

If you want to use a regex, you can use the `postProcessingRegex` field.

This might be useful if you want to extract the IP address from a HTML page.

{{% alert color="info" %}}
**Note**\
The regex must contain a single capture group.
{{% /alert %}}

Here is an example for ifconfig.me:

Request content:

```bash
$ curl 'https://ifconfig.me/'
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
...
```

In this case, the IP object would look like this:

```yaml
---
apiVersion: cf.containeroo.ch/v1beta1
kind: IP
metadata:
  name: external-v4
spec:
  ipSources:
    - requestMethod: GET
      url: https://ifconfig.me/
      postProcessingRegex: '(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})'
  type: dynamic
  interval: 5m0s
```

## Fetching IP from an API

If you want to fetch the IP address from an API, you can use the `responseJQFilter` field.

In this example, we are going to fetch the IP from a Hetzner cloud instance.\
To authenticate with the API, you can configure the IP object to use a secret
where the secret key will be used as the header name
and the secret value will be used as the header value.

Create a secret with the following content:

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: hetzner-cloud-api-key
  namespace: cloudflare-operator
stringData:
  Authorization: Bearer <your-api-token>
```

Now, create an IP object with the following content:

```yaml
---
apiVersion: cf.containeroo.ch/v1beta1
kind: IP
metadata:
  name: my-instance-v4
spec:
  ipSources:
    - requestHeaders:
        Accept: application/json
      requestHeadersSecretRef:
        name: hetzner-cloud-api-key
        namespace: cloudflare-operator
      requestMethod: GET
      responseJQFilter: .servers[] | select(.name == "my-instance.example.com").public_net.ipv4.ip
      url: https://api.hetzner.cloud/v1/servers
  type: dynamic
  interval: 5m0s
```
