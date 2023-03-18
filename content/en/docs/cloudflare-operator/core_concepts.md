---
title: "Core Concepts"
description: "cloudflare-operator core concepts"
weight: 10
---

## Architecture

cloudflare-operator is designed to be the single source of truth for
Cloudflare DNS records.

It relies on the Kubernetes API to store the desired state of DNS records
using Custom Resource Definitions (CRDs).

![cloudflare-operator architecture](/img/cloudflare-operator-architecture.svg)

## Self-Healing

Failed objects will be reconciled by the controller at the given interval.

### Account

| Error                                                 | Interval |
| :---------------------------------------------------- | :------- |
| Referenced secret (`secretRef`) not found             | 30s      |
| `apiKey` in referenced secret (`secretRef`) not found | 30s      |
| Connection to Cloudflare                              | 30s      |
| Fetching zones from Cloudflare                        | 30s      |
| Fetching zones from `Zone` object                     | 30s      |

### Zone

| Error                                                | Interval |
| :--------------------------------------------------- | :------- |
| `apiKey` in secret from `Account.secretRef` is empty | 5s       |
| Fetching zones from Cloudflare                       | 30s      |
| Fetching `DNSRecord` objects                         | 30s      |
| Fetching DNS records from Cloudflare                 | 30s      |

### IP

| Error                                                       | Interval |
| :---------------------------------------------------------- | :------- |
| None of the provided IP sources return a valid IPv4 address | 60s      |
| Fetching `DNSRecord` objects                                | 30s      |

### Ingress

| Error                        | Interval |
| :--------------------------- | :------- |
| Fetching `DNSRecord` objects | 30s      |

### DNSRecord

| Error                                                     | Interval |
| :-------------------------------------------------------- | :------- |
| `apiKey` in secret from `Account.spec.secretRef` is empty | 5s       |
| Fetching zones from Cloudflare                            | 30s      |
| `Zone.name` in Cloudflare not found                       | 30s      |
| `Zone` object not ready                                   | 5s       |
| Fetching zones from Cloudflare                            | 30s      |
| Fetching DNS records from Cloudflare                      | 30s      |
| Referenced `IP` object (`spec.ipRef.name`) not found      | 30s      |
