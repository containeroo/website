---
title: "Zone Migration"
linkTitle: "Zone Migration"
description: "Migrate to user managed zones"
weight: 30
---

## Zone Migration

In the latest update to the cloudflare-operator, the management of DNS records has been refined to allow for externally managed records. This change requires users to explicitly define which zones the operator should manage, providing greater control and preventing unintended modifications to DNS records.

**Key Changes:**

- **Explicit Zone Management:** Users must now specify the zones they want the cloudflare-operator to manage. This is done by creating a `Zone` resource for each zone. Any DNS records not within these specified zones will remain unmanaged by the operator.

**Updated Configuration Steps:**

1. **Create Cloudflare API Token:**
   - Ensure your API token has the necessary permissions:
     - Zone:Zone:Read
     - Zone:DNS:Edit
   - Configure the token to include only the specific zones you intend to manage.

2. **Configure Cloudflare Account:**
   - Create a Kubernetes secret containing your API token:
     ```yaml
     apiVersion: v1
     kind: Secret
     type: Opaque
     metadata:
       name: cloudflare-api-token
       namespace: cloudflare-operator
     stringData:
       apiToken: your_api_token_here
     ```
   - Define an `Account` resource referencing this secret:
     ```yaml
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

3. **Define Managed Zones:**
   - For each zone you want the operator to manage, create a `Zone` resource:
     ```yaml
     apiVersion: cloudflare-operator.io/v1
     kind: Zone
     metadata:
       name: example-com
     spec:
       account:
         name: account-sample
       zone: example.com
     ```
   - Replace `example.com` with your actual zone name.

4. **Create DNS Records:**
   - With the zones defined, you can now create `DNSRecord` resources within these zones:
     ```yaml
     apiVersion: cloudflare-operator.io/v1
     kind: DNSRecord
     metadata:
       name: www-example-com
       namespace: cloudflare-operator
     spec:
       zone: example.com
       name: www
       type: A
       content: 192.0.2.1
       proxied: true
       ttl: 1
     ```
   - Ensure the `zone` field matches one of the zones you've defined in your `Zone` resources.

**Important Notes:**

- **Data Safety:** Previously, the operator would delete all DNS records in every zone accessible by the API token upon configuration. With the new changes, only the DNS records within the explicitly defined zones (`Zone` resources) will be managed, leaving other zones unaffected.

- **Migration:** If you're updating from a previous version of the operator, ensure that you define `Zone` resources for all zones you wish to continue managing. This will prevent any unintended loss of DNS records.

For a comprehensive guide and additional details, please refer to the official documentation: ([contineroo.ch](https://containeroo.ch/docs/cloudflare-operator/get_started/?utm_source=chatgpt.com))

By following these updated steps, you can ensure that the cloudflare-operator manages only the DNS records you intend, providing a safer and more controlled DNS management experience.
