---
title: "Migration to cloudflare-operator"
description: "Learn how to migrate DNS records to cloudflare-operator"
weight: 50
---

cloudflare-operator is designed to be the single source of truth for DNS records. This means that all records not known to cloudflare-operator will be deleted.

In order to prevent the deletion of the entire DNS zone after you have installed cloudflare-operator, you need to migrate your DNS records to cloudflare-operator.

For that we have created a <a href="https://github.com/containeroo/cfop-generator" target="blank">migration tool</a> that generates DNSRecord objects for all DNS records in your DNS zone.

To export your DNS records, follow <a href="https://developers.cloudflare.com/dns/manage-dns-records/how-to/import-and-export/#export-records" target="blank">this guide</a>.

Then, run the migration tool:

```bash
cfop-generator -file <path-to-exported-file>
```

The migration tool will output the generated DNSRecord objects to the console.

Make sure to verify the generated objects before applying them to your cluster.

```bash
kubectl apply -f <path-to-generated-file>
```
