+++
title = "Setup DoT (DNS over TLS) on Fedora with systemd-resolved"
date = "2024-12-26T20:36:43+05:30"
description = "Guide for setting up DNS over TLS on Fedora with systemd-resolved so that your ISP doesn't see your DNS traffic"

tags = ["blog","developer","systemd","notes","resolved","doh","dot","systemd-resolved", "dns"]
+++

# DoH vs DoT

"DNS over HTTPS" (DoH) and "DNS over TLS" (DoT) are protocols for encrypted DNS resolution, they are both using TLS for encrypting your DNS queries and responses.
Other alternatives for encrypted DNS are DNSSEC and DNSCrypt.

One of the main differences between DoT and DoH is that the latter uses the standard HTTPS port 443 and HTTP/2 or HTTP/1.1 protocol for the DNS queries
which essentially masqurades the DNS traffic as normal web traffic, DoT on the other hand uses port 853 and directly uses UDP with TLS encrypting on top.

The default caching resolver on Fedora is `systemd-resolved` which doesn't yet support DoH but it does support DoT.

# Configure `systemd-resolved` to use DoT

The main configuration file for `systemd-resolved` is `/etc/systemd/resolved.conf` and the drop-in config files are at `/etc/systemd/resolved.conf.d/`. Fedora ships a sample config file
at `/usr/lib/systemd/resolved.conf`

## Create the config file for `systemd-resolved`

1. Create the drop-in config directory for resolved:
```
$ mkdir -p /etc/systemd/resolved.conf.d
```
2. Copy and paste the following content in the file `/etc/systemd/resolved.conf.d/01-dns-o-tls.conf`:

```
[Resolve]
DNS=1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4700::1001#cloudflare-dns.com
FallbackDNS=9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net 2620:fe::9#dns.quad9.net
DNSOverTLS=yes
```
3. Restart the `systemd-resolved` service:

```
$ sudo systemctl restart systemd-resolved
```
4. Verify that the drop-in config file is being used by checking the presence of the path `/etc/systemd/resolved.conf.d/01-dns-o-tls.conf` in the output of the following command:

```
$ systemd-analyze cat-config systemd/resolved.conf

[...]
# /etc/systemd/resolved.conf.d/01-dns-o-tls.conf
[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4700::1001#cloudflare-dns.com
# Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:4860:4860::8844#dns.google
# Quad9:      9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net 2620:fe::9#dns.quad9.net
DNS=1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4700::1001#cloudflare-dns.com
FallbackDNS=9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net 2620:fe::9#dns.quad9.net
DNSOverTLS=yes

```
We can also use `resolvectl status` to get the current effective config. Please check `man systemd-resolved` and `make resolved.conf` for more in-depth details.
