# 04 — Security Lessons Learned

Building this against a live network surfaced several real-world issues that a purely "happy path" tutorial hides. These are the interesting part.

## 1. Client-side privacy features bypass network DNS filtering

An iPhone on the network kept resolving normally even after DNS was pointed at Pi-hole. Root cause: **iCloud Private Relay**. iOS routes DNS (and Safari traffic) through Apple's encrypted relay, ignoring the network-assigned resolver entirely — the manual DNS field is even locked with the message *"DNS requests are being routed by iCloud Private Relay… Turn off Private Relay to manually configure DNS settings."*

**Takeaway:** network-level DNS filtering is not absolute. Modern OSes and browsers ship privacy features that tunnel out of it. Enforcing policy on such clients requires either an MDM profile, blocking the relay's known endpoints, or disabling the feature per-device.

## 2. Encrypted DNS (DoH) in browsers also bypasses Pi-hole

Chrome/Edge default to **DNS-over-HTTPS** to their own resolver. Even with the OS pointed at Pi-hole, the browser can resolve names itself, so blocks don't apply. Testing must be done with DoH/"Secure DNS" disabled, or the block simply appears not to work.

## 3. Hyper-V host ↔ guest UDP limitation

The Windows **host** running the VM could not query the VM's Pi-hole over UDP/53 (queries timed out), even though every *other* device on the LAN could, and TCP (SSH, the web UI) worked host→guest. This is a known quirk of the Hyper-V External virtual switch for host-to-guest traffic.

**Takeaway:** don't validate a service from the hypervisor host — test from an independent client. It's also a concrete reason to run this on **dedicated hardware (Raspberry Pi)** rather than a VM on a daily-driver PC.

## 4. Freeing port 53 on Ubuntu

`systemd-resolved` owns `127.0.0.53:53` by default, so Pi-hole can't bind port 53 until the **stub listener** is disabled and `/etc/resolv.conf` is repointed at the real upstream. (See [02-deployment.md](02-deployment.md).)

## 5. Container image path changes

Recent NetAlertX images moved persistence from `/app/config` + `/app/db` to **`/data/config`** + **`/data/db`**. Mounting the old paths makes the container crash-loop on its startup checks (exit code 126). Always check the current image's expected mount points.

## 6. Wildcard deny rules are regexes

A one-character typo (`youtub.com` vs `youtube.com`) silently blocks nothing. Verify every rule with `nslookup <domain> 127.0.0.1`.
