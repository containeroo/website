---
title: "Zone"
description: "Learn how to use the Zone resource"
weight: 20
---

The API specification can be viewed [here](/docs/cloudflare-operator/api_reference/#cloudflare-operator.io/v1.Zone).

Zone resources represent Cloudflare zones and are used to tell cloudflare-operator which zones to manage.

Learn more about the zone resource in the [getting started guide](/docs/cloudflare-operator/get_started).

## Prevent Certain DNS Records from Being Deleted When Zone Prune Was Enabled

Prior to **v1.7.0**, `cloudflare-operator` had a fixed set of DNS records that were always ignored when `Zone` prune was enabled. These included records required for ACME certificate validation and Cloudflare-specific TXT records. Users could not modify this behavior.

Starting with **v1.7.0**, the operator introduces the `ignoredRecords` field in the `Zone` spec, allowing fine-grained control over which DNS records are exempt from deletion when `Zone` prune is enabled.

## Behavior

- The `ignoredRecords` field is **only evaluated when `Zone` prune is enabled** (`prune: true`).
- Records can be specified per type (e.g., `TXT`, `CNAME`).
- Names can be treated as prefixes or regular expressions: lines starting with `^` are interpreted as regex; all other entries are treated as literal prefixes.

## Default Records

By default, the operator ignores a minimal set of essential records to prevent accidental deletion:

- `_acme-challenge` (`TXT`, used for ACME certificate validation)
- `cf2024-1._domainkey` (`TXT`, Cloudflare-specific record)

## Customizing Ignored Records

You can extend or override the defaults by specifying your own `ignoredRecords` in the `Zone` spec. For example:

```yaml
spec:
  ignoredRecords:
    TXT:
      - "_acme-challenge"
      - "custom-txt-record"
    CNAME:
      - "^cf-.*"
```

In this configuration:

- `_acme-challenge` and `custom-txt-record` will be ignored for `TXT` records.
- Any `CNAME` record matching the regex `^cf-.*` will also be ignored.

**Important:** If you define your own `ignoredRecords`, you must include any default entries you want to preserve (such as `_acme-challenge`). The operator does **not** automatically merge defaults with user-defined values.
