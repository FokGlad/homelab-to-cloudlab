# Production Architecture

> The final hybrid state: VPS + on-prem, connected by WireGuard, proxied by Cloudflare.

---

## Current Topology

```
                         ┌─────────────┐
                         │  Internet   │
                         └──────┬──────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    │   Cloudflare Proxy    │
                    │   (DNS + WAF + TLS)   │
                    │                       │
                    └───────────┬───────────┘
                                │
                         ┌──────▼──────┐
                         │  Edge VPS   │
                         │             │
                         │  - Postfix  │ → ProtonMail SMTP relay
                         │  - ntfy     │
                         │  - Reverse  │ → Caddy/Traefik
                         │    proxy    │
                         └──────┬──────┘
                                │ WireGuard
                         ┌──────▼──────┐
                         │  Core VPS   │
                         │             │
                         │  - FreeIPA  │ ← SSH auth, DNS zones, access control
                         │  - Vikunja  │
                         │  - Memos    │
                         │  - Grafana  │
                         │  - Prometheus│
                         │  - SeaFile  │
                         │  - Docker   │ ← Compose stacks, Git-controlled
                         └──────┬──────┘
                                │ WireGuard (site-to-site)
                         ┌──────▼──────┐
                         │  OPNsense   │
                         │  (home)     │
                         └──────┬──────┘
                                │
                    ┌───────────┼───────────┐
                    │           │           │
              ┌─────▼───┐ ┌────▼────┐ ┌───▼──────────┐
              │ Proxmox │ │ TrueNAS │ │ PBS          │
              │ PVE     │ │ CORE    │ │ (Backup Srv) │
              └─────────┘ └─────────┘ └──────────────┘
```

## Service Placement

| Service | Placement | Rationale |
|---------|-----------|-----------|
| Postfix | Edge VPS | Public-facing SMTP relay to ProtonMail |
| ntfy | Edge VPS | Lightweight push notifications, public-facing |
| Reverse proxy | Edge VPS | Terminates public traffic, routes to Core/on-prem |
| FreeIPA | Core VPS | Central auth/DNS — must be VPN-only |
| Vikunja | Core VPS | Internal app, no need for public exposure |
| Memos | Core VPS | Internal app |
| Grafana + Prometheus | Core VPS | Monitoring, needs persistent storage |
| SeaFile | Core VPS | File sync, heavier storage needs |
| Docker workloads | Core VPS | Primary container host, stacks in Git |
| Storage / backups | On-prem (TrueNAS + PBS) | Heavy storage stays home, power-optional |

## Security Boundaries

- **Edge VPS**: Public HTTP/HTTPS only. SSH through WireGuard only.
- **Core VPS**: No public connections at all. SSH through WireGuard only.
- **On-prem**: LAN-only. Reachable from cloud via WireGuard site-to-site.
- **Management interfaces** (Proxmox, TrueNAS, OPNsense): LAN-only, never
  exposed.

## DNS Flow

1. User visits `service.domain.tld`
2. DNS resolves through **Cloudflare** (proxy mode) → Edge VPS IP
3. Cloudflare applies WAF rules, terminates TLS
4. Request reaches Edge VPS reverse proxy (Caddy/Traefik)
5. Reverse proxy routes to backend on Core VPS or on-prem via WireGuard
