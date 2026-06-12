# State 0 — The Homelab

> The starting point: a fully on-premise setup with zero public exposure.

---

## Hardware

| Component | Details |
|-----------|---------|
| **Hypervisor** | Proxmox VE (single node) |
| **CPU** | Intel i5-10600K (6C/12T) |
| **RAM** | 48 GB DDR4 |
| **Storage** | TrueNAS CORE (bare metal, storage only) |
| **Network** | OPNsense firewall (bare metal) |

## Compute — Proxmox

The Proxmox node runs **8 VMs** and **10 LXC containers**. All workloads are
internal-only; nothing is exposed to the public internet.

### VMs

| VM | Role | OS |
|----|------|----|
| *(to be documented)* | | |

### LXC Containers

| CT | Role | OS |
|----|------|----|
| *(to be documented)* | | |

> **Note:** Specific VM/CT details will be filled in as the documentation
> progresses. The important takeaway: a moderately complex virtualized
> environment, entirely self-hosted.

## Storage — TrueNAS CORE

TrueNAS runs on dedicated hardware as a pure storage appliance. It provides
NFS shares to the Proxmox node and other services on the network.

- **No apps or jails** — storage only, keeping the appliance focused
- **NFS** used as the primary sharing protocol
- Backups and snapshot strategy: *(to be documented)*

## Network — OPNsense

OPNsense runs on bare metal as the network's firewall and gateway. It handles
all routing, filtering, and DNS for the homelab.

### Plugins

| Plugin | Purpose |
|--------|---------|
| **Unbound** | Local DNS resolver (recursive, not forwarding) |
| **AdGuard Home** | DNS-level ad/tracker blocking |
| **IDS** | Intrusion detection (Suricata) |

### Network Topology (Simplified)

```
Internet
    │
    ▼
┌──────────┐
│ OPNsense │  ← Firewall, DNS, IDS
│ (bare    │
│  metal)  │
└────┬─────┘
     │
     ▼
┌──────────┐     ┌──────────┐
│ Proxmox  │────▶│ TrueNAS  │  ← NFS storage
│ (VMs/CTs)│     │ (CORE)   │
└──────────┘     └──────────┘
```

## Access Model

- **No public services.** Nothing is exposed to the internet.
- **WireGuard VPN** is the only way to access internal services remotely.
- All management interfaces (Proxmox, TrueNAS, OPNsense) are LAN-only.

## Limitations of This State

| Limitation | Impact |
|------------|--------|
| **Single-homed** | Residential ISP = single point of failure. No redundancy. |
| **No public presence** | Can't host services accessible without a VPN. |
| **Power dependency** | Home power outage = total downtime. |
| **ISP constraints** | No static IP (likely), CGNAT possible, bandwidth asymmetry. |
| **Hardware risk** | Single Proxmox node = no HA. Disk failure on TrueNAS = data loss. |

These limitations are the **motivation** for the hybrid architecture described
in [02-The-plan.md](02-The-plan.md).
