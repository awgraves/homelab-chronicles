---
weight: 1
title: "Ch 3: DNS Server"
bookToc: true
type: "docs"
bookFlatSection: true
---

# DNS Server

A DNS server resolves host names like 'google.com' to an IP address.

I want a DNS server to resolve a host name like 'homelab.lan' on my local network.

## Router DNS

Some home routers include a local DNS server that can be configured.

Mine does not, so I will run one separately on my linux box.

Then I'll update my router to point to it.

## Dnsmasq

[This](https://thekelleys.org.uk/dnsmasq/doc.html) is an extremely lightweight and simple DNS server.

It often comes bundled with home routers.

### Installation

Easy with `sudo apt install dnsmaq`

### Configuration

I create a new config file at `/etc/dnsmasq.d/homelab.conf` with my settings:

![dnsmasq config](/dns/dnsmasq_conf.png)

Next, I'll enter the homelab host record in my `/etc/hosts` file that dnsmasq will read.

![homelab host](/dns/homelab_host.png)

I reload the config changes by restarting dnsmaq.

![dnsmasq reloaded](/dns/dnsmasq_reload.png)

### Router Update

I configure my linux box IP as the primary DNS.

![update router dns settings](/dns/router_dns_setting.png)

My router does a full restart and changes should be in effect.

## Troubleshooting

At first, I'm unable to ping `homelab.lan` from my laptop.

I run an `nslookup` on the domain and notice that it's using `127.0.0.53` as my DNS.

This is not expected.

![ping and nslookup fails](/dns/ping_and_nslookup.png)

Inspecting my `/etc/resolv.conf` reveals my DNS is handled by systemd-resolved.

![resolv conf](/dns/resolv_conf.png)

I edit the config at `/etc/systemd/resolved.conf` and manually set my primary & fallback DNS.

![resolved conf](/dns/resolved_conf.png)

After my config is reloaded, nslookup works!

![nslookup correct](/dns/nslookup_correct.png)

But I'm still unable to ping.

I learn that ping resolves hosts using a separate config at `/etc/nsswitch.conf`.

The "[NOTFOUND=return]" on the hosts line is blocking fallback to my DNS (which happens later in this list).

![problem nsswitch](/dns/problem_nsswitch.png)

So I remove that early return, and success! `homelab.lan` is reachable.

![ping homelab.lan](/dns/ping_working.png)
