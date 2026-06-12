# Proxmox — VMs & LXC Containers

> Full inventory of on-prem Proxmox workloads, organized by VLAN placement.

---

## LXC Containers

| Container | Role | VLAN | Notes |
|-----------|------|------|-------|
| **Portainer** | Docker management | Infra | Manages all Docker containers on-prem |
| **Postfix relay** | Mail relay | Infra | Forwards mail to Edge VPS Postfix |
| **PVE-exporter** | Monitoring | Infra | Prometheus metrics from Proxmox |
| **Vault** | Secrets management | Infra | WIP — training purposes, not yet implemented |
| **Caddy ext** | Reverse proxy | External | Serves `ext.domain.ltd` zone |
| **Caddy int** | Reverse proxy | Internal | Serves `int.domain.ltd` zone |
| **n8n** | Automation | Internal | Workflow automation engine |
| **Gitea Runner** | CI/CD | Infra | Triggers Portainer API on Gitea push (planned) |

## Virtual Machines

| VM | Role | VLAN | Notes |
|----|------|------|-------|
| **FreeIPA** | Identity/DNS | Internal | Central SSH auth, HBAC, DNS zones (`idm.domain.ltd`) |
| **Media-srv** | Media stack | External | Arr stack (Sonarr, Radarr, etc.), torrent client, Jellyfin |
| **Radio-srv** | Radio broadcast | External | Azuracast — broadcast available via self-hosted web player on Edge VPS |
| **App-srv** | Internal apps | Internal | Internal-only applications |
| **Home Assistant** | Smart home | Internal | Home automation hub |
| **AI VM** | Voice pipeline | Internal | WIP — not yet implemented |
| **Backup relay** | Dual backup | Internal | Syncs from TrueNAS share, then backed up by PBS |
| **Hermes VM** | Agent host | External | Hermes agent home |

---

## VLAN Distribution

```
Infra VLAN:     Portainer, Postfix relay, PVE-exporter, Vault (WIP), Gitea Runner
Internal VLAN:  FreeIPA, Caddy int, n8n, App-srv, Home Assistant, AI VM (WIP), Backup relay
External VLAN:  Caddy ext, Media-srv, Radio-srv, Hermes VM
VoIP VLAN:      FreePBX
```

---

## Notes

- **Backup relay VM** serves as a dual-backup mechanism: sensitive data is
  synced from a TrueNAS share to this VM, then Proxmox Backup Server backs up
  the VM itself. Two copies, two media.
- **Hermes VM** lives on the External VLAN — accessible for management but
  isolated from Internal/Infra segments.
- **Vault** and **AI VM** are works in progress and not yet in production.
- **FreePBX** runs as a CT on the VoIP VLAN, dedicated to telephony.
