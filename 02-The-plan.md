# State 1 — The Plan

> Hybrid VPS + on-prem architecture: a core VPS in the cloud, edge VPS
> instances for public availability, and the on-prem homelab as the origin.

---

## Architecture Overview

Three tiers connected via WireGuard:

```
                         ┌─────────────┐
                         │  Internet   │
                         └──────┬──────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
              ┌─────▼─────┐           ┌─────▼─────┐
              │ Edge VPS  │           │ Edge VPS  │
              │ (minimal) │           │ (minimal) │
              └─────┬─────┘           └─────┬─────┘
                    │                       │
                    └───────────┬───────────┘
                                │
                          WireGuard mesh
                          (daisy-chained)
                                │
                         ┌──────▼──────┐
                         │  Core VPS   │
                         │ (IONOS)     │
                         │             │
                         │  Services:  │
                         │  - Reverse  │
                         │    proxy    │
                         │  - DNS      │
                         │  - Auth     │
                         │  - Docker   │
                         │    workloads│
                         └──────┬──────┘
                                │
                          WireGuard
                          (site-to-site)
                                │
                    ┌───────────▼───────────┐
                    │    On-Prem Homelab    │
                    │                       │
                    │  ┌─────┐   ┌──────┐  │
                    │  │OPNsense│ │Proxmox│ │
                    │  └──┬──┘   └──┬───┘  │
                    │     │     ┌───▼───┐  │
                    │     │     │TrueNAS│  │
                    │     │     └───────┘  │
                    └──────────────────────┘
```

## The Three Tiers

### Tier 1 — Core VPS (IONOS)

The **brain** of the hybrid setup. This is where orchestration and management
live.

- **Provisioned and hardened** — minimal attack surface, automatic updates,
  fail2ban, key-only SSH
- **Joined to the WireGuard mesh** — connects to both edge VPS instances and
  the on-prem homelab
- **Docker workloads** — all containerized services run here, stacks managed
  via Portainer and **kept in Git**
- **Reverse proxy, DNS, TLS** — Caddy/Traefik terminates TLS and routes to
  backend services
- **Identity / auth** — Single Sign-On for all exposed services
- **Central logging and monitoring**

### Tier 2 — Edge VPS

**Minimal, disposable, public-facing.** Each edge VPS runs only what's needed
to maintain 24/7 availability of a specific service.

- Connects **through** the Core VPS via WireGuard (daisy-chain)
- Runs only lightweight services (static sites, specific app frontends,
  maybe a game server or two)
- Can be rebuilt from scratch in minutes — nothing stateful lives here
- **Cost-optimized**: smaller instances, purged when no longer needed
- Multiple edge instances for redundancy or service isolation

### Tier 3 — On-Prem Homelab

The **origin** — where data lives and where workloads run day-to-day.

- Connected to Core VPS via WireGuard site-to-site tunnel
- Proxmox VMs, TrueNAS storage, OPNsense firewall remain active
- Services that **don't need** public availability stay here (media server,
  personal tools, experimental VMs, Backup target)
- Gradually **decommissioned** as cloud equivalents prove reliable

## Why a Daisy-Chain?

| Concern | Daisy-Chain (via Core) | Direct (Full Mesh) |
|---------|----------------------|-------------------|
| **Simplicity** | Edge only needs 1 tunnel | Edge needs N tunnels |
| **Security** | Edge can't reach homelab directly | If edge is compromised, homelab is exposed |
| **Control** | All traffic flows through core (logging, filtering) | Distributed trust model |
| **Cost** | Less bandwidth at edge | Edge needs more bandwidth |

The daisy-chain means the Core VPS acts as a **chokepoint by design** — all
traffic between edge and on-prem is visible and filterable. If an edge instance
is compromised, the attacker still has to get through the Core VPS to reach
the homelab.

## Connectivity

- **WireGuard** for all inter-site tunnels
- Every node has a persistent, static WireGuard IP within the mesh
- On-prem connects to Core via **site-to-site** tunnel (OPNsense ↔ Core VPS)
- Edge VPS instances connect back to Core, **not** directly to on-prem

## Service Placement Strategy

| Service Type | Placement | Rationale |
|-------------|-----------|-----------|
| Public websites / APIs | Edge VPS + Core VPS proxy | 24/7 availability, TLS termination |
| Auth / SSO | Core VPS | Centralized, behind VPN, not on edge |
| Email | Core VPS | Needs stable IP, reputation management |
| Monitoring / Logging | Core VPS | Aggregation point, persistent storage |
| Docker workloads | Core VPS (primary), some at edge | Bulk of container services |
| Media / personal tools | On-prem only | No need to be public, saves money |
| Storage / backups | On-prem (TrueNAS) + VPS backups | Primary on-prem, off-site copies |

## Roadmap

### Phase 1 — Foundation ✅

- [x] VPS provisioned and hardened
- [x] WireGuard mesh established (on-prem ↔ core)
- [x] DNS, reverse proxy, TLS configured
- [x] Docker stacks running, Git-controlled via Portainer

### Phase 2 — Edge Deployments

- [ ] First edge VPS provisioned (daisy-chained through core)
- [ ] Service migrated to edge for 24/7 availability
- [ ] TLS terminating at edge, proxied back to core or on-prem

### Phase 3 — Infrastructure as Code

- [ ] Terraform/Pulumi definitions for VPS provisioning
- [ ] Ansible playbooks for OS hardening
- [ ] Docker Compose stacks fully in CI/CD pipeline

### Phase 4 — Decommissioning

- [ ] Identify candidate services for full cloud migration
- [ ] Migrate workloads from on-prem Proxmox to VPS
- [ ] Reduce on-prem footprint
- [ ] Document lessons from each migration

---

*This plan evolves as the architecture does. Last updated: June 2026.*
