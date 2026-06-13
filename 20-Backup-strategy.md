# Backup Strategy

> Current state and future plan for data protection across the hybrid
> architecture.

---

## Current State

The on-prem backup pipeline is operational. The gap is off-site replication
and long-term archival — not the core backup mechanism itself.

### VPS → On-Prem Pipeline

Each VPS runs a script that **pushes data to an NFS share** on the on-prem
TrueNAS. A **Backup relay VM** on the Internal VLAN:

1. Picks up the data from the TrueNAS NFS share
2. Gets itself backed up by **Proxmox Backup Server (PBS)**

This gives two copies of VPS data: the original on the NAS share, and the PBS
backup on-prem.

```
Edge VPS ──┐
            ├──▶ Script → NFS share (TrueNAS) → Backup relay VM → PBS
Core VPS ──┘
```

### What's Covered

| Data | Backed up | Method |
|------|-----------|--------|
| VPS application data | ✅ | Script → NFS → PBS |
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

---

*Last updated: June 2026.*
