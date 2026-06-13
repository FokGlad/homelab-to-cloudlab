# DNS Architecture

> Four DNS zones, each handled by a different server, each serving a distinct
> purpose in the hybrid architecture.

---

## The Four Zones

| Zone | Domain | Handled By | Purpose |
|------|--------|------------|---------|
| **Public** | `domain.ltd` | Cloudflare | Public-facing services |
| **Internal** | `int.domain.ltd` | AdGuard Home + internal Caddy instance | Internal-only services |
| **External** | `ext.domain.ltd` | AdGuard Home + external Caddy instance | On-prem services, externally accessible via VPN |
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
- **AdGuard Home** handles DNS rewrites for `int.domain.ltd`, resolving
  service hostnames to their local IPs
- A **dedicated internal Caddy instance** serves the actual services behind
  those hostnames
- Only accessible from the **Internal VLAN** (see [16-Networking.md](16-Networking.md))
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
- **AdGuard Home** handles DNS rewrites for `ext.domain.ltd` (same technique
  as `int.domain.ltd`)
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

## AdGuard Home — DNS Rewrites & Forwarding

AdGuard Home has been part of the DNS setup since before the hybrid transition.
Originally it handled rewrites for a single `internal.domain.ltd` zone. Today
it handles **three** local zones:

| Zone | Action | Target |
|------|--------|--------|
| `int.domain.ltd` | DNS rewrite | Internal Caddy IP |
| `ext.domain.ltd` | DNS rewrite | External Caddy IP |
| `idm.domain.ltd` | Forward | FreeIPA DNS |

Anything that doesn't match a local zone is forwarded to **Unbound** for
recursive resolution via DNS over TLS.

The two on-prem Caddy instances (internal and external) serve the actual
services. AdGuard handles the name resolution; Caddy handles the HTTP routing
and TLS.

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

Every DNS query from a device on the home network follows this path:

```
Device
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  AdGuard Home (on OPNsense)                         │
│                                                     │
│  1. Check local cache                               │
│  2. Apply filtering (ad/tracker blocking)           │
│  3. Zone-based routing:                             │
│     ├─ int.domain.ltd  → rewrite to Internal Caddy  │
│     ├─ ext.domain.ltd  → rewrite to External Caddy  │
│     └─ idm.domain.ltd  → forward to FreeIPA DNS     │
│  4. Unknown/unmatched → forward to Unbound          │
└──────────────────────┬──────────────────────────────┘
                       │
              ┌────────┴────────┐
              ▼                 ▼
     ┌──────────────┐   ┌──────────────┐
     │  Unbound DNS │   │  Zone target │
     │  (on OPNsense│   │  (Caddy /    │
     │   DoT to     │   │   FreeIPA)   │
     │   upstream)  │   └──────────────┘
     └──────────────┘
```

### Step by Step

1. **Device** sends DNS query to AdGuard Home (configured as DNS server via DHCP on all VLANs)
2. **AdGuard Home** checks its cache and applies filtering rules (ad/tracker blocking)
3. **Zone dispatch** — AdGuard matches the requested domain against its configured zones:
   - `*.int.domain.ltd` → DNS rewrite to the **Internal Caddy** IP
   - `*.ext.domain.ltd` → DNS rewrite to the **External Caddy** IP
   - `*.idm.domain.ltd` → forwarded to **FreeIPA DNS** (which holds forward + reverse records for all machine hostnames)
4. **Fallback** — anything that doesn't match a local zone is forwarded to **Unbound**, which resolves it via **DNS over TLS** to upstream resolvers
5. **Response** flows back through the same chain to the device

### Public Queries (Cloudflare)

External clients resolve `domain.ltd` through **Cloudflare** (authoritative + proxy), completely bypassing the on-prem DNS chain. Cloudflare points to the Edge VPS IP.

```bash
# Example: on-prem device resolves an internal host
$ dig app.int.domain.ltd
# → AdGuard rewrite → Internal Caddy IP (e.g. 10.0.10.5)

# Example: any device resolves a machine hostname
$ dig proxmox.idm.domain.ltd
# → AdGuard forwards to FreeIPA → 10.0.20.10

# Example: any device resolves a public service
$ dig service.domain.ltd
# → AdGuard forwards to Unbound → resolves via Cloudflare → Edge VPS IP

# Example: query for an unknown external domain
$ dig example.com
# → AdGuard cache miss → Unbound → DoT to upstream → resolved
```

### Key Points

- **AdGuard Home is the single entry point** for all on-prem DNS — devices never query Unbound or FreeIPA directly
- **Filtering and caching happen first** — every query benefits from ad-blocking and cached responses regardless of destination
- **DNS over TLS** ensures queries that leave the network (via Unbound) are encrypted in transit to upstream resolvers
- **FreeIPA DNS is authoritative only for `idm.domain.ltd`** — AdGuard forwards, it doesn't query FreeIPA for other zones
