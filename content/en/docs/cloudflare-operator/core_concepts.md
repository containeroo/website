---
title: "Core Concepts"
description: "cloudflare-operator core concepts"
weight: 10
---

## Architecture

cloudflare-operator is designed to serve as the single source of truth for Cloudflare DNS records.  
It relies on the Kubernetes API to store the desired state of DNS records using Custom Resource Definitions (CRDs).

![cloudflare-operator architecture](/img/cloudflare-operator-architecture.svg)

## DNS records

Cloudflare DNS records are specified using a CRD (`dnsrecords.cf.containeroo.ch`).  
These records can be created manually, through a GitOps workflow, or automatically generated from Kubernetes Ingress resources.

The Kubernetes API serves as the "single source of truth" for all zones in the configured Cloudflare account.

For more information on creating and using DNS records, please refer to the [DNSRecords documentation](/docs/cloudflare-operator/dns_records).

## IP objects

IP objects can be utilized to follow the "don't repeat yourself" (DRY) principle.

DNS records can be configured to use an IP object as the target content.  
If the IP object is updated, all DNS records that use it will be updated automatically.

The effective IP can either be configured in the IP object, or it can be dynamically fetched from the internet.  
This enables you to use cloudflare-operator as a dynamic DNS controller.

## Reconciliation

Reconciliation is the process of ensuring that the state of the cluster aligns with the desired state.

This process also incorporates "self-healing" by retrying failed operations after a specified interval.
