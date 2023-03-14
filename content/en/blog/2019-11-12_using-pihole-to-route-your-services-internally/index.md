---
title: "Using Pi-hole to route your services internally"
author: "containeroo"
date: 2019-11-12

description: "Connecting to your local server without looping trough the internet"

tags:
  - pihole
  - dns
  - traefik
  - docker
---

<img alt="Header image" src="featured-image.png" />

[Image Source](https://www.taste-of-it.de/wp-content/uploads/2019/09/pihole-logo.png)

## The Problem

If you have followed our previous guides, chances are that you have a domain, some DNS records pointing to your public IP, port forwarding enabled and a Docker server running some services.

Most likely your domain resolves to your public IP from you internal network as well.
This causes a problem: All the traffic between your devices (e.g. your phone) to your server (physically in the same location) gets routed trough the internet, which means you have to utilize your upload and download bandwidth at the same time (e.g. while streaming from Plex), which not only causes a slower connection but also adds an unnecessary high latency.

## The Solution

If you have a [Pi-hole](https://pi-hole.net/) running at home (which you should) you can configure it to resolve your domain (\*.example.com and example.com) to your servers local IP instead of your public IP. This means all your devices will directly connect to your server without looping through the internet, making everything faster. Yay!

Since Pi-hole is nothing else but a DNS server with some special software, we can easily configure the underlying dnsmasq service.

Here's how we can achieve that:

1. Connect to your Raspberry Pi (or wherever Pi-hole is running) via SSH
2. Open the file `/etc/dnsmasq.d/05-custom.conf`

   ```bash
   sudo nano /etc/dnsmasq.d/05-custom.conf
   ```

3. Add the following line (change example.com to your domain and 192.168.1.10 to your servers local IP)

   ```text
   address=/example.com/192.168.1.10
   ```

4. Restart the DNS service

   ```bash
   sudo pihole restartdns
   ```

You can verify the changes by looking up your domain:

Before

```console
[user@server ~]$ dig +short example.com
216.91.241.176
```

After:

```console
[user@server ~]$ dig +short example.com
192.168.1.10
```

If you don't have `dig` installed, you can use the following command to install it on Ubuntu:

```bash
sudo apt install dnsutils
```

That's it!
