# Production Architecture

> The final hybrid state: VPS + on-prem, connected by WireGuard, proxied by
> Cloudflare.

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
                         │  - Reverse  │ → Caddy/Traefik
                         │    proxy    │
                         │  - Mail     │ → ProtonMail SMTP relay
                         │    relay    │
                         │  - Light    │ → notifications, etc.
                         │    services │
                         └──────┬──────┘
                                │ WireGuard
                         ┌──────▼──────┐
                         │  Core VPS   │
                         │             │
                         │  - Docker   │ ← Compose stacks, Git-controlled
                         │  - Monitoring│
                         │  - SeaFile  │
                         └──────┬──────┘
                                │ WireGuard (site-to-site)
                         ┌──────▼──────┐
                         │  OPNsense   │
                         │  (home)     │
                         └──────┬──────┘
                                │
                    ┌───────────┼───────────────────────┐
                    │           │                       │
              ┌─────▼───┐ ┌────▼────┐ ┌───────────────▼──────────┐
              │ Proxmox │ │ TrueNAS │ │ PBS                      │
              │ PVE     │ │ CORE    │ │ (Backup Srv)             │
              └────┬────┘ └─────────┘ └──────────────────────────┘
                   │
              ┌────┴─────────────────────────────────────────────┐
              │ On-Prem VMs & LXC                               │
              │ (see [18-Proxmox-inventory.md](18-Proxmox-inventory.md)) │
              └──────────────────────────────────────────────────┘
```

## Service Placement

For detailed service lists, see:
- **Cloud services:** [13-Core-VPS-services.md](13-Core-VPS-services.md) and [14-Service-migration.md](14-Service-migration.md)
- **On-prem VMs/CTs:** [18-Proxmox-inventory.md](18-Proxmox-inventory.md)

### Cloud

| Service | Placement | Rationale |
|---------|-----------|-----------|
| Reverse proxy | Edge VPS (systemd) | Terminates public traffic, routes to Core/on-prem |
| Mail relay | Edge VPS (systemd) | Public-facing SMTP relay to ProtonMail |
| Light services | Edge VPS (systemd) | Notifications and similar lightweight public services |
| Docker containers | Core VPS | Always-on services, VPN-only, stacks in Git |
| Monitoring stack | Core VPS | Metrics, dashboards, planned alerting |
| SeaFile | Core VPS | Self-hosted file sync — demonstrates hybrid value |

### On-Prem

| Service | Placement | VLAN | Rationale |
|---------|-----------|------|-----------|
| FreeIPA | VM | Internal | Central auth/DNS — must be VPN-only |
| Portainer | CT | Infra | Docker management |
| Gitea | CT | Infra | Compose file storage |
| Caddy int | CT | Internal | `int.domain.ltd` reverse proxy |
| Caddy ext | CT | External | `ext.domain.ltd` reverse proxy |
| Media-srv | VM | External | Arr stack, Jellyfin — heavy compute, can sleep |
| Radio-srv | VM | External | Azuracast — broadcast via Edge VPS web player |
| Home Assistant | VM | Internal | Smart home hub |
| App-srv | VM | Internal | Internal-only applications |
| n8n | CT | Internal | Workflow automation |
| Postfix relay | CT | Infra | Forwards mail to Edge VPS |
| Hermes VM | VM | External | Agent host |
| FreePBX | CT | VoIP | Telephony |
| Backup relay | VM | Internal | Dual-backup: TrueNAS sync → PBS |
| Vault | CT | Infra | WIP — training |
| AI VM | VM | Internal | WIP — voice pipeline |

## Security Boundaries

- **Edge VPS**: Public HTTP/HTTPS only. SSH through WireGuard only.
- **Core VPS**: No public connections at all. SSH through WireGuard only.
- **On-prem**: LAN-only. Reachable from cloud via WireGuard site-to-site.
- **Management interfaces** (Proxmox, TrueNAS, OPNsense): LAN-only, never
  exposed.

For VLAN-level segmentation details, see [16-Networking.md](16-Networking.md).

## DNS Flow

1. User visits `service.domain.ltd`
2. DNS resolves through **Cloudflare** (proxy mode) → Edge VPS IP
3. Cloudflare applies WAF rules, terminates TLS
4. Request reaches Edge VPS reverse proxy (Caddy/Traefik)
5. Reverse proxy routes to backend on Core VPS or on-prem via WireGuard

For full DNS zone details, see [15-DNS-architecture.md](15-DNS-architecture.md).
