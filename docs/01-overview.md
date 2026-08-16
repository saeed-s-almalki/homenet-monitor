# 01 — Overview & Lab Setup

## Goal

Get full visibility and control over a home network:

1. **Know every device** connected to the network and get alerted when a new/unknown one appears.
2. **Route all DNS** through a filtering resolver to block ads/trackers and, optionally, specific sites.
3. Do it on a **portable, reproducible** stack (Docker) that can later move to a Raspberry Pi for 24/7 operation.

## Lab environment

| Item | Value |
|---|---|
| Host OS | Ubuntu Server 22.04.5 LTS |
| Platform | Hyper-V VM, Generation 2 |
| vCPU / RAM / Disk | 2 vCPU / 3 GB (dynamic) / 25 GB |
| Network | **External** virtual switch (VM sits directly on the home LAN) |
| Server IP | 192.168.0.155 (`192.168.0.0/24`, gateway 192.168.0.1) |
| Runtime | Docker Engine + Docker Compose |

> **Why an External switch?** NetAlertX discovers devices by ARP-scanning the local subnet. The VM must be a first-class member of the home LAN (not NAT'd behind the host), so it uses a Hyper-V **External** switch bridged to the physical adapter.

## Why these tools

- **NetAlertX** (formerly PiAlert) — lightweight ARP-based device discovery with a clean UI, vendor lookup, first-seen tracking, and new-device notifications. Container-friendly and Raspberry-Pi-ready.
- **Pi-hole v6** — the de-facto self-hosted DNS sinkhole: blocklists, per-client query logging, and simple allow/deny rules.

## Cost

Zero recurring cost — everything is open-source and runs on hardware already owned. Designed to move to a Raspberry Pi later.
