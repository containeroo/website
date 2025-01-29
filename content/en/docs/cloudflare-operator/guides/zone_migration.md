---
title: "Zone Migration"
linkTitle: "Zone Migration"
description: "Migrate to user managed zones"
weight: 30
---

Starting with version v1.4.0, the management of DNS records has been refined to allow for externally managed records.
This change requires users to explicitly define which zones the operator should manage, providing greater control and preventing unintended modifications to DNS records.

If you already use cloudflare-operator prior to v1.4.0, cloudflare-operator will continue working as before. However, if you follow the GitOps approach, you now also have
to manage the zones that the operator should manage.
