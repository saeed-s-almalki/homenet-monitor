# 03 — Routing the LAN through Pi-hole & Verifying

## Point the whole network at Pi-hole

On the home router, set the **DHCP DNS** server it hands to clients to the Pi-hole server IP:

- Router admin → **LAN / DHCP settings** → **DNS** → *Manual*
- **Primary DNS:** `192.168.0.155` (Pi-hole)
- **Secondary DNS:** `1.1.1.1` (fail-safe so the internet keeps working if Pi-hole is down)
- Save. Clients pick up the new DNS on their next DHCP lease renewal (toggle Wi-Fi to force it).

> **Availability note:** while this lab runs on a VM, if the host PC is powered off the whole network loses its primary DNS. The secondary (`1.1.1.1`) keeps the internet working. Moving Pi-hole to an always-on Raspberry Pi removes this dependency.

## Verify real devices resolve through Pi-hole

A packet capture on the server proves clients are actually using it:

```bash
sudo tcpdump -i eth0 -nn 'udp port 53'
```

Example — two smart ACs resolving through the server:

```
192.168.0.149.9405  > 192.168.0.155.53: A? infome.gree.com
192.168.0.163.53201 > 192.168.0.155.53: A? infome.gree.com
```

You can also confirm from the Pi-hole **Query Log** (client column shows the LAN IPs) and the dashboard's *Top Clients*.

## Custom domain blocking

Pi-hole → **Domains** → add a domain, type **Deny**, mode **Wildcard** (covers subdomains):

```
(\.|^)tiktok\.com$
(\.|^)youtube\.com$
```

Verify from the server:

```bash
nslookup tiktok.com 127.0.0.1   # -> :: / 0.0.0.0  (blocked)
```

> **Tip:** wildcard deny rules are regexes — double-check spelling. `youtub.com` will *not* block YouTube.
