# The Homelab — Starting Point

> The on-premise infrastructure before the hybrid transition.

---

## Hardware

| Component | Role |
|-----------|------|
| **OPNsense** (bare metal) | Firewall, gateway, DNS, IDS |
| **Cisco SG-300** | Managed switch |
| **Proxmox PVE** | Primary hypervisor (VMs + LXC containers) |
| **Proxmox Backup Server** | Dedicated backup target for PVE |
| **TrueNAS CORE** (bare metal) | NFS storage appliance |
| **Windows machine** | Workstation / misc |

## Network — Before

The original network was simple. No VLANs — everything sat on flat subnets:

| Subnet | Purpose |
|--------|---------|
| **DMZ** | Raspberry Pi (public-facing services via Cloudflare Tunnels) |
| **LAN** | PCs, workstations, general devices |
| **Infra** | All servers and VMs — Proxmox, TrueNAS, every container and VM lived here |
| **IoT** | Wireless IoT devices |

There was no segmentation between services. Every server and VM shared the
same Infra subnet.

## Network Topology — Before

```
Internet
    │
    ▼
┌──────────┐
│ OPNsense │  ← Firewall, AdGuard Home (DNS), Unbound, Suricata
│ (bare    │
│  metal)  │
└────┬─────┘
     │
  ┌──┴──┐
  │     │
  ▼     ▼
┌─────┐ ┌────────────┐
│ LAN │ │ DMZ        │
│     │ │            │
 │   │ ┌──────┐     │
 │   │ │ RPi  │ ← Cloudflare Tunnel
 │   │ │(mgmt)│     │
 │   │ └──────┘     │
 │   └────────────┘
 │
 ├──▶ Infra subnet (flat — all servers, VMs, containers)
 │         │
 │         ├── Proxmox PVE
 │         ├── TrueNAS CORE
 │         ├── Proxmox Backup Server
 │         └── All VM/CT traffic
 │
 └──▶ IoT subnet
```

## DNS — Before

A single **Caddy** instance served `internal.domain.ltd` for local service
discovery. **AdGuard Home** handled DNS rewrites for `internal.domain.ltd`,
resolving service hostnames to their local IPs.

There was no split between internal and external zones — one Caddy instance,
one domain, AdGuard doing the rewrites.

> AdGuard Home still handles DNS rewrites today, now for both `int.domain.ltd`
> and `ext.domain.ltd`. The technique survived the migration; only the zones
> and Caddy instances changed. See [15-DNS-architecture.md](15-DNS-architecture.md)
> for the current setup.

## Switch — Cisco SG-300

The managed switch handles VLAN segmentation between LAN and DMZ. The DMZ
isolates the Raspberry Pi from the internal network while still allowing
controlled outbound access.

## Firewall — OPNsense

OPNsense runs on bare metal with the following plugins:

| Plugin | Purpose |
|--------|---------|
| **Unbound** | Recursive DNS resolver (local, not forwarding) |
| **AdGuard Home** | DNS-level ad/tracker blocking + DNS rewrites |
| **Suricata** | Intrusion detection and prevention (IDS/IPS) |

### WireGuard on OPNsense

A WireGuard tunnel is configured directly on the OPNsense box. This is the
only method for remote access to internal services — no management interfaces
are ever exposed publicly.

## Compute — Proxmox

The Proxmox node served dual duty: primary hypervisor for all VMs and LXC
containers, **and** host for several public-facing services (before the
migration to VPS).

## Storage — TrueNAS CORE

TrueNAS runs on dedicated hardware as a pure storage appliance:

- **NFS** is the primary sharing protocol (used by Proxmox for VM storage)
- No apps or jails — keeping the appliance focused on storage
- Backup and snapshot strategy: *(to be documented)*

## Public Access — DMZ Raspberry Pi + Cloudflare Tunnels

A small Raspberry Pi sits in the DMZ, running Cloudflare Tunnels to expose
selected services to the internet without opening ports on the home router.

This was the original public-access mechanism — functional, but limited:
- Single point of failure (one Pi)
- Tied to Cloudflare's tunnel model (proxy, not direct)
- Pi in DMZ adds a device to maintain and secure

This setup was replaced entirely by the Edge VPS + Cloudflare proxy approach
described in [14-Service-migration.md](14-Service-migration.md).
