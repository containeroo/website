---
title: "Traefik 2.0: Wildcard Let's Encrypt Certificates"
author: "containeroo"
date: 2019-11-12
lastmod: 2022-02-04

description: "How to use wildcard certificates with Traefik 2.0"

tags:
  - traefik
  - docker
  - letsencrypt
  - cloudflare
---

<img alt="Header image" src="featured-image.png" />

[Image Source](https://letsencrypt.org/images/le-logo-wide.png)

## Introduction

In this tutorial we will setup Traefik to obtain wildcard certificates from Let's Encrypt. This requires DNS challenge to be setup. Usually Traefik obtains a certificate for every subdomain. We can simplify this process by telling Traefik to use a wildcard (\*.example.com) certificate instead.

## Prerequisites

- Registered Domain
- Authoritative DNS Servers from one of [these providers](https://docs.traefik.io/https/acme/#providers) (you may need to change your DNS servers of your domain to one of the provider in the list)

In this tutorial we will use Cloudflare as our DNS servers for our domain.

## Setup DNS challenge

If you have followed our other guides, chances are you currently use HTTP challenge. These types of challenges define how Let's Encrypt assures that you are the owner of the domain you want to obtain a certificate for.

In order to get a wildcard certificate, you have to use DNS challenge.

First of all make sure you connect your domain with one of the supported DNS providers. We are using Cloudflare. This depends on where you bought your domain, so we can't show you exactly how to do it.

In the Traefik Docker compose file we add the following lines:

```yaml
environment:
  - CF_API_EMAIL=your-cloudflare@email.com
  - CF_API_KEY=your-cloudflare-api-key
```

[Here is the full Traefik Docker compose](https://gist.github.com/containeroo-gists/0e79fb145252611ee1bb0da2c31f243d)

Next, we tell Traefik to use DNS challenge (edit the file traefik.yml):

```yaml
certificatesResolvers:
  cloudflare:
    acme:
      email: your@email.com
      storage: acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

[Here is the full Traefik configuration file](https://gist.github.com/containeroo-gists/373247c3c54a11272b65daf99bca6fc4)

Create a backup of your existing acme.json and clear the current file:

```bash
cp -p acme.json acme.json.bak && > acme.json
```

## Setup the wildcard certificate

Change the Traefik Docker compose labels (make sure to change your domain accordingly):

```yaml
- "traefik.http.routers.traefik.tls=true"
- "traefik.http.routers.traefik.tls.certresolver=cloudflare"
- "traefik.http.routers.traefik.tls.domains[0].main=example.com"
- "traefik.http.routers.traefik.tls.domains[0].sans=*.example.com"
- "traefik.http.routers.traefik.service=api@internal"
```

[Here is the full Traefik Docker compose](https://gist.github.com/containeroo-gists/0e79fb145252611ee1bb0da2c31f243d)

## Tell Traefik to use the wildcard certificate for each service

Now we have to remove one label from every service:

```yaml
- traefik.http.routers.service.tls.certresolver=cloudflare
```

[Here is an example compose file](https://gist.github.com/containeroo-gists/27666fe7d32199f40c01b20f49cc0454)

Once you have removed the line above from all your services, Traefik should always use the wildcard certificate.
