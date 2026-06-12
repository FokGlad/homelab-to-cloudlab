# DNS Architecture

> Four DNS zones, each handled by a different server, each serving a distinct
> purpose in the hybrid architecture.

---

## The Four Zones

| Zone | Domain | Handled By | Purpose |
|------|--------|------------|---------|
| **Public** | `domain.ltd` | Cloudflare | Public-facing services |
| **Internal** | `int.domain.ltd` | On-prem Caddy (internal instance) | Internal-only services |
| **External** | `ext.domain.ltd` | On-prem Caddy (external instance) | On-prem services, externally accessible via VPN |
| **Machine** | `idm.domain.ltd` | FreeIPA | Server hostname resolution (forward + reverse) |

---

## Zone Details

### `domain.ltd` — Public

- Hosted on **Cloudflare** (authoritative DNS + proxy)
- Only records for services that need to be publicly reachable
- Cloudflare handles TLS termination at the edge, WAF, and DDoS protection
- Points to the **Edge VPS** IP (orange-clouded/proxied)
- Edge VPS reverse proxy (Caddy) routes to backend services

This replaced the old Cloudflare Tunnel (`cloudflared`) setup that ran on the
DMZ Raspberry Pi.

### `int.domain.ltd` — Internal

- Hosted on-prem, **LAN-only**
- Handled by **dedicated internal Caddy instance** running a local DNS resolver
- Only accessible from the **Internal VLAN** (see [15-Networking.md](15-Networking.md))
- Also accessible via a **dedicated admin WireGuard tunnel** for remote access
- Never exposed to the internet — no public records

Used for services that should only ever be reachable from inside the home
network (dashboards, admin panels, internal APIs).

### `ext.domain.ltd` — External (On-Prem Services)

- Hosted on-prem, split access model:
  - **Locally**: accessible via **dedicated external Caddy instance**
    (separate from the internal one)
  - **Externally**: accessible via a **dedicated VPN tunnel** — this is a
    separate tunnel from the admin tunnel, specifically for reaching on-prem
    external services from outside
- Runs on the **External VLAN**
- For services hosted on-prem that you want to reach from outside, without
  putting them on the VPS

This is the zone for "I host this at home but still want to reach it when I'm
not home" — securely, through a VPN tunnel rather than exposing it publicly.

### `idm.domain.ltd` — Machine Hostnames

- Hosted on **FreeIPA** (which includes an integrated DNS server)
- Used exclusively for resolving **server and VM hostnames**
- Forward records: `servername.idm.domain.ltd` → IP
- Reverse records: IP → `servername.idm.domain.ltd`
- Only machines joined to the FreeIPA realm can resolve these records

This means you SSH to machines by hostname without needing to remember IPs:
```bash
ssh user@proxmox.idm.domain.ltd
ssh user@core-vps.idm.domain.ltd
```

---

## Caddy DNS On-Prem

Two separate Caddy instances handle the on-prem DNS zones:

| Instance | Zone | VLAN | Access |
|----------|------|------|--------|
| **Internal Caddy** | `int.domain.ltd` | Internal VLAN | Local + admin VPN tunnel |
| **External Caddy** | `ext.domain.ltd` | External VLAN | Local + external VPN tunnel |

The split means internal services are never routable from the external VLAN
and vice versa — true DNS-level separation matching the VLAN boundaries.

---

## Resolution Flow

```
User → service.domain.ltd
  └→ Cloudflare DNS → Edge VPS (public)

User (on LAN) → app.int.domain.ltd
  └→ Internal Caddy → Internal VLAN service

User (admin VPN) → app.int.domain.ltd
  └→ WireGuard admin tunnel → Internal Caddy → Internal VLAN service

User (LAN or ext VPN) → service.ext.domain.ltd
  └→ External Caddy → External VLAN service

Any FreeIPA machine → server.idm.domain.ltd
  └→ FreeIPA DNS → IP (forward or reverse)
```
