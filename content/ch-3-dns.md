---
weight: 1
title: "Ch 3: DNS Server"
bookToc: true
type: "docs"
bookFlatSection: true
---

# DNS Server

A DNS server resolves host names like 'google.com' to an IP address.

I want a DNS server to resolve a host name like 'homelab.lan' on my local network to my box's IP.

## Router DNS

Some home routers include a local DNS server that can be configured.

Mine does not, so I will run one separately on my linux box.

Then I'll update my router to point to it.

## Dnsmasq

[This](https://thekelleys.org.uk/dnsmasq/doc.html) is an extremely lightweight and simple DNS server that often comes bundled with home routers.

### Installation

It installs easily with `sudo apt install dnsmaq`

### Configuration

I create a new config file at `/etc/dnsmasq.d/homelab.conf` with my settings:

![dnsmasq config](/dns/dnsmasq_conf.png)

Next, I'll enter the homelab host record in my `/etc/hosts` file that dnsmasq will read.

![homelab host](/dns/homelab_host.png)

Finally to load the config changes, I restart dnsmaq.

![dnsmasq reloaded](/dns/dnsmasq_reload.png)

### Router Update

I update my router settings and configure my linux box IP as the primary DNS.

![update router dns settings](/dns/router_dns_setting.png)

My router does a full restart so the changes take effect.

## Troubleshooting

I'm unable to ping `homelab.lan` from my laptop.

I run an `nslookup` on the domain and notice that it's using `127.0.0.53` as my DNS, which is odd.
![ping and nslookup fails](/dns/ping_and_nslookup.png)

Inspecting `/etc/resolv.conf` reveals that systemd-resolved is handling my DNS config.
![resolv conf](/dns/resolv_conf.png)

I edit the config at `/etc/systemd/resolved.conf` and manually set my primary & fallback DNS.

![resolved conf](/dns/resolved_conf.png)

Restart afterwards with `sudo systemctl restart systemd-resolved`

This fixes my issue with nslookup!

![nslookup correct](/dns/nslookup_correct.png)

But I'm still unable to ping.

I learn that ping resolves hosts using a config at `/etc/nsswitch.conf`.

The "[NOTFOUND=return]" on the hosts line is blocking fallback to my DNS (which happens later in this list).

![problem nsswitch](/dns/problem_nsswitch.png)

So I remove that early return on that line, and success!  I can ping.
![ping homelab.lan](/dns/ping_working.png)

DNS is all set up.
