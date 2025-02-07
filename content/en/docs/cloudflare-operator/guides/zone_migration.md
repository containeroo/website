---
title: "Zone Migration"
linkTitle: "Zone Migration"
description: "Migrate to user managed zones with v1.4.0"
weight: 30
---

Starting with version v1.4.0, DNS record management has been redesigned to support externally managed records. This update requires users to explicitly specify which zones the operator should manage, providing greater control and preventing unintended modifications.

For users already running cloudflare-operator prior to v1.4.0, existing functionality remains unchanged, except that DNS records are no longer automatically pruned. To restore the previous behavior, set the `spec.prune` field to `true` in the Zone object.

Additionally, if you follow a GitOps approach, you must now explicitly manage the list of zones the operator handles.

cloudflare-operator can also assume control of manually created DNS records, provided the record's name, content, and type match the specifications in the DNSRecord object.
