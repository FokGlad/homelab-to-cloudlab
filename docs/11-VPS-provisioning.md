# VPS Provisioning

> Two VPS instances from IONOS, sized for their respective roles.

---

## Core VPS

The workhorse of the hybrid setup. Hosts always-on services, Docker
workloads, reverse proxy, and DNS.

| Spec | Value |
|------|-------|
| **Provider** | IONOS |
| **vCPUs** | 6 |
| **RAM** | 8 GB |
| **Storage** | 240 GB |
| **OS** | Debian (hardened) |

## Edge VPS

Minimal instance for public-facing services only.

| Spec | Value |
|------|-------|
| **Provider** | IONOS |
| **vCPUs** | 2 |
| **RAM** | 2 GB |
| **Storage** | 80 GB |
| **OS** | Debian (hardened) |

## Hardening (Both VPS)

Applied immediately after provisioning:

- **Key-only SSH** — password authentication disabled
- **Fail2Ban** — brute-force protection on SSH
- **Unattended-upgrades** — automatic security patches
- **Minimal install** — no unnecessary packages
- **SSH through VPN only** — no public SSH access on either VPS (see
  [12-WireGuard-daisy-chain.md](12-WireGuard-daisy-chain.md))

---

Both instances running Debian for consistency with the existing Linux-focused
homelab tooling.
