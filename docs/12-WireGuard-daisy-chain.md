# WireGuard Daisy-Chain

> All inter-site connectivity is WireGuard. No public SSH anywhere.

---

## Topology

```
Internet
    │
    │  (SSH via VPN only — no public SSH)
    │
┌───▼────────────┐
│   Edge VPS     │  ← Public-facing: web services, ntfy, postfix
│   (IONOS)      │
└───┬────────────┘
    │ WireGuard
    │
┌───▼────────────┐
│   Core VPS     │  ← No public connections at all
│   (IONOS)      │     Services: reverse proxy, DNS, auth, Docker
└───┬────────────┘
    │ WireGuard (site-to-site)
    │
┌───▼────────────┐
│  OPNsense      │  ← On-prem gateway
│  (home)        │
└───┬────────────┘
    │
    ▼
  Homelab LAN
  (Proxmox, TrueNAS, etc.)
```

## Connection Rules

### Core VPS
- **Accepts** connections from Edge VPS and on-prem (OPNsense)
- **Denies** all other public connections
- **SSH is only accessible through the WireGuard tunnel** — no public SSH
  port, no password auth, key-only

### Edge VPS
- **Connects to on-prem through Core** — never directly
- **Public-facing** for web services (HTTP/HTTPS from the internet)
- **SSH is only accessible through the WireGuard tunnel** — same as Core,
  no public SSH access

### On-Prem (OPNsense)
- Site-to-site tunnel to Core VPS
- All internal services reachable through the tunnel, never exposed to
  the internet

## Why Daisy-Chain Instead of Full Mesh?

| Concern | Daisy-Chain (via Core) | Full Mesh |
|---------|----------------------|-----------|
| **Simplicity** | Edge only needs 1 WireGuard peer | Edge needs N peers |
| **Security** | Edge can't reach homelab directly | Compromised edge = exposed homelab |
| **Control** | All cross-site traffic flows through Core (logging, filtering) | Distributed trust model |
| **Cost** | Lower bandwidth requirement at edge | Edge needs more bandwidth |

The Core VPS is a **chokepoint by design**. If the Edge VPS is compromised,
the attacker still has to get through Core to reach the homelab.
