# Backup Strategy

> Current state and future plan for data protection across the hybrid
> architecture.

---

## Current State

The on-prem backup pipeline is operational. The gap is off-site replication
and long-term archival — not the core backup mechanism itself.

### Core VPS & Edge VPS — Scripted Backup

Both VPS instances run the **same backup script**. The script archives important
directories (`/etc`, `/opt`) and pushes them to an NFS share on the on-prem
TrueNAS.

On the Core VPS, every Docker container is configured with a mountpoint to
local storage — this is what the script backs up.

```
Edge VPS ──┐
            ├──▶ Backup script → NFS share (TrueNAS) → Backup relay VM → PBS
Core VPS ──┘
```

### Proxmox — PBS

All VMs and LXC containers on the Proxmox node are backed up daily to
Proxmox Backup Server (PBS).

### What's Covered

| Data | Backed Up | Method |
|------|-----------|--------|
| VPS application data (`/etc`, `/opt`, container storage) | ✅ | Script → NFS → PBS |
| Proxmox VMs/CTs | ✅ | PBS (local backup) |
| TrueNAS data | ❌ | No off-site copy |
| Configs / Compose files | ✅ | Gitea (Git, on-prem) |

---

## Future Plan

### Blu-ray Archival

For **important documents and irreplaceable data**: periodic Blu-ray disc
backups stored offline. This covers the scenario of both on-prem and VPS
being compromised simultaneously.

### Off-site TrueNAS Replication

A **second TrueNAS** at a different location (family member, friend's house)
replicating from the on-prem TrueNAS:

- ZFS send/receive for efficient incremental replication
- Covers TrueNAS data that currently has no off-site copy
- Replication over WireGuard or a dedicated tunnel

### Target State (Planned)

```
On-Prem                          Off-site
┌──────────┐                    ┌──────────┐
│ TrueNAS  │─── ZFS replicate ─▶│ TrueNAS  │
│ (primary)│                    │ (replica)│
└────┬─────┘                    └──────────┘
     │
     ▼
┌──────────┐
│ PBS      │ ← Proxmox VMs/CTs + VPS data
└──────────┘

Blu-ray discs ← Important documents (offline, off-site)
```

---

## Open Questions

- **Replication frequency:** hourly? daily? depends on bandwidth to off-site
- **Retention:** how long to keep ZFS snapshots on the off-site TrueNAS
- **Blu-ray schedule:** quarterly? semi-annually?
- **Off-site location:** still being arranged
